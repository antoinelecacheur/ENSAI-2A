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

