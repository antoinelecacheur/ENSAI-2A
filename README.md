# Retours sur les TPs, aide à la compréhension des librairies et config git

[Par ici](https://github.com/antoinelecacheur/ENSAI-2A/tree/master/retourtp)

# Configuration des postes

## Git Bash

Si vous rencontrez un problème du type "Failed to connect to github.com port 443: Connection refused" avec la commande git clone, il faut configurer le proxy sur git bash.
```
git config --global http.proxy http://pxcache-02.ensai.fr:3128
```

Vous pouvez également configurer votre identité auprès de git :
```
git config --global user.name "Prénom Nom"
git config --global user.email "adresse@mail.fr"
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
- Une fois que vous avez fait cette étape et que tout est bon, vous pouvez alors cliquer sur install requirements, ça devrait fonctionner.. !

# TP 1

## Présentation

:arrow_forward: <a href="https://antoinelecacheur.github.io/ENSAI-2A/index.html" target="_blank">Diapo TP1</a>

:open_file_folder: <a href="https://github.com/HealerMikado/2019Ensai_complement-info_TP1" target="_blank">Sujet et code du tp</a>

# Accompagnement Séance 1
:arrow_forward: <a href="https://antoinelecacheur.github.io/ENSAI-2A/brewerydb.html" target="_blank">Diapo Projet Info</a>

# TP 2

## Présentation
:arrow_forward: <a href="https://antoinelecacheur.github.io/ENSAI-2A/tp2.html" target="_blank">Diapo TP2</a>

:open_file_folder: <a href="https://github.com/HealerMikado/2019Ensai_complement_info_TP2" target="_blank">Sujet et code du tp</a>
## Data Rennes Metropole

L'<a href="https://data.rennesmetropole.fr/explore/dataset/equipement-accessibilite-arrets-bus/api/" target="_blank">url</a> des données géographiques du réseau STAR avec les arrêts physiques.


# TP 4

## Présentation
:arrow_forward: <a href="https://antoinelecacheur.github.io/ENSAI-2A/tp4.html" target="_blank">Diapo TP4</a>

:open_file_folder: <a href="https://foad-moodle.ensai.fr/course/view.php?id=11#section-5" target="_blank">Sujet et code du tp</a>

## Précisions pour le TP

### Configuration de git bash (encore..!)
- Configurer vos infos personnelles (comme proposé plus haut), attention il y a une erreur dans le sujet du TP il faut bien configurer `user.email` et `user.name`.

```
git config --global user.name "Prénom Nom"
git config --global user.email "adresse@mail.fr"
```

- Configurer un éditeur de texte au cas où vous oubliez de mettre un message de commit, pour éviter de vous retrouver à lutter contre vim

```
git config --global core.editor "'\\filer-thinapps.domensai.ecole\thinapps\Notepad64\Notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

#### Raccourcis clavier git bash :

- ctrl + inser = copier le texte sélectionné dans git bash
- shift + inser = coller du texte dans git bash
- Flèches haut - bas = parcourir l'historique des lignes de commandes saisies

### Fonctionnalité - Authentification : 4.3
- Dans compte_service, ajouter une méthode d'authentification qui prend en paramètres pseudo et mot de passe (la méthode find_by_pseudo de la DAO vous simplifie le travail)
- Repartir de l'exemple de `creation_compte.py` pour implémenter `compte_authentification.py`
- Utilisez la méthode d'authentification que vous venez de créer pour vérifier que les informations saisies par l'utilisateur sont valides
- Testez votre méthode d'authentification dans `test_compte.py` avec un pseudo connu et un pseudo inconnu

### Gestion d'un conflit - Merge

Dans le point 5.6 du TP, pour faire apparaître la fenêtre de fusion du fichier :
- Clic droit sur le fichier contenant les conflits
- Sélectionner git > Resolve conflicts

