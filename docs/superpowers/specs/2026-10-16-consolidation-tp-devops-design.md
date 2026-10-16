# Regroupement des TP DevOps

## Objectif

Regrouper les trois TP dans un seul dépôt GitHub, sans perdre le travail déjà réalisé, avec une navigation simple pour la correction.

Le dépôt final sera renommé `docker-tp1-tp2-tp3`.

## Structure retenue

```text
docker-tp1-tp2-tp3/
├── README.md
├── tp1-docker/
│   ├── README.md
│   ├── backend/
│   ├── database/
│   ├── httpd/
│   └── compose.yaml
├── tp2-github-actions/
│   └── README.md
├── tp3-ansible/
│   ├── README.md
│   ├── inventories/
│   ├── roles/
│   └── playbook.yml
└── .github/workflows/
```

## Contenu de chaque partie

- Le `README.md` principal sert uniquement de sommaire et renvoie vers chaque TP.
- `tp1-docker` contient l'application Spring Boot, PostgreSQL, Apache HTTPD et Docker Compose déjà validés localement. Son README répond clairement aux questions du TP1.
- `tp2-github-actions` contient son README explicatif. Les workflows restent dans `.github/workflows`, car GitHub ne les exécute qu'à cet emplacement. Ils utilisent les fichiers de `tp1-docker` pour les tests, SonarCloud et la publication des images Docker.
- `tp3-ansible` contient l'inventaire, le playbook et les rôles Ansible existants. Son README explique le déploiement et répond aux questions du TP3.

## Sources conservées

- Le dépôt actuel `docker-course` sert de base afin de conserver son historique Git et le travail Ansible.
- Le TP1 local validé remplace l'ancienne version Docker présente dans ce dépôt.
- Les workflows du dépôt `tp-devops-correction-docker` sont repris et adaptés aux nouveaux chemins.
- Le dépôt séparé `tp-devops-correction-docker` n'est ni supprimé ni modifié.

## Adaptations nécessaires

- Mettre à jour les chemins utilisés par GitHub Actions après le déplacement de l'application.
- Mettre à jour les chemins et noms de conteneurs utilisés par Ansible afin qu'ils correspondent à la pile Docker du TP1.
- Garder les secrets uniquement dans GitHub Secrets ou dans les variables d'environnement, jamais dans le dépôt.
- Ajouter des liens relatifs entre le sommaire principal et les trois README.

## Vérifications finales

Avant l'envoi sur GitHub :

1. vérifier tous les liens des README ;
2. lancer les tests Maven ;
3. valider la configuration Docker Compose ;
4. vérifier la syntaxe Ansible ;
5. contrôler les chemins et déclencheurs des workflows GitHub Actions ;
6. vérifier que le dépôt ne contient aucun secret ;
7. pousser les changements, puis renommer `docker-course` en `docker-tp1-tp2-tp3`.

Le résultat attendu est un dépôt unique, lisible et directement parcourable par l'enseignante, avec un README distinct pour chaque TP.
