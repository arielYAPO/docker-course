# TP3 — Ansible

[Retour au sommaire](../README.md)

Ce TP déploie automatiquement l'application Docker du TP1 sur le serveur Takima. Le playbook applique cinq rôles dans l'ordre : installation de Docker, création du réseau, base de données, API, puis reverse proxy Apache.

## Q3-1 - Inventaire et commandes de base

L'inventaire [`inventories/setup.yml`](inventories/setup.yml) contient :

- `ansible_user: admin`, l'utilisateur SSH ;
- `ansible_ssh_private_key_file`, le chemin de la clé privée ;
- le groupe `prod` et le serveur `assah-paul-ariel.yapo.takima.school`.

Depuis le dossier `tp3-ansible` :

```bash
# Tester la connexion SSH
ansible all -i inventories/setup.yml -m ping

# Lire les informations du système distant
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"

# Supprimer Apache installé sur l'hôte, s'il existe
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become
```

Relancer la dernière commande ne doit produire aucun changement supplémentaire : les modules Ansible sont idempotents.

## Q3-2 - Playbook

Le fichier [`playbook.yml`](playbook.yml) cible tous les hôtes, collecte leurs informations avec `gather_facts` et utilise `become` pour les opérations administrateur. Il appelle les rôles suivants :

1. `docker` installe Docker et le SDK Python ;
2. `network` crée le réseau Docker `my-network` ;
3. `database` construit et démarre PostgreSQL avec un volume persistant ;
4. `app` construit et démarre l'API sous le nom `backend` ;
5. `proxy` construit Apache HTTPD et expose le port 80.

Commande manuelle :

```bash
ansible-playbook -i inventories/setup.yml playbook.yml
```

Le mot de passe PostgreSQL est lu depuis le fichier local ignoré `.db-password`. Dans GitHub Actions, ce fichier est créé temporairement à partir du secret `DB_PASSWORD`.

## Q3-3 - Configuration des tâches `docker_container`

Les rôles utilisent `community.docker.docker_container` :

- `database` démarre PostgreSQL sur `my-network`, reçoit `POSTGRES_DB`, `POSTGRES_USER` et `POSTGRES_PASSWORD`, puis monte `db-volume` dans `/var/lib/postgresql/data` ;
- `app` démarre l'API sous le nom `backend`, lui transmet les identifiants PostgreSQL et la relie au même réseau ;
- `proxy` démarre HTTPD, le relie au réseau et publie uniquement `80:80`.

Tous les conteneurs utilisent `restart_policy: unless-stopped`. La base et l'API ne publient aucun port sur l'hôte : Apache est le seul point d'entrée.

## Déploiement automatique

Le workflow [`deploy.yml`](../.github/workflows/deploy.yml) installe Ansible, configure la clé SSH et le mot de passe depuis GitHub Secrets, puis exécute le playbook. Il peut aussi être lancé manuellement avec `workflow_dispatch`.
