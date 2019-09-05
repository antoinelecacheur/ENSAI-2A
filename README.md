# Configuration des postes

## Git Bash

Si vous rencontrez un problème du type "Failed to connect to github.com port 443: Connection refused" avec la commande git clone, il faut configurer le proxy sur git bash.
```
git config --global http.proxy http://pxcache-02.ensai.fr:3128
```

Vous pouvez également configurer votre identité auprès de git :
```
git config --global user.name "Antoine Lecacheur"
git config --global user.email "antoine.lecacheur@hotmail.fr"
```

## PyCharm
### Configure a Python Interpreter
- File > Settings | Dans la barre de recherche tapez <b>interpreter</b>. Vous devriez vous retrouver dans le menu <b>Project Interpreter</b> avec un menu déroulant vide. Dans les options du menu déroulant, cliquez sur <b>Show All</b>.
- Cliquez sur le "+" sur la droite, un nouveau menu apparaît, tout ce qui est préselectionné (New Environment, Location, Base interpreter) devrait être bien configuré. Cliquez sur ok pour valider, puis à nouveau sur ok une fois le chargement terminé. Après quelques minutes vous ne devriez plus avoir le message "Configure a Python Interpreter".

### Install Requirements
- Vous allez devoir installer les librairies python nécessaires à chaque TP. PyCharm vous proposera lui-même de les installer, mais avant ça il faut configurer le proxy de PyCharm pour l'ENSAI...
- File > Settings | Tapez dans la recherche <b>proxy</b>, vous devriez arriver dans le menu <b>HTTP Proxy</b>.
- Sélectionnez "Manual Proxy Configuration". Dans le champ "Host name" saisissez <b>pxcache-02.ensai.fr</b> et dans le champ "Port number" saisissez <b>3128</b>
- Vous pouvez vérifier que tout est bon en cliquant sur "check connection" et puis en saisissant ```https://www.google.com ```

# TP 1

## Présentation

:arrow_forward: <a href="https://antoinelecacheur.github.io/ENSAI-2A/index.html" target="_blank">Diapo TP1</a>

:open_file_folder: <a href="https://github.com/HealerMikado/2019Ensai_complement-info_TP1" target="_blank">Sujet et code du tp</a>

# Accompagnement Séance 1
:arrow_forward: <a href="https://antoinelecacheur.github.io/ENSAI-2A/brewerydb.html" target="_blank">Diapo Projet Info</a>

# TP 2

## Présentation

## Data Rennes Metropole

L'<a href="https://data.rennesmetropole.fr/explore/dataset/equipement-accessibilite-arrets-bus/api/" target="_blank">url</a> des données géographiques du réseau STAR avec les arrêts physiques.
