# Retour sur les TPs

## TP 1 - DAO : utilisation de psycopg2

* La couche DAO est celle où vous implémentez les méthodes de persistance des données. Pour faire simple, dans notre cas : on implémente toutes les méthodes qui exécutent les requêtes SQL pour modifier notre BDD.

* Pour les explications à suivre, il vaut mieux se référer au code du TP 4, car c'est le code que vous utiliserez pour débuter le projet. Par exemple avoir la classe compte_dao.py sous les yeux en même temps pour mieux comprendre.

### Connection et curseur

* Pour pouvoir exécuter les requêtes SQL, il faut être connecté à la BDD. Chaque méthode d'une classe DAO doit donc commencer par l'instruction suivante :

``` 
cur = AbstractDao.connection.cursor()
```

Cela permet de récupérer un objet cursor (de psycopg2) qui va gérer les connections et requêtes à la BDD.

### Exécution des requêtes et gestion de la réponse

Ensuite, le code diffère selon qu'on ne veut récupérer qu'un seul objet avec la requête SQL, ou plusieurs :

* Un seul objet :

``` python
# On exécute la requête SQL avec les paramètres voulus
cur.execute(
            "INSERT INTO account (pseudo, motdepasse) VALUES (%s, %s) RETURNING id;", (compte.pseudo, compte.motdepasse))

# Puis on stocke le résultat dans un objet python (ici l'insertion en base renvoie l'id associé)
# La méthode fetchone() permet simplement de récupérer la ligne renvoyée par la requête
compte.id = cur.fetchone()[0]
```

Autre exemple avec plusieurs paramètres retournés par la requête : 

``` python
# On sélectionne les champs qu'on veut récupérer avec la requête
cur.execute(
            "select id, pseudo, motdepasse from account where pseudo=%s", (pseudo,))

# On stocke la ligne trouvée dans found
            found = cur.fetchone()
            # On s'asure que found n'est pas vide (pas de ligne retournée par la requête)
            if found:
                # Puis on récupère les champs dans l'ordre qu'on les a spécifiés dans la requête : found[0] = id, found[1] = pseudo, etc
                return Compte(found[1], found[2], found[0])
```

- Plusieurs objets (on veut la liste des tous les agents du shield du TP 1) :
```python
# On exécute à nouveau notre requête
cur.execute(
    "select id, name, security_level from shield_agents")
    # Cette fois on récupère des tuples et on les transforme en objects ShieldAgent qu'on insère dans un tableau
    result = [ShieldAgent(id=item[0], name=item[1], security_level=item[2])
        # On ne se sert pas de fetchone(), mais de fetchall() pour récupérer toutes les lignes retournées par la requête, sur lesquelles on fait une boucle for
        for item in cur.fetchall()]
    return result
```

- Une fois ces étapes réalisées, on enregistre en base ou on annule la transaction si une erreur est soulevée (cette étape ne change jamais, le code que vous aurez vraiment à gérer est celui de l'exécution de requête et de gestion du résultat)
```python
    # la transaction est enregistrée en base
    AbstractDao.connection.commit()
except psycopg2.Error as error:
    # la transaction est annulée
    AbstractDao.connection.rollback()
    raise error
finally:
    cur.close()
```

- Pour un exemple de requête faisant intervenir une table d'association, voir la méthode find_all_agent_with_avengers de agent_dao du TP 1.

- Plus d'informations sur psycopg2, en particulier sur [l'objet cursor](http://initd.org/psycopg/docs/cursor.html)

## TP2 - Webservices : récupération et envoi de données
* Les webservices sont des ressources accessibles via des requêtes HTTP. Ils fonctionnent comme des boites noires : Ils disposent d'une API (application programming interface) pour laquelle l'on ne connait que l'entrée/sortie (Boite Noire)

* Par différentes requêtes (GET et POST pour la plupart), vous pouvez récupérer des données et échanger avec un serveur distant par sa couche Controller. Ces requêtes peuvent avoir différents type : changer l'état de la base de données par transmission de l'information coté DAO, récupération de l'état de la base de données, ou encore récupération de l'état du serveur par exemple (Healthcheck).

### Tester des requêtes avec Insomnia

Insomnia est un client [REST](https://fr.wikipedia.org/wiki/Representational_state_transfer) permettant justement d'effectuer des requêtes HTTP sur une URL. Il propose une interface graphique simple permettant de tester les requêtes que l'on va ensuite pouvoir intégrer dans le code Python par la suite. 

* Il implémente quasiment tous les types de [requêtes HTTP](https://www.tutorialspoint.com/http/http_requests.htm)

* Exemple avec une requête POST
![](https://codimd.s3.shivering-isles.com/demo/uploads/upload_2eace30fa67eefad0be33ce68d763c49.jpg)
  * Avec en rouge: l'ajout d'une nouvelle requête
  * Avec en bleu : l'url où effectuer la requête
  * Avec en vert : la zone où ajouter les éventuels paramètres
### Récupération de données avec Requests
Requests est la référence des librairies Python pour effectuer des requêtes HTTP. Son utilisation est assez simple bien que formalisée. 

#### Installation : 
Vous pouvez l'installer via pip
```
pip install requests
```
ou via Pycharm qui détectera les install Requirements. 
#### Utilisation : 
* Importez la librairie dans la classe de votre choix, la bonne pratique étant de s'inspirer du squelette projet du tp4.
```python
import requests
```
* Effectuez une configuration préalable permettant d'éviter le proxy de l'ensai, ne le mettez pas si vous êtes hors vm.
```python
proxies = {
                'http': http://pxcache-02.ensai.fr:3128,
                'https': http://pxcache-02.ensai.fr:3128
            }
```
* Ici c'est une étape importante qui changera d'une fois a l'autre: indiquez les différents paramètres de la requête (correspondant à ce que vous rentreriez dans Insomnia). (ici: C'est l'exemple du tp2)
```	python		
params = {
                'apikey': properties.api_key,
                'dataset': "statistiques-de-prets-de-dvd-en-2017-cesson-sevigne",
                "rows": 6288}
response = requests.get(
                'https://data.rennesmetropole.fr/api/records/1.0/search/'
                , proxies=proxies
                , params=params)
```
* Récuperez les données dans un objet python pour pouvoir les manipuler par la suite dans votre couche Business Object ! 
```	python	
data = response.json()['records']
```

### Créer son propre webservice avec la bibliothèque Flask

* Importez la librairie dans une Classe présente dans le dossier controller de votre projet
```	python	
from flask import Flask, request
```
* Mettez en place votre serveur dans une classe Server
    ```	python	
    app = Flask(__name__)
    ```
    [...]
    ```	python	
    if __name__ == "__main__":
        app.run()
    ```
* Définissez les différents "endpoints" de votre API en annotant vos méthodes qui seront appelées en effectuant des requêtes HTTP
    ```	python	
    @app.route("/healthCheck", methods=['GET'])
    def movieList():
    ```
    Ici par exemple si l'on fait une requête GET sur localhost:5000/healthCheck, on appelera la fonction healthCheck()
* Effectuez différents traitement métiers en appelant des méthodes objets de vos business objects
* Effectuez un retour pour la requête 
```	python	
""" retour par defaut (avec import json préalable) """
 return json.dumps({"result": "success"}) 
```

## TP 3 - Formats de données - Web Scraping
Il existe différents types de formats de données structurés non SQL, ils sont principalement utilisés pour l'échange de données et la sauvegarde de données 
### Différents formats de données 
* JSON : 
```
[ 
{
      "titre": "Star Wars : le retour du Jedi",
      "commune": "Cesson-Sevigne",
      "auteur": "Kershner Irvin",
      "code_insee": 35051.0,
      "nombre_de_prets_2017": 10.0
  },{ 
      "titre": "Blanche Neige et le chasseur",
      "commune": "Cesson-Sevigne",
      "auteur": "Sanders Rupert",
      "code_insee": 35051.0,
      "nombre_de_prets_2017": 10.0
  }
]
```
Format actuellement très utilisé avec le yml, il est surtout très commun dans le monde du web et des web services.
* XML
```
<Bibliotheque>
<CD> <titre>Star Wars : le retour du Jedi</titre>
      <commune> Cesson-Sevigne</commune>
      <auteur> Kershner Irvin</auteur>
      <code_insee> 35051.0</code_insee>
      <nombre_de_prets_2017> 10.0</nombre_de_prets_2017>
</CD>
<CD> <titre>Blanche Neige et le chasseur</titre>
      <commune> Cesson-Sevigne</commune>
      <auteur> Sanders Rupert</auteur>
      <code_insee> 35051.0</code_insee>
      <nombre_de_prets_2017> 10.0</nombre_de_prets_2017>
</CD>
</Bibliotheque>
```
Format plus verbeux utilisé dans beaucoup de configuration mais également pour l'échange des données (comme le JSON)
* CSV
```
titre;commune;auteur;code_insee;nombre_de_prets_2017
Star Wars : le retour du Jedi;Cesson-Sevigne;Kershner Irvin;35051.0;10.0
Blanche Neige et le chasseur ;Cesson-Sevigne;Sanders Rupert;35051.0;10.0
```	

Format très classique dans le monde de la donnée et de la sauvegarde. 

### Lire les données 
#### Lecture et écriture du csv en python

* Lecture
```python
import csv

#Ouvrir le fichier en lecture
with open('./test.csv', 'r') as csvfile:
    #Définition du reader
    csvreader = csv.reader(csvfile, delimiter=';')
    #Lecture ligne par ligne
    for row in csvreader:
        for column in row:
            print(column)
```

* Ecriture
 
```python
import csv

#Ouvrir le fichier en écriture
with open('./test2.csv', 'w',  newline='') as csvfile:
    #Définition de writer
    csvwriter = csv.writer(csvfile, delimiter=',',)
    #Ecriture d'une ligne
    csvwriter.writerow(['nom', 'prénom', 'age'])

```

#### Lecture et écriture du json en python
* Lecture
```python
import json

#Ouvrir le fichier en lecture
with open('./test.json', 'r', encoding="utf-8") as jsonfile:
    #Lecture du json en dictionnaire
    data = json.load(jsonfile)
    #Lecture ligne par ligne
    for row in data:
        # on ajoute une ligne dans le fichier et on enleve la donnée depuis le data
                fieldsValues = row['fields'].values()
                row.pop('fields')
	# puis on l'écrit (en fct de la conversion)
```	

* Ecriture

						
```python
import json

#Ouvrir le fichier en écriture
with open('./test2.json', 'r', encoding="utf-8") as jsonfile:
    #données : tableau d'entrées clé:valeur
    result = [dico1,dico2]
    #Lecture ligne par ligne
    jsonfile.write(json.dumps(result))
```
#### Lecture et écriture du xml en python

* Lecture
```python
import xml.etree.ElementTree as ET

#Ouvrir le fichier en lecture
with open('./test.xml', 'wb') as xmlfile:
    #données : utilisation d'un parseur xml de la librarie etree
    tree=ET.parse(xmlfile)
    #Lecture ligne par ligne
    root = tree.getroot()
    for child in root : 
        print(child.tag,child.attrib)
```

* Ecriture			

```python
from lxml import etree

#Ouvrir le fichier en écriture
with open('./test2.xml', 'wb') as xmlfile:
    #données : dico en entrée
    result = [dico1,dico2]
    xmlfile.write(etree.tostring(result, pretty_print=True))
```						

### Web Scraping avec Python
#### Rappel : Le webscraping c'est quoi? 
* Il s'agit littéralement d'extraction de données sur des sites web via un script ou un programme 
* C'est a dire un document balisé qui est donc parsable par différentes librairies afin de récupérer les informations que l'on veut.

* Beaucoup de librairies existent, nous avons présenté dans le tp3 la librairie lxhtml pour l'extraction de données depuis l'html

#### Exemple avec une page wikipédia
En utilisant request pour le requetage http on peut par exemple récupérer la liste des différents médaillés au judo comme ci-dessous qui sont sous différentes balises html respectives. 
```python
import requests
from lxml import html

pageContent=requests.get('https://en.wikipedia.org/wiki/List_of_Olympic_medalists_in_judo')
tree = html.fromstring(pageContent.content)

goldWinners=tree.xpath('//*[@id="mw-content-text"]/table/tr/td[2]/a[1]/text()')
silverWinners=tree.xpath('//*[@id="mw-content-text"]/table/tr/td[3]/a[1]/text()')
#bronzeWinner we need rows where there's no rowspan - note XPath
bronzeWinners=tree.xpath('//*[@id="mw-content-text"]/table/tr/td[not(@rowspan=2)]/a[1]/text()')
medalWinners=goldWinners+silverWinners+bronzeWinners

medalTotals={}
for name in medalWinners:
    if medalTotals.has_key(name):
        medalTotals[name]=medalTotals[name]+1
    else:
        medalTotals[name]=1

for result in sorted(
        medalTotals.items(), key=lambda x:x[1],reverse=True):
        print '%s:%s' % result
```


## TP 4 - Git : étapes de configuration sur les postes

### Générer sa clef SSH (reprise de la configuration partie 3 du TP)

* Lancez Git bash (Menu démarrer > tapez git bash dans la barre de recherche).

* La première chose à faire est de configurer le proxy si ce n’est pas déjà fait

``` 
git config --global http.proxy http://pxcache-02.ensai.fr:3128
git config --global https.proxy http://pxcache-02.ensai.fr:3128
```

* Puis de configurer vos informations personnelles

``` 
git config --global user.email VOTRE EMAIL
git config --global user.name PRENOM NOM
```

 En remplaçant VOTRE EMAIL par votre email et PRENOM NOM par vos prénoms et noms.
 

* Changez l’éditeur git par défaut par notepad++ (sinon vous allez utiliser vim)

``` 
git config --global core.editor "'\\filer-thinapps.domensai.ecole\thinapps\Notepad64\Notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

Cette config est conservée vous n’avez à la faire qu’une fois sur les machines de l’ENSAI. Si vous utilisez git chez vous, il faudra reconfigurer votre client (potentiellement en installer un)

* Ensuite, générez votre couple de clé ssh

``` 
ssh-keygen -t rsa -b 2048
```

<p>Ceci va permettre de faciliter de la communication entre gitlab et vous. En effet le coupleque vous allez générer va servir à prouver que vous êtes bien la personne que vous dites être, et ainsi ne plus à avoir à saisir votre couple id/mdp.</p>

* Tapez entrée pour chaque question que l’on vous pose.
* Sous Gitlab, allez dans vos réglages : https://gitlab.com/profile. Dans le menu à gauche allez dans SSH Keys. Vous avez un champ texte sous Key, vous allez mettre le contenu de votre clé.
* Votre clé devrait se trouver ici : `P:\ssh_key\id_rsa` . Pour l’afficher, il faut que dans l'explorateur de fichier vous cliquiez sur Organiser > Options des dossiers et de recherche > Affichage > Afficher les fichiers, dossiers et lecteurs cachés.
* Sélectionnez le fichier id_rsa.pub (celui qui s'ouvre pas défaut avec Document Microsoft Publisher) puis ouvrir avec > autres programmes > bloc-notes.
* Copiez tout le contenu du fichier (raccourcis claviers ctrl + a puis ctrl + c).

* Dans gitlab, collez la clé puis cliquez sur Add key. La clé ssh est conservée. Vous n’aurez donc à la configurer qu’une fois à l’Ensai. Par contre si vous travaillez avec un ordinateur perso, il faudra créer une clé ssh dessus (et l’ajouter sur gitlab).

* Une fois cette configuration effectuée, vous pouvez reprendre le fil du TP.
