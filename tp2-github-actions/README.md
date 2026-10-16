# TP2 — GitHub Actions

[Retour au sommaire](../README.md)

Ce TP automatise les tests, l'analyse de qualité et la publication des images du [TP1](../tp1-docker/README.md). Les workflows doivent rester dans [`.github/workflows`](../.github/workflows/) pour être détectés par GitHub.

## Fonctionnement

- [`main.yml`](../.github/workflows/main.yml) est la CI. Elle s'exécute sur les push vers `main` et `develop`, ainsi que sur les pull requests. Elle installe Java 21, lance `mvn clean verify`, puis transmet l'analyse à SonarCloud et attend le Quality Gate.
- [`cd.yml`](../.github/workflows/cd.yml) est la CD. Elle démarre uniquement après une CI réussie issue d'un push sur `main`, se connecte à Docker Hub, puis construit et publie les images API, PostgreSQL et HTTPD.
- [`deploy.yml`](../.github/workflows/deploy.yml) exécute le déploiement Ansible du TP3 lors d'un push sur `main` ou manuellement.

## Q2-1 - Que sont les Testcontainers ?

Testcontainers est une bibliothèque de test qui démarre des conteneurs Docker temporaires. Ici, les tests d'intégration lancent une vraie base PostgreSQL isolée grâce à l'URL `jdbc:tc:postgresql`. Cela rend les tests plus proches de la production et reproductibles, puis le conteneur est supprimé à la fin.

Les dépendances et la configuration se trouvent dans :

- [`pom.xml`](../tp1-docker/backend/simple-api/pom.xml) ;
- [`application.yml` de test](../tp1-docker/backend/simple-api/src/test/resources/application.yml).

## Q2-2 - Pourquoi utiliser des variables sécurisées ?

Les identifiants et jetons ne doivent pas apparaître dans le dépôt ou dans le YAML des workflows. Les GitHub Secrets les rendent disponibles seulement pendant l'exécution autorisée du workflow et réduisent le risque de fuite.

Secrets utilisés :

- `SONAR_TOKEN` pour SonarCloud ;
- `DOCKERHUB_USERNAME` et `DOCKERHUB_TOKEN` pour Docker Hub ;
- `TAKIMA_SSH_PRIVATE_KEY` et `DB_PASSWORD` pour le déploiement Ansible.

## Q2-3 - Pourquoi imposer la dépendance avec le job de test ?

Le but de `needs: build-and-test-backend` est d'empêcher la publication si les tests échouent. Dans ce dépôt, la CI et la CD sont séparées : `workflow_run` joue le même rôle et la condition vérifie que la conclusion de la CI vaut `success`. Sans cette dépendance, une image défectueuse pourrait être publiée.

## Q2-4 - Pourquoi pousser les images Docker ?

Une image locale n'est disponible que sur la machine qui l'a construite. La pousser sur Docker Hub permet à l'équipe et aux serveurs de déploiement de récupérer la même version sans la reconstruire, avec un résultat identique d'un environnement à l'autre.

## Exécution locale équivalente

Depuis la racine du dépôt :

```bash
mvn clean verify --file tp1-docker/backend/simple-api/pom.xml
```

La publication n'est faite que par la CD afin que les identifiants Docker Hub restent dans GitHub Secrets.
