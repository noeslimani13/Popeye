# Popeye – Application de vote multi-conteneurs avec Docker

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

Projet DevOps réalisé en groupe à **Epitech** : conteneuriser et orchestrer avec **Docker Compose** une application de vote composée de **5 services** écrits dans 3 langages différents.

## Architecture

```
 navigateur ──> poll (Python/Flask, :5000) ──> redis (file d'attente)
                                                  │
                                                  ▼
                                          worker (Java) ──> db (PostgreSQL)
                                                                │
 navigateur <── result (Node.js + Socket.io, :5001) <───────────┘
```

| Service | Rôle | Image / build |
| :--- | :--- | :--- |
| `poll` | Page de vote (Flask) qui pousse les votes dans Redis | `python:3.12-slim` |
| `redis` | File d'attente des votes | `redis:7.2` |
| `worker` | Lit les votes dans Redis et les enregistre dans PostgreSQL | build **multi-stage** Maven → `eclipse-temurin:21-jre-alpine` |
| `db` | Base PostgreSQL, volume persistant, schéma chargé au démarrage | `postgres:15` |
| `result` | Affiche les résultats en temps réel (Socket.io) | `node:20-alpine` |

## Points DevOps

- **Un Dockerfile par service**, images légères (`slim`, `alpine`), build **multi-stage** pour le worker Java (l'image finale ne contient que le JRE et le JAR).
- **Réseaux isolés** : `poll-tier`, `result-tier` et `back-tier` ; la base PostgreSQL n’est joignable que par le réseau interne.
- **Volume nommé** `db-data` pour persister les données de PostgreSQL.
- **Configuration par variables d'environnement** (fichier `.env`, non versionné), politique `restart: unless-stopped` et ordre de démarrage avec `depends_on`.

## Lancer le projet

```bash
git clone https://github.com/noeslimani13/Popeye.git
cd Popeye
cp .env.example .env      # puis adapter le mot de passe
docker compose up --build
```

- Vote : http://localhost:5000
- Résultats : http://localhost:5001

## Équipe

Projet réalisé en groupe à Epitech (dépôt d'origine : [LaFicelleCmoi/Docker](https://github.com/LaFicelleCmoi/Docker)).
