# TP-MongoDB-ReplicaSet

___

## 0. Prérequis

___

- Avoir [Docker](https://docs.docker.com/get-docker/) installé sur votre machine
- Cloner le projet

```bash
git clone https://github.com/Dxvy/TP-Mongo-Replica.git
```

## 1. Configuration de MongoDB en mode ReplicaSet

___

Pour mettre en place les instances de MongoDB en mode ReplicaSet, nous allons employer le fichier `docker-compose.yml` qui décrit la configuration de 3 instances de MongoDB. 
<br> Il vous suffit d'exécuter la commande ci-dessous pour démarrer ces instances de MongoDB.
```bash
docker-compose up -d
```

Ce qui va créer 3 conteneurs Docker avec les noms suivants:

- mongo1
- mongo2
- mongo3

Pour s'assurer que les instances MongoDB ont été correctement lancées, il est possible de s'y connecter en utilisant la commande suivante :
```bash
docker exec -it mongo1 bash
```

_Pour les instances mongo2 et mongo3, il suffit de remplacer `mongo1` par `mongo2` ou `mongo3` dans la commande
ci-dessus._

Ensuite, on accède à l'instance MongoDB en utilisant la commande suivante :
```bash
mongosh
```

Par la suite, on peut vérifier que les instances sont correctement configurées en mode ReplicaSet en exécutant les commandes suivantes :

```bash
rs.status()
```

## 2. Génération de Fausses Données

___

Pour créer des données factices, on peut se servir du script `data_generation.py` situé dans le répertoire `scripts`. 
<br> Avant cela, il est nécessaire de transférer le script vers le conteneur `mongo1` en utilisant la commande suivante (dans notre terminal local, pas dans le conteneur) :

```bash
docker cp .\scripts mongo1:usr/src
```

_On copie le dossier entier, car il contient également un autre script qui nous sera utile plus tard._

On se connecte alors au conteneur `mongo1`, et il va nous falloir installer `python3` et `pip` pour pouvoir exécuter le
script.

```bash
apt-get update
apt-get install python3
apt-get install python3-pip
```

Ensuite, on installe les dépendances nécessaires pour les scripts.

```bash
pip3 install pymongo
pip3 install faker
```

On se déplace ensuite vers le dossier `usr/src/scripts` pour exécuter le script.

```bash
cd usr/src/scripts
python3 data_generation.py
```

Ceci va nous générer une liste de 100 utilisateurs avec leurs informations respectives.

## 3. Manipulations via la CLI MongoDB

___

Il est nécessaire de débuter par l'insertion des données générées précédemment dans la base de données.
<br>Pour cela, on va effectuer la commande suivante :

```bash
mongoimport --db db_cli --collection users --file users.json --jsonArray
```

_Cela permettra d'insérer les données dans la collection `users` de la base de données `db_cli`. 
<br>Si la base de données ou la collection n'existent pas, elles seront créées automatiquement. 
<br>Le chemin du fichier `users.json` peut être différent en fonction de l'endroit où vous exécutez la commande._

Maintenant que notre base de données est remplie, nous pouvons effectuer des requêtes pour récupérer des informations. Pour ce faire, nous retournons dans le `mongosh` pour exécuter les requêtes.

Avant de saisir une commande, il est essentiel de sélectionner la base de données `db_cli` en utilisant la commande suivante:

```bash
use db_cli
```

### Insertion - Nouvel utilisateur

___

```bash
db.users.insertOne({
    "name": "John Doe",
    "email": "johndoe@example.com",
    "age": 25,
    "createdAt": "2024-01-12T10:29:54"
})
```

### Lecture - Tous les utilisateurs ayant plus de 30 ans

___

```bash
db.users.find({ "age": { $gt: 30 } })
```

### Mise à jour - Augmenter l'âge de tous les utilisateurs de 5 ans

___

```bash
db.users.updateMany({}, { $inc: { "age": 5 } })
```

### Suppression - Supprimer un utilisateur spécifique

___

```bash
db.users.deleteOne({ "name": "John Doe" })
```

## 4. Automatisation avec Python

___

Pour automatiser les opérations précédentes, le script `crud_automatisation.py` situé dans le dossier `scripts` peut être utilisé. 
<br>Vous l'avez déjà copié dans le conteneur `mongo1` lors de la génération des données, il vous suffit donc de vous rendre dans le dossier `usr/src/scripts` et d'exécuter le script.

```bash
cd usr/src/scripts
python3 crud_automatisation.py
```

Après l'exécution du script, vous pouvez vérifier que les opérations ont été effectuées avec succès. Tout d'abord, une liste des utilisateurs trouvés sera affichée, puis vous pourrez 
vérifier dans le `mongosh` si les manipulations ont été correctement effectuées. <br>Vous trouverez une nouvelle base de données ainsi qu'une nouvelle collection `users`, contenant les mêmes utilisateurs 
que la base de données précédente, car les données ont été extraites du fichier `users.json`. <br>Il vous suffira ensuite d'examiner le contenu du script pour visualiser les manipulations réalisées.

## 5. Différences entre les manipulations via la CLI et via Python

___

Les actions réalisées via la ligne de commande (CLI) et via Python sont essentiellement les mêmes. La différence réside dans le fait qu'avec Python, il est possible d'automatiser les opérations, 
ce qui permet de les répéter autant de fois que nécessaire sans avoir à saisir manuellement les commandes à chaque fois. <br> Cela permet un gain de temps et une meilleure reproductibilité des manipulations. 
De plus, il est envisageable d'ajouter une gestion des erreurs pour les opérations via Python, ce qui n'est pas faisable avec la CLI. 

<br> Cependant, la CLI est plus rapide pour des manipulations ponctuelles et convient davantage pour des tâches simples. <br> Pour des manipulations plus complexes, l'utilisation de Python est préférable, 
mais il est nécessaire de définir à l'avance ce qui doit être accompli, car le script devra être écrit en conséquence.

## 6. Difficultés rencontrées

___

La principale difficulté rencontrée résidait dans la configuration du Docker-Compose pour mettre en place les instances MongoDB en mode ReplicaSet. Ce processus a demandé de nombreuses recherches sur des 
forums et des documentations pour en comprendre le fonctionnement. 

<br> Une autre difficulté rencontrée était liée à l'utilisation des scripts Python pour manipuler les données dans la base de données. Initialement, j'avais envisagé une approche différente en voulant utiliser 
un Dockerfile pour exécuter les scripts, installer Python et pip, mais je n'ai jamais réussi à le démarrer lors du lancement du Docker Compose. Finalement, j'ai trouvé une solution plus simple en copiant directement
les scripts dans le conteneur et en les exécutant à partir de là.