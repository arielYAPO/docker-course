# TP1 — Docker

[Retour au sommaire](../README.md)

## 1 - Database

### Q1-1 - Pourquoi utiliser `-e` pour les variables d'environnement ?

Il est préférable de passer les variables d'environnement avec `-e` plutôt que de les écrire directement dans le Dockerfile afin d'éviter de stocker des informations sensibles comme les mots de passe dans le code.

Cela permet aussi de modifier la configuration du container sans reconstruire l'image.

### Q1-2 - Pourquoi utiliser un volume avec PostgreSQL ?

Un volume permet de conserver les données de PostgreSQL même si le container est supprimé ou recréé.

Les données sont stockées indépendamment du container, ce qui permet de les rendre persistantes.

### Q1-3 - Éléments essentiels de la base de données

Dockerfile :

```dockerfile
FROM postgres:17.2-alpine

COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
COPY 02-InsertData.sql /docker-entrypoint-initdb.d/
```

Construction de l'image :

```bash
docker build -t my-database .
```

Création du réseau :

```bash
docker network create app-network
```

Création du volume :

```bash
docker volume create postgres-data
```

Lancement de PostgreSQL :

```bash
docker run -d --name database --network app-network \
-e POSTGRES_DB=db \
-e POSTGRES_USER=usr \
-e POSTGRES_PASSWORD=pwd \
-v postgres-data:/var/lib/postgresql/data \
my-database
```

Lancement d'Adminer :

```bash
docker run -d --name adminer --network app-network -p 8090:8080 adminer
```

Commandes utiles :

```bash
docker ps
docker logs database
docker stop database
docker rm -f database
```

Les fichiers `01-CreateScheme.sql` et `02-InsertData.sql` sont exécutés automatiquement par PostgreSQL lors de la première initialisation de la base.

## 2 - Backend API

### Q1-4 - Pourquoi utiliser un build multistage ?

Un build multistage sépare la construction de l'application de son exécution. La première étape contient le JDK et Maven pour compiler le projet. La seconde ne conserve que le JRE et le fichier JAR produit. L'image finale est donc plus petite et ne contient pas les outils de compilation inutiles à l'exécution.

Étapes du Dockerfile :

1. `FROM eclipse-temurin:21-jdk-alpine AS myapp-build` crée l'étape de compilation avec un JDK Java 21.
2. `WORKDIR /opt/myapp` définit le répertoire de travail.
3. `RUN apk add --no-cache maven` installe Maven dans l'étape de compilation.
4. `COPY pom.xml .` puis `RUN mvn dependency:go-offline` téléchargent les dépendances dans une couche réutilisable tant que le `pom.xml` ne change pas.
5. `COPY src ./src` copie le code source, puis `RUN mvn package -DskipTests` produit le JAR.
6. `FROM eclipse-temurin:21-jre-alpine` démarre une image d'exécution propre avec uniquement le JRE.
7. `COPY --from=myapp-build ...` récupère le JAR depuis la première étape.
8. `ENTRYPOINT ["java", "-jar", "myapp.jar"]` lance l'application.

Construction et lancement de l'API simple :

```bash
cd backend/simple-api
docker build -t simple-api .
docker run -d --name simple-api -p 8080:8080 simple-api
```

Test du endpoint :

```bash
curl "http://localhost:8080/?name=World"
```

### API connectée à PostgreSQL

L'API utilise le nom du conteneur `database` comme nom d'hôte, car les deux conteneurs sont reliés au réseau Docker `app-network` :

```yaml
spring:
  datasource:
    url: jdbc:postgresql://database:5432/db
    username: ${POSTGRES_USER:usr}
    password: ${POSTGRES_PASSWORD:pwd}
```

Construction et lancement manuel :

```bash
docker build -t simple-api backend/simple-api
docker run -d --name simple-api --network app-network -p 8080:8080 \
  -e POSTGRES_USER=usr \
  -e POSTGRES_PASSWORD=pwd \
  simple-api
```

Endpoints principaux fournis par l'application :

```text
GET    /departments
GET    /departments/{name}
GET    /departments/{name}/students
GET    /departments/{name}/count
GET    /students
GET    /students/{id}
POST   /students
PUT    /students/{id}
DELETE /students/{id}
```

## 3 - Serveur HTTP

L'image utilise `httpd:2.4-alpine`. Le fichier `index.html` est copié dans le dossier servi par Apache. Les modules `proxy` et `proxy_http` sont activés dans la configuration Apache, puis le virtual host transmet les requêtes au service `backend:8080`.

### Q1-5 - Pourquoi utiliser un reverse proxy ?

Le reverse proxy constitue le point d'entrée de l'application. Il reçoit les requêtes des utilisateurs et les transmet au backend, qui n'a donc pas besoin d'être directement exposé. Il peut également servir les fichiers du frontend et centraliser plus tard la configuration HTTPS ou la répartition de charge.

Configuration utilisée :

```apache
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://backend:8080/
    ProxyPassReverse / http://backend:8080/
</VirtualHost>
```

Construction et lancement manuel :

```bash
docker build -t my-httpd httpd
docker run -d --name httpd --network app-network -p 80:80 my-httpd
```

## 4 - Docker Compose

### Q1-6 - Pourquoi Docker Compose est-il important ?

Docker Compose décrit tous les services de l'application dans un seul fichier. Il permet de construire, démarrer, arrêter et recréer de manière reproductible les trois conteneurs, leur réseau, leurs dépendances et leur volume. Cela évite de relancer manuellement une longue série de commandes Docker.

### Q1-7 - Commandes principales

```bash
# Définir le mot de passe pour la session du terminal
export POSTGRES_PASSWORD=pwd

# Construire les images et démarrer les services en arrière-plan
docker compose up -d --build

# Afficher l'état des services
docker compose ps

# Suivre les journaux
docker compose logs -f

# Reconstruire les images
docker compose build

# Arrêter et supprimer les conteneurs et le réseau
docker compose down

# Arrêter l'application et supprimer également son volume
docker compose down -v
```

Sous PowerShell, la variable est définie ainsi :

```powershell
$env:POSTGRES_PASSWORD = "pwd"
```

### Q1-8 - Explication du fichier Compose

Le fichier `compose.yaml` définit trois services :

- `database` construit l'image PostgreSQL, reçoit sa configuration par variables d'environnement et monte le volume `postgres-data` dans `/var/lib/postgresql/data`. Son healthcheck utilise `pg_isready`.
- `backend` construit l'API Spring Boot et attend que la base soit saine avant de démarrer. Il communique avec elle grâce au nom de service `database`.
- `httpd` construit le reverse proxy Apache, attend le démarrage du backend et publie le seul port accessible depuis la machine : le port 80.

Les trois services utilisent le réseau privé `app-network`. Aucun port du backend ou de PostgreSQL n'est publié sur la machine hôte. La politique `restart: unless-stopped` relance les services après une erreur ou un redémarrage de Docker, sauf s'ils ont été arrêtés volontairement.

Lancement complet :

```bash
export POSTGRES_PASSWORD=pwd
docker compose up -d --build
curl http://localhost/departments/IRC/students
```

## 5 - Publication des images

### Q1-9 - Commandes de publication et images Docker Hub

La publication est automatisée par le workflow [`cd.yml`](../.github/workflows/cd.yml). Après la réussite de la CI sur `main`, il se connecte à Docker Hub, construit les trois images et les publie avec les noms suivants :

```text
<DOCKERHUB_USERNAME>/tp-devops-simple-api:latest
<DOCKERHUB_USERNAME>/tp-devops-database:latest
<DOCKERHUB_USERNAME>/tp-devops-httpd:latest
```

Les commandes manuelles équivalentes sont :

```bash
docker login
docker build -t <DOCKERHUB_USERNAME>/tp-devops-simple-api:latest backend/simple-api
docker build -t <DOCKERHUB_USERNAME>/tp-devops-database:latest database
docker build -t <DOCKERHUB_USERNAME>/tp-devops-httpd:latest httpd
docker push <DOCKERHUB_USERNAME>/tp-devops-simple-api:latest
docker push <DOCKERHUB_USERNAME>/tp-devops-database:latest
docker push <DOCKERHUB_USERNAME>/tp-devops-httpd:latest
```

Le nom réel du compte n'est pas écrit en dur : GitHub Actions utilise le secret `DOCKERHUB_USERNAME`.

### Q1-10 - Pourquoi placer les images dans un registre en ligne ?

Un registre comme Docker Hub permet à une autre machine ou à un serveur de télécharger exactement les mêmes images avec `docker pull`. Le déploiement est reproductible, les images peuvent être partagées avec l'équipe et la machine cible n'a pas besoin de reconstruire l'application depuis le code source.
