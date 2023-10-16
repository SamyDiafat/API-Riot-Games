# Utilisation de l'API de Riot Games

## I - Accéder aux identifiants d'un compte

Import de notre clef


```python
api_key = 'RGAPI-d26162a7-7ea3-4af5-91b3-1087f97591e8'
```


```python
api_key
```




    'RGAPI-d26162a7-7ea3-4af5-91b3-1087f97591e8'



On récupère ensuite une URL venant du site de Riot Games : https://developer.riotgames.com/apis

On s'interesse tout d'abord aux données d'un compte en particulier, on utilise ici l'API V4


```python
pseudo = 'Yomm'
```


```python
api_url = 'https://euw1.api.riotgames.com/lol/summoner/v4/summoners/by-name/' + pseudo
```


```python
api_url
```




    'https://euw1.api.riotgames.com/lol/summoner/v4/summoners/by-name/Yomm'



Afin d'avoir l'autorisation d'accès à l'API, nous devons ajouter notre clef à l'url


```python
api_url = api_url + '?api_key=' + api_key
```


```python
api_url
```




    'https://euw1.api.riotgames.com/lol/summoner/v4/summoners/by-name/Yomm?api_key=RGAPI-d26162a7-7ea3-4af5-91b3-1087f97591e8'



On importe ensuite la bibliothèque requests qui va nous permettre de faire des requêtes via python

La bibliothèque requests en Python est utilisée pour envoyer des requêtes HTTP vers des serveurs web et interagir avec des ressources en ligne. Elle permet de créer des applications qui peuvent effectuer des opérations telles que l'envoi de requêtes GET, POST, PUT, DELETE, etc., pour récupérer des données depuis des API, envoyer des données à des serveurs, télécharger des fichiers à partir d'Internet, et bien plus encore.


```python
import requests
```


```python
requests.get(api_url)
```




    <Response [200]>



La réponse nous permet de savoir l'état de la requête envoyée, 200 correspond à une requête réussi !


```python
resp = requests.get(api_url)
player_info = resp.json()
player_info
```




    {'id': 'sowIMP6jk8jfAEMsUR9W9TwUlqcHLpk-sGD727LdzXTJKJU',
     'accountId': 'XEOycCgqv9jSoYu25_24LPzueZGvKrcdUX7yUy_p-6vrxw',
     'puuid': 'u-bz6k4v2VzDEVkG5brW_NBGfYmjCGstDEoK0ruij3BpbwkhseE4knsJZH_Sx3NjnSzQy8iv60vTjA',
     'name': 'Yomm',
     'profileIconId': 608,
     'revisionDate': 1697225190000,
     'summonerLevel': 184}



La méthode .json() est appelée, et le contenu JSON de la réponse est automatiquement analysé et converti en une structure de données Python

On voit que player_info est un dictionnaire contenant plusieurs informations


```python
player_info.keys()
```




    dict_keys(['id', 'accountId', 'puuid', 'name', 'profileIconId', 'revisionDate', 'summonerLevel'])



On peut par exemple obtenir l'ID associée à un pseudo


```python
player_info['id']
```




    'sowIMP6jk8jfAEMsUR9W9TwUlqcHLpk-sGD727LdzXTJKJU'



## II - Accéder aux données d'un compte

Maintenant que nous avons accès aux différents identifiants d'un compte, nous pouvons faire des requêtes à l'API qui nous interesse 

Par exemple, nous allons utiliser l'API MATCH V5 pour obtenir des données relatives aux parties de jeu d'un joueur


```python
puuid = player_info['puuid']
puuid
```




    'u-bz6k4v2VzDEVkG5brW_NBGfYmjCGstDEoK0ruij3BpbwkhseE4knsJZH_Sx3NjnSzQy8iv60vTjA'




```python
api_url = 'https://europe.api.riotgames.com/lol/match/v5/matches/by-puuid/u-bz6k4v2VzDEVkG5brW_NBGfYmjCGstDEoK0ruij3BpbwkhseE4knsJZH_Sx3NjnSzQy8iv60vTjA/ids?start=0&count=20'
```

Pour ajouter la clef dans l'url, il faut cette fois-ci mettre un & car il y a déjà un argument avant


```python
api_url = api_url + '&api_key=' + api_key
```


```python
api_url
```




    'https://europe.api.riotgames.com/lol/match/v5/matches/by-puuid/u-bz6k4v2VzDEVkG5brW_NBGfYmjCGstDEoK0ruij3BpbwkhseE4knsJZH_Sx3NjnSzQy8iv60vTjA/ids?start=0&count=20&api_key=RGAPI-d26162a7-7ea3-4af5-91b3-1087f97591e8'




```python
resp = requests.get(api_url)
resp
```




    <Response [200]>




```python
match_info = resp.json()
```


```python
match_info
```




    ['EUW1_6620612418',
     'EUW1_6620591242',
     'EUW1_6620117730',
     'EUW1_6616569027',
     'EUW1_6616516559',
     'EUW1_6615412808',
     'EUW1_6611335474',
     'EUW1_6611256844',
     'EUW1_6610248381',
     'EUW1_6610147341',
     'EUW1_6609656368',
     'EUW1_6607254752',
     'EUW1_6607085859',
     'EUW1_6606463762',
     'EUW1_6603360397',
     'EUW1_6603288425',
     'EUW1_6603210097',
     'EUW1_6601582307',
     'EUW1_6601554856',
     'EUW1_6600766557']



Nous avons réussi à récupérer les identifiants des 20 dernières parties d'un joueur

On va maintenant s'intéresser à une partie en particulier

On récupère l'identifiant de la dernière partie


```python
last_game = match_info[0]
last_game
```




    'EUW1_6620612418'



Puis l'url sur le site de Riot Games


```python
api_url = 'https://europe.api.riotgames.com/lol/match/v5/matches/' + last_game
api_url
```




    'https://europe.api.riotgames.com/lol/match/v5/matches/EUW1_6620612418'




```python
api_url = api_url + '?api_key=' + api_key
```


```python
api_url
```




    'https://europe.api.riotgames.com/lol/match/v5/matches/EUW1_6620612418?api_key=RGAPI-d26162a7-7ea3-4af5-91b3-1087f97591e8'




```python
resp = requests.get(api_url)
resp
```




    <Response [200]>








    dict_keys(['metadata', 'info'])



Les meta données contient l'identifiant des joueurs dans la partie





On obtient ainsi toutes les données du joueur




On peut par exemple regarder le champion joué par notre joueur


```python
matchdata['info']['participants'][player_index]['championName']
```




    'TwistedFate'



On regarde les données de fin de partie correspondant au joueur numéro 1


```python
matchdata['info']['participants'][player_index]['totalDamageDealtToChampions']
```




    23061



## III - Création de fonctions permettant de récupérer des données simplement


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
    # we need to add this "while" statement so that we continuously loop until it's successful
    while True:
        resp = requests.get(api_url)
        
        # whenever we see a 429, we sleep for 10 seconds and then restart from the top of the "while" loop
        if resp.status_code == 429:
            print("Rate Limit hit, sleeping for 10 seconds")
            time.sleep(10)
            # continue means start the loop again
            continue
            
        # if resp.status_code isn't 429, then we carry on to the end of the function and return the data
        resp = requests.get(api_url)
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
    # we need to add this "while" statement so that we continuously loop until it's successful
    while True:
        resp = requests.get(api_url)
        
        # whenever we see a 429, we sleep for 10 seconds and then restart from the top of the "while" loop
        if resp.status_code == 429:
            print("Rate Limit hit, sleeping for 10 seconds")
            time.sleep(10)
            # continue means start the loop again
            continue
            
        # if resp.status_code isn't 429, then we carry on to the end of the function and return the data
        resp = requests.get(api_url)
        match_data = resp.json()
        return match_data 
```


```python
pseudo = 'Yomm'
region = 'euw1'
mass_region = 'EUROPE'

```


```python
get_puuid(pseudo,region,api_key)
```




    'u-bz6k4v2VzDEVkG5brW_NBGfYmjCGstDEoK0ruij3BpbwkhseE4knsJZH_Sx3NjnSzQy8iv60vTjA'




```python
puuid_player = get_puuid(pseudo,region,api_key)
id_matches = get_match_ids(puuid_player, mass_region, api_key)
print('Les identifiants des 20 dernières parties de',pseudo,'sont',id_matches)
```

    Les identifiants des 20 dernières parties de Yomm sont ['EUW1_6620612418', 'EUW1_6620591242', 'EUW1_6620117730', 'EUW1_6616569027', 'EUW1_6616516559', 'EUW1_6615412808', 'EUW1_6611335474', 'EUW1_6611256844', 'EUW1_6610248381', 'EUW1_6610147341', 'EUW1_6609656368', 'EUW1_6607254752', 'EUW1_6607085859', 'EUW1_6606463762', 'EUW1_6603360397', 'EUW1_6603288425', 'EUW1_6603210097', 'EUW1_6601582307', 'EUW1_6601554856', 'EUW1_6600766557']
    

=


```python
# Fonction permettant de récupérer les données d'un joueur à partie des données d'une partie et d'un ID
def find_player_data(match_data, puuid):
    participants = match_data['metadata']['participants']
    player_index = participants.index(puuid)
    player_data = match_data['info']['participants'][player_index]
    return player_data
```


    

Si l'on veut regarder uniquement une info


```python
print(data)
data['kda']
```

    {'champion': ['TwistedFate', 'TwistedFate', 'TwistedFate', 'Talon', 'Taliyah', 'Taliyah', 'Taliyah', 'Taliyah', 'KSante', 'Taliyah', 'Taliyah', 'Gangplank', 'Gangplank', 'Taliyah', 'Taliyah', 'Taliyah', 'Gangplank', 'Gangplank', 'Gangplank', 'Gangplank'], 'kills': [4, 7, 6, 3, 2, 1, 4, 18, 1, 8, 4, 3, 5, 2, 5, 15, 14, 9, 5, 14], 'deaths': [11, 6, 11, 19, 11, 14, 10, 10, 9, 12, 4, 14, 13, 13, 9, 13, 12, 17, 16, 8], 'assists': [11, 8, 7, 10, 6, 4, 8, 10, 4, 9, 7, 3, 8, 7, 7, 14, 9, 7, 5, 3], 'kda': [5.0, 8.333333333333334, 6.636363636363637, 3.526315789473684, 2.5454545454545454, 1.2857142857142856, 4.8, 19.0, 1.4444444444444444, 8.75, 5.75, 3.2142857142857144, 5.615384615384615, 2.5384615384615383, 5.777777777777778, 16.076923076923077, 14.75, 9.411764705882353, 5.3125, 14.375], 'win': ['Défaite', 'Défaite', 'Défaite', 'Victoire', 'Défaite', 'Défaite', 'Victoire', 'Victoire', 'Défaite', 'Défaite', 'Victoire', 'Défaite', 'Défaite', 'Défaite', 'Victoire', 'Défaite', 'Victoire', 'Défaite', 'Défaite', 'Victoire']}
    




    [5.0,
     8.333333333333334,
     6.636363636363637,
     3.526315789473684,
     2.5454545454545454,
     1.2857142857142856,
     4.8,
     19.0,
     1.4444444444444444,
     8.75,
     5.75,
     3.2142857142857144,
     5.615384615384615,
     2.5384615384615383,
     5.777777777777778,
     16.076923076923077,
     14.75,
     9.411764705882353,
     5.3125,
     14.375]



## IV Gestion des rate limits

Nous n'avons droit qu'à 100 requête toutes les 2 minutes, pour les grosses exctraction, il faut gérer les cas où l'on obtient l'erreur 429 (limite dépasée), mettre un temps d'arrêt à nos fonctions. 
Voici un exemple permettant de gérer cela.



```python
import time


def get_match_data2(match_id, mass_region, api_key):
    api_url = (
        "https://" + 
        mass_region + 
        ".api.riotgames.com/lol/match/v5/matches/" +
        match_id + 
        "?api_key=" + 
        api_key
    )
    
    # we need to add this "while" statement so that we continuously loop until it's successful
    while True:
        resp = requests.get(api_url)
        
        # whenever we see a 429, we sleep for 10 seconds and then restart from the top of the "while" loop
        if resp.status_code == 429:
            print("Rate Limit hit, sleeping for 10 seconds")
            time.sleep(10)
            # continue means start the loop again
            continue
            
        # if resp.status_code isn't 429, then we carry on to the end of the function and return the data
        match_data = resp.json()
        return match_data   
```
