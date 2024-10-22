# TP DevOps

## Question TP part 01 – Docker

### Question 1: Why should we run the container with a flag -e to give the environment variables?
	
Le flag -e est utilisé pour passer des variables d'environnement au conteneur, ce qui permet de configurer l'application (par exemple, les identifiants de connexion ou les paramètres spécifiques) sans modifier l'image Docker elle-même. Cela rend l'exécution du conteneur adaptable à différents environnements (développement, production, etc.).

### Question 2: Why do we need a volume to be attached to our postgres container?

Attacher un volume à un conteneur PostgreSQL permet de stocker les données en dehors du conteneur. Ainsi, même si le conteneur est arrêté ou supprimé, les données restent accessibles. C'est essentiel pour éviter la perte des données de la base de données et garantir leur persistance à long terme.

### 1-1 Document your database container essentials: commands and Dockerfile.

Nous utiliserons l'image : `postgres:14.1-alpine`. Voici le Dockerfile :

`Dockerfile`
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```
Nous pouvons y trouver les informations de connexions our accédez à la base de donnée.

Construire l'image Docker :
```
docker build -t database
```
Créer un réseau :
```
docker network create app-network
```
Exécuter le conteneur PostgreSQL :
```
docker run --name mydatabase -v /mon/dossier/datadir:/var/lib/postgresql/data --net=app-network -d database
```

L'utilisation du volume (`-v`) permet que les données persisteront même si le conteneur est supprimé

### 1-2 Why do we need a multistage build? And explain each step of this dockerfile.

Le build multi-étape permet de réduire la taille des images Docker et de séparer l'étape de construction (build) de l'étape d'exécution (run). Cela permet d'avoir une image finale plus légère et plus sécurisée.
Le build multi-étape permet de créer des images Docker plus légères et sécurisées. En séparant les étapes de construction et d'exécution, nous n'incluons que les fichiers nécessaires à l'exécution de l'application dans l'image finale.

#### Étapes du Dockerfile

1. **Étape de construction (Build)** :
    ```Dockerfile
    FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY pom.xml .
    COPY src ./src
    RUN mvn package -DskipTests
    ```
   - **Description** : Cette étape utilise l'image Maven pour compiler l'application. Elle copie le fichier `pom.xml` et le répertoire `src`, puis exécute la commande Maven pour créer le package de l'application (sans exécuter les tests).

2. **Étape d'exécution (Run)** :
    ```Dockerfile
    FROM amazoncorretto:17
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar
    ENTRYPOINT java -jar myapp.jar
    ```
   - **Description** : Cette étape utilise une image légère de Java pour exécuter l'application. Elle copie uniquement le fichier JAR généré à partir de l'étape de construction et définit la commande d'entrée pour exécuter l'application.
    
### Question 3: Why do we need a reverse proxy?

Un reverse proxy est utilisé pour rediriger les requêtes HTTP vers différents services dans un environnement de conteneurs.

### Question 4: Why is docker-compose so important?

`docker-compose` permet de définir et de gérer plusieurs conteneurs Docker avec une seule commande. Il est essentiel pour orchestrer des environnements complexes où plusieurs services (par exemple, une application, une base de données, un reverse proxy) doivent fonctionner ensemble.

### 1-3 Document docker-compose most important commands. 

1. **Démarrer les services définis dans le fichier `docker-compose.yml`** :
    ```bash
    docker-compose up
    ```
2. **Arrêter et supprimer les conteneurs, réseaux et volumes** :
    ```bash
    docker-compose down
    ```
3. **Afficher l'état des conteneurs** :
    ```bash
    docker-compose ps
    ```

### 1-4 Document your docker-compose file.

```yaml
version: '3.7'  # Version de la syntaxe docker-compose utilisée

services:  # définition des services
  backend:  # Service du backend
    build: # Instructions pour construire l'image du service
      context: ./simpleapi  # Répertoire avec le Dockerfile du backend
    ports:  
      - "8080:8080"  # Expose le port 8080 de l'hôte vers le conteneur
    env_file:  # Fichier contenant les variables d'environnement
      - .env  
    networks:  
      - app-network  # Réseau app-network
    container_name: mybackend  # Nom du conteneur pour le backend
    depends_on:  # Spécifie les dépendances entre les services
      - database

  database:  # Service de la base de données
    build:  
      context: ./Database  
    ports:  
      - "5432:5432"  # Expose le port 5432 de l'hôte vers le conteneur
    env_file: 
      - .env  
    networks:  
      - app-network  # Réseau app-network
    container_name: database  
    volumes:
      - db-data:/var/lib/postgresql/data  # Monte le volume db-data pour persister les données

  http:  # Service pour le serveur HTTP
    build: 
      context: ./http 
    ports:
      - "80:80"  # Expose le port 80 de l'hôte vers le conteneur
    networks: 
      - app-network  # Réseau app-network
    container_name: http  
    depends_on: 
      - backend  # Ce service dépend du service backend

networks:  # Définition des réseaux
  app-network:

volumes:  # Définition des volumes
  db-data: 
```
### 1-5 Document your publication commands and published images in dockerhub.

### Question 5: Why do we put our images into an online repo?
