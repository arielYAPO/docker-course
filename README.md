# Docker — TP1, TP2 et TP3

Ce dépôt regroupe les trois parties du TP DevOps dans un seul projet.

## Navigation

| Partie | Sujet | Contenu |
| --- | --- | --- |
| [TP1](tp1-docker/README.md) | Docker | PostgreSQL, API Spring Boot, Apache HTTPD, reverse proxy et Docker Compose |
| [TP2](tp2-github-actions/README.md) | GitHub Actions | Tests, Testcontainers, SonarCloud et publication Docker Hub |
| [TP3](tp3-ansible/README.md) | Ansible | Installation de Docker et déploiement de l'application sur le serveur |

## Architecture

```text
Client :80
   |
Apache HTTPD (reverse proxy)
   |
API Spring Boot :8080
   |
PostgreSQL :5432 + volume persistant
```

Seul Apache publie un port vers l'extérieur. L'API et PostgreSQL communiquent sur un réseau Docker privé.

## Lancement local

Depuis `tp1-docker` :

```bash
export POSTGRES_PASSWORD=pwd
docker compose up -d --build
curl http://localhost/departments/IRC/students
```

Sous PowerShell :

```powershell
$env:POSTGRES_PASSWORD = "pwd"
docker compose up -d --build
Invoke-RestMethod http://localhost/departments/IRC/students
```

Pour arrêter l'application :

```bash
docker compose down
```

Les réponses détaillées aux questions et les commandes de chaque partie sont dans le README correspondant.
