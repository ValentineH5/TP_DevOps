# TP_DevOps

## TP partie 01 - Docker

### 1-1 Pour quelle raison est-il préférable de faire tourner le conteneur avec un drapeau -e pour donner les variables d'environnement plutôt que de les mettre directement dans le Dockerfile?

Pour des questions de sécurité, il vaut mieux utiliser un drapeau -e pour les variables d'environnement plutôt que de les mettre directement dans le Dockerfile. Cela évite qu'elles soient directement visibles dans le Dockerfile. Cela permet également de pouvoir modifier les variables d'environnement sans devopir reconstruire toute l'image Docker.

### 1-2 Pourquoi avons-nous besoin d'un volume pour être attaché à notre conteneur postgres?

Nous avons besoin d'un volume attaché à notre conteneur pour persister les données durablement. Sans le volume, lorsque le conteneur est arrêté ou supprimé, les données sont supprimées. 

### 1-3 Documentez votre conteneur de base de données essentiels: commandes et Dockerfile.

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

### 1-4 Pourquoi avons-nous besoin d'une construction en plusieurs étapes? Et expliquez chaque étape de ce dossier.

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

### 1-5 Pourquoi utiliser un reverse proxy ?

Un reverse proxy reçoit les requêtes des utilisateurs et les transmet au backend. Il permet de centraliser les accès, de sécuriser l’application et de faciliter la gestion du trafic.

### 1-6 Pourquoi Docker Compose est-il important ?

Docker Compose permet de gérer plusieurs conteneurs avec un seul fichier. Il facilite leur création, leur configuration, leur démarrage et leur arrêt.

### 1-7 Document your docker-compose most important commands

- docker compose up -d : démarre les services en arrière-plan.
- docker compose down : arrête et supprime les conteneurs et réseaux créés.
- docker compose build : construit les images.
- docker compose ps : affiche l’état des services.
- docker compose logs : affiche les logs.
- docker compose restart : redémarre les services.

### 1-8 Document your docker-compose file

Le fichier docker-compose.yml définit les trois services de l’application :

- Database : héberge la base PostgreSQL et utilise un volume pour conserver les données.
- Backend : exécute l’API Java et communique avec la base de données sur le réseau Docker.
- HTTPD : sert de reverse proxy et expose l’application sur le port 80.

Les directives : 
- build : construire les images
- networks : connecter les services
- depends_on : définir leurs dépendances
- ports : exposer les ports
- volumes : conserver les données

### 1-9 Document your publication commands and published images in Docker Hub

Les commandes principales sont :

- docker login : se connecte à Docker Hub.
- docker tag my-database USERNAME/my-database:1.0 : ajoute un nom et une version à l’image.
- docker push USERNAME/my-database:1.0 : publie l’image sur Docker Hub.

Les images publiées peuvent être consultées dans le Docker Hub.

### 1-10 Pourquoi publier les images sur un dépôt en ligne ?

Un dépôt en ligne permet de partager les images Docker, de les récupérer sur d’autres machines et de faciliter le déploiement de l’application. Il permet également de conserver différentes versions des images.


## TP partie 02 - Github Actions

### 2-1 What are testcontainers?

Testcontainers est une bibliothèque Java qui permet de lancer des conteneurs Docker pendant les tests automatisés. Dans ce TP, elle permet d’utiliser une base de données PostgreSQL pour tester l’application.

### 2-2 For what purpose do we need to use secured variables ?

Les variables sécurisées permettent de stocker les informations sensibles, comme les identifiants Docker Hub et les tokens, sans les afficher dans le code source.

### 2-3 Why did we put needs: build-and-test-backend on this job? Maybe try without this and you will see!

needs: test-backend permet de lancer la construction des images Docker uniquement si les tests du backend ont réussi. Cela évite de publier une application qui ne fonctionne pas.

### 2-4 For what purpose do we need to push Docker images?

Le push permet d’envoyer les images Docker sur Docker Hub afin de pouvoir les récupérer et les déployer sur un serveur.

        
## TP part 03 - Ansible

### 3-1 Document your inventory and base commands

L’inventaire setup.yml contient les informations des serveurs distants : l’adresse IP, l’utilisateur SSH et la clé privée.

Les commandes principales sont :
- ansible all -i inventories/setup.yml -m ping : vérifie la connexion au serveur.
- ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*" : récupère les informations sur le système d’exploitation.
- ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become : désinstalle Apache2.

### 3-2 Document your playbook

Le playbook permet d'automatiser les tâches à effectuer sur un serveur. Les tâches sont définies dedans en yaml.
Il permet de tester la connexion, installer Docker et démarrer son service. On utilise des rôles pour mieux organiser et réutiliser ces tâches.
Pour exécuter le playbook, on utilise la commande : ansible-playbook -i inventories/setup.yml playbook.yml

### 3-3 Document your docker_container tasks configuration

On crée les tasks suivants :
- docker : installe Docker sur le serveur.
- network : crée le réseau Docker.
- database : démarre le conteneur de base de données avec ses variables d’environnement.
- app : démarre l’application et configure sa connexion à la base de données.
- proxy : démarre le proxy HTTPD pour rendre l’application accessible.

### 3-4 Is it really safe to deploy automatically every new image on the hub? Explain. What can I do to make it more secure?

Non, ce n’est pas totalement sûr. Une nouvelle image peut contenir des bugs ou des failles de sécurité.

Pour sécuriser le déploiement, il faut :
- Exécuter des tests automatiques avant le déploiement.
- Scanner les images pour détecter les vulnérabilités.
- Utiliser des versions précises des images plutôt que le tag latest.
- Protéger les secrets avec GitHub Secrets ou Ansible Vault.
- Prévoir un moyen de revenir à une version précédente en cas de problème.