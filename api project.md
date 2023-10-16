# Utilisation de l'API de Riot Games et affichage des données du compte

Importation des bibliothèques et modules utiles


```python
import requests
import time
import pandas as pd

pd.set_option('display.max_rows',None)

```

Définition des fonctions 


```python
# Cette fonction permet de récupérer le puuid d'un joueur à partir du nom d'invocateur, de la région, et d'une clef api
def get_puuid(summoner_name, region, api_key):
    api_url = (
        "https://" + 
        region +
        ".api.riotgames.com/lol/summoner/v4/summoners/by-name/" +
        summoner_name +
        "?api_key=" +
        api_key
    )
    resp = requests.get(api_url)
    player_info = resp.json()
    puuid = player_info['puuid']
    return puuid  


# Cette fonction permet de récupérer l'identifiant correspondant à l'icone d'un joueur
def get_icon_id(summoner_name,region, api_key):
    api_url = (
        "https://" + 
        region +
        ".api.riotgames.com/lol/summoner/v4/summoners/by-name/" +
        summoner_name +
        "?api_key=" +
        api_key
    )
    resp = requests.get(api_url)
    player_info = resp.json()
    icon_id = player_info['profileIconId']
    return icon_id  


# Cette fonction permet de récupérer le niveau d'un joueur
def get_summoner_level(summoner_name,region, api_key):
    api_url = (
        "https://" + 
        region +
        ".api.riotgames.com/lol/summoner/v4/summoners/by-name/" +
        summoner_name +
        "?api_key=" +
        api_key
    )
    resp = requests.get(api_url)
    player_info = resp.json()
    summoner_level = player_info['summonerLevel']
    return summoner_level  



# cette fonction permet de récupérer les identifiants des 20 dernières parties d'un joueur
def get_match_ids(puuid, mass_region, api_key):
    api_url = (
        "https://" +
        mass_region +
        ".api.riotgames.com/lol/match/v5/matches/by-puuid/" +
        puuid + 
        "/ids?start=0&count=20" + 
        "&api_key=" + 
        api_key
    )   
    # Boucle permettant de gérer les cas d'erreurs (rate limits)
    while True:
        resp = requests.get(api_url)
        
        if resp.status_code == 429:
            print("Rate Limit hit, sleeping for 10 seconds")
            time.sleep(10)
            continue
            
        match_ids = resp.json()
        return match_ids 


# Cette fonction permet de récupérer les données d'une partie
def get_match_data(match_id, mass_region, api_key):
    api_url = (
        "https://" + 
        mass_region + 
        ".api.riotgames.com/lol/match/v5/matches/" +
        match_id + 
        "?api_key=" + 
        api_key
    )   
    while True:
        resp = requests.get(api_url)
        
        if resp.status_code == 429:
            print("Rate Limit hit, sleeping for 10 seconds")
            time.sleep(10)
            continue
            
        match_data = resp.json()
        return match_data  


  
    
    

# Fonction permettant de récupérer les données d'un joueur à partie des données d'une partie et d'un ID
def find_player_data(match_data, puuid):
    participants = match_data['metadata']['participants']
    player_index = participants.index(puuid)
    player_data = match_data['info']['participants'][player_index]
    return player_data


# Cette fonction permet de récupérer le rang flex et solo/duo d'un joueur aisi que son winrate
def get_rank(encryptedSummonerId, region, api_key):
    api_url = f"https://{region}.api.riotgames.com/lol/league/v4/entries/by-summoner/{encryptedSummonerId}?api_key={api_key}"
    resp = requests.get(api_url)
    player_info = resp.json()
    rank_solo = "unranked"
    winrate_solo = None
    rank_flex = "unranked"
    winrate_flex = None

    for elem in player_info :
        if elem['queueType'] == 'RANKED_SOLO_5x5':
            rank_solo = elem['tier'] + " " + elem['rank']  + " " + str(elem['leaguePoints'])
            winrate_solo = round(elem['wins'] / (elem['wins'] + elem['losses']) *100)
        if elem['queueType'] == 'RANKED_FLEX_SR':
            rank_flex = elem['tier'] + " " + elem['rank']  + " " + str(elem['leaguePoints'])
            winrate_flex = round(elem['wins'] / (elem['wins'] + elem['losses']) *100)
    return rank_solo,rank_flex,winrate_solo,winrate_flex


# Cette fonction permet de récupérer les données d'une partie en filtrant le type de partie (ranked solo ou flex)
def get_match_ids_ranked(puuid, mass_region, api_key, queue):
    print('Récupération des parties classés')
    """
    SOLO : queue = "420"
    FLEX : queue = "440"
    """
    api_url = f"https://{mass_region}.api.riotgames.com/lol/match/v5/matches/by-puuid/{puuid}/ids?queue={queue}&startTime=69897600&count=100&api_key={api_key}"
    while True:   
        resp = requests.get(api_url)
        if resp.status_code == 429:
            print("Rate Limit hit, sleeping for 10 seconds")
            time.sleep(10)
            continue
        match_ids = resp.json()
        return match_ids

# Cette fonction permet de récupérer les données d'un joueur (rang,winrate,kda,icon_id...) sous la forme d'un dictionnaire
def get_player_data(pseudo,region,api_key):
    "Récupération des données du comptes"
    data = {
    'pseudo': [],
    'rank_solo': [],
    'rank_flex': [],
    'winrate' :[],
    'kda': [],
    'icon_id' : [],
    'summoner_level' : []
    }
    data['pseudo'].append(pseudo)
    data['icon_id'].append(get_icon_id(pseudo,region, api_key))
    data['summoner_level'].append(get_summoner_level(pseudo,region, api_key))
    encryptedSummonerId = get_encryptedSummonerId(pseudo, region, api_key)
    puuid = get_puuid(pseudo,region,api_key)
    data['rank_solo'] = get_rank(encryptedSummonerId, region, api_key)[0]
    data['rank_flex'] = get_rank(encryptedSummonerId, region, api_key)[1]
    match_ids = get_match_ids(puuid, mass_region, api_key)
    #On récupère le winrate et le kda des dernières game
    win_list = []
    kda_list = []
    for match_id in match_ids:
        match_data = get_match_data(match_id, mass_region, api_key)
        player_data = find_player_data(match_data, puuid)
        #champion = player_data['championName']
        k = player_data['kills']
        d = player_data['deaths']
        a = player_data['assists']
        win = player_data['win']
        if d == 0:
            d=1
        kda = (k+a)/d
        kda_list.append(kda)
        if win == True:
            win_list.append(1)
        else:
            win_list.append(0)
    data['kda'].append(round(sum(kda_list)/len(kda_list)))
    data['winrate'].append(round(sum(win_list)/len(win_list)*100))
    return data
    
# Cette fonction permet de récupérer un autre identifiant d'un joueur
def get_encryptedSummonerId(summoner_name, region, api_key):
    api_url = (
        "https://" + 
        region +
        ".api.riotgames.com/lol/summoner/v4/summoners/by-name/" +
        summoner_name +
        "?api_key=" +
        api_key
    )
    resp = requests.get(api_url)
    player_info = resp.json()
    encryptedSummonerId = player_info['id']
    return encryptedSummonerId



# Cette fonction permet de modifier le format du rang d'un joueur ex : PLATINUM III -> platine
def change_rank(raw_rank):
    raw_rank = raw_rank.upper()
    if "IRON" in raw_rank:
        return 'iron'
    elif "BRONZE" in raw_rank:
        return 'bronze'
    elif "SILVER" in raw_rank:
        return 'silver'
    elif "GOLD" in raw_rank:
        return 'gold'
    elif "PLATINUM" in raw_rank:
        return 'platine'
    elif "EMERALD" in raw_rank:
        return 'emeraude'
    elif "DIAMOND" in raw_rank:
        return 'diamant'
    elif "MASTER" in raw_rank:
        return 'master'
```


```python
def get_champion_data(pseudo,mass_region,region,api_key,queue):


    df_data = pd.DataFrame(columns=['Party','Date','Type','Champion','Win',"Kills","Death","Assists","KDA"])
    puuid = get_puuid(pseudo,region,api_key)
    match_ids = get_match_ids_ranked(puuid, mass_region, api_key, queue)
    for match_id in match_ids:
        match_data = get_match_data(match_id, mass_region, api_key)
        #On vérifie si status est dans le dico match data, c'est à dire qu'on écarte le cas où il n'y a pas de data
        if 'status' in match_data:
            pass
        else:
            player_data = find_player_data(match_data, puuid)
            if player_data is not None:
                champion = player_data['championName']
                k = player_data['kills']
                d = player_data['deaths']
                a = player_data['assists']
                win = player_data['win']
                kda = (k + a) / (d or 1)
                date = match_data['info']['gameCreation']
                df_data['Win'] = df_data['Win'].astype(bool)
                df_new_row = pd.DataFrame([[match_id,date,queue,champion,win,k,d,a,kda]], columns=['Party','Date','Type','Champion','Win', "Kills", "Death", "Assists", "KDA"])
                df_data = pd.concat([df_data, df_new_row], axis=0, ignore_index=True)
            else :
                df_new_row = pd.DataFrame([[match_id,None,None,None,False,None,None,None,None]], columns=['Party','Date','Type','Champion','Win', "Kills", "Death", "Assists", "KDA"])
                df_data = pd.concat([df_data, df_new_row], axis=0, ignore_index=True)
    
    df_data_filtre = df_data.loc[df_data['Date'] > 1688973583000] 
    return df_data_filtre




def get_dataframe_champion(df_data):
    grouped_df = df_data.groupby("Champion")

    df_new = pd.DataFrame({
        "Champion": grouped_df["Champion"].first(),
        "Nombre de parties jouées": grouped_df.size(),
        "Fraction des parties jouées": grouped_df.size() / len(df_data),
        "Nombre de Win" : grouped_df["Win"].sum(),
        "Taux de victoire" : grouped_df["Win"].sum() / grouped_df.size(),
        "Nombre de kills": grouped_df["Kills"].sum(),
        "Nombre de morts": grouped_df["Death"].sum(),
        "Nombre d'assists": grouped_df["Assists"].sum(),
        "Moyenne du KDA": grouped_df["KDA"].mean()
    })

    df_new = df_new.sort_values(by="Nombre de parties jouées", ascending=False)
    return df_new





# Cette fonction permet de récupérer le personnage le plus joué ainsi son winrate associé
def get_data_champion(pseudo,region,api_key):
    print('Récupération des statistiques en classés')
    df_data_solo = get_champion_data(pseudo,mass_region,region,api_key,420)
    df_data_flex = get_champion_data(pseudo,mass_region,region,api_key,440)

    df_new_solo = get_dataframe_champion(df_data_solo)
    df_new_flex = get_dataframe_champion(df_data_flex)
    

    
    first_row_solo = df_new_solo.iloc[0]
    first_row_flex = df_new_flex.iloc[0]

    dict_solo = {"Champion_solo" : first_row_solo["Champion"], "Winrate_solo" : first_row_solo["Taux de victoire"]}
    dict_flex = {"Champion_flex" : first_row_flex["Champion"], "Winrate_flex" : first_row_flex["Taux de victoire"]}

    final_dict = {"solo" : dict_solo, "flex" : dict_flex}

    return final_dict
```


```python
api_key = 'RGAPI-d26162a7-7ea3-4af5-91b3-1087f97591e8'
```


```python
region = 'euw1'
mass_region = 'EUROPE'
```

# Interface graphique


```python
pip install Flask

```

    Requirement already satisfied: Flask in c:\users\samid\anaconda3\lib\site-packages (1.1.2)
    Requirement already satisfied: Werkzeug>=0.15 in c:\users\samid\anaconda3\lib\site-packages (from Flask) (1.0.1)
    Requirement already satisfied: click>=5.1 in c:\users\samid\anaconda3\lib\site-packages (from Flask) (7.1.2)
    Requirement already satisfied: Jinja2>=2.10.1 in c:\users\samid\anaconda3\lib\site-packages (from Flask) (2.11.2)
    Requirement already satisfied: itsdangerous>=0.24 in c:\users\samid\anaconda3\lib\site-packages (from Flask) (1.1.0)
    Requirement already satisfied: MarkupSafe>=0.23 in c:\users\samid\anaconda3\lib\site-packages (from Jinja2>=2.10.1->Flask) (1.1.1)
    Note: you may need to restart the kernel to use updated packages.
    

    
    [notice] A new release of pip available: 22.3.1 -> 23.2.1
    [notice] To update, run: python.exe -m pip install --upgrade pip
    


```python
from flask import Flask, render_template,request
app = Flask(__name__)

@app.route("/")
@app.route("/home")
def home():
    return render_template("homepage.html")
 
@app.route("/league")
def search():
    return render_template("league.html")

@app.route("/tft")
def datapage():
    return render_template("tft.html")

@app.route("/valorant")
def datapage2():
    return render_template("valorant.html")


@app.route('/submit',methods = ['POST', 'GET'])
def submit():
    print('je rentre dans submit')
    if request.method == 'POST':
        pseudo = request.form['nm']
        return f"Login successfully by POST method, Hello {pseudo}"
    else:
        pseudo = request.args.get('nm')
        data_player_ranked = get_data_champion(pseudo,region,api_key)
        data_player = get_player_data(pseudo,region,api_key)
        #print('bouton activé')
        print('le pseudo du site')
        print(pseudo)
        if pseudo:
            #data_player_ranked = get_data_champion(pseudo,region,api_key)
            print(data_player_ranked)
            #data_player = get_player_data(pseudo,region,api_key)
            icone = data_player['icon_id'][0]
            rank_flex = data_player['rank_flex']
            rank_solo = data_player['rank_solo']
            rank_flex_icon = change_rank(data_player['rank_flex'])
            rank_solo_icon = change_rank(data_player['rank_solo'])
            summoner_level = data_player['summoner_level'][0]
                
            champ_solo = data_player_ranked['solo']['Champion_solo']
            champ_flex =data_player_ranked['flex']['Champion_flex']
            winrate_solo = round(data_player_ranked['solo']['Winrate_solo'] * 100)
            winrate_flex = round(data_player_ranked['flex']['Winrate_flex'] * 100)
            
            
            print(champ_solo,champ_flex,winrate_solo,winrate_flex)
            print('pseudo=',pseudo)
            print('lvl=',summoner_level)
            print('rank solo',rank_solo)
            print('rank flex',rank_flex)
            print('icone solo',rank_solo_icon)
            print('icone flex',rank_flex_icon)
            print('icone joueur',icone)
            print(data_player_ranked)
            return render_template("datapage.html",pseudo=pseudo,summoner_level=summoner_level,icone=icone,rank_flex=rank_flex,rank_solo=rank_solo,rank_flex_icon=rank_flex_icon,rank_solo_icon=rank_solo_icon,champ_solo=champ_solo,champ_flex=champ_flex,winrate_solo=winrate_solo,winrate_flex=winrate_flex)
        else:
            return "Aucun pseudo n'a été spécifié dans la demande GET."
   
if __name__ =="__main__":  
    app.run()

