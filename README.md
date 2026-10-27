# TP_DevOps

## TP partie 01 - Docker

*1-1 Pour quelle raison est-il préférable de faire tourner le conteneur avec un drapeau -e pour donner les variables d'environnement plutôt que de les mettre directement dans le Dockerfile?*

Pour des questions de sécurité, il vaut mieux utiliser un drapeau -e pour les variables d'environnement plutôt que de les mettre directement dans le Dockerfile. Cela évite qu'elles soient directement visibles dans le Dockerfile. Cela permet également de pouvoir modifier les variables d'environnement sans devopir reconstruire toute l'image Docker.

*1-2 Pourquoi avons-nous besoin d'un volume pour être attaché à notre conteneur postgres?*

Nous avons besoin d'un volume attaché à notre conteneur pour persister les données durablement. Sans le volume, lorsque le conteneur est arrêté ou supprimé, les données sont supprimées. 

*1-3 Documentez votre conteneur de base de données essentiels: commandes et Dockerfile.*

image utilisée pour le Docker : postgres:17.2-alpine
pour se connecter au Adminer, il faut utiliser : 
* système : postgresSQL
* serveur : db_tp1
* utilisateur : usr
* mot de passe : pwd
* base de données : db

Pour initialiser la base de données, on a créé les scripts CreateScheme.sql et InsertData.sql qui sont copiés dans /docker-entrypoint-initdb.d/ dans le Dockerfile. CreateScheme créé les tables et InsertData les remplies. 

On crée le conteneur db_tp1 avec docker build -t db_tp1 .
On crée un réseau Docker app-network avec docker network create app-network
On démarre PostgreSQL avec docker run -d --name db_tp1 --network app-network db_tp1
On accède à Adminer sur le port 8090

Pour la persistance des données, on utilise un volume Docker nommé pgdata : 
créer le volume : volume create pgdata
démarrer le conteneur : docker run -d --name db_tp1 --network app-network -v pgdata:/var/lib/postgresql/data db_tp1

*1-4 Pourquoi avons-nous besoin d'une construction en plusieurs étapes? Et expliquez chaque étape de ce dossier.*

Les différentes étapes permettent de séparer la compilation et l'exécution de l'application.

Les différentes étapes : 
1. Build stage :
- utilise une image Java JDK 21 et installe Maven pour compiler le code
- définition du répertoire de travail
- copie les fichiers nécessaires 
- compile l’application Spring Boot
2. Run stage : 
- utilise une image pour exécuter l’application
- copie le fichier JAR généré lors de la première étape
- lance l’application Spring Boot lorsqu'on démarre le conteneur

## TP partie 02 - Github Actions

*2-1 What are testcontainers?*


*2-2 For what purpose do we need to use secured variables ?*

*2-3 Why did we put needs: build-and-test-backend on this job? Maybe try without this and you will see!*


        
## TP part 03 - Ansible

*3-1 Document your inventory and base commands*

L’inventaire setup.yml contient les informations des serveurs distants : l’adresse IP, l’utilisateur SSH et la clé privée.

Les commandes principales sont :
- ansible all -i inventories/setup.yml -m ping : vérifie la connexion au serveur.
- ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*" : récupère les informations sur le système d’exploitation.
- ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become : désinstalle Apache2.

*3-2 Document your playbook*

Le playbook permet d'automatiser les tâches à effectuer sur un serveur. Les tâches sont définies dedans en yaml.
Il permet de tester la connexion, installer Docker et démarrer son service. On utilise des rôles pour mieux organiser et réutiliser ces tâches.
Pour exécuter le playbook, on utilise la commande : ansible-playbook -i inventories/setup.yml playbook.yml

