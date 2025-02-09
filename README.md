# Docker Cats vs Dogs - Application de Vote

Cette application est un système de vote distribué utilisant plusieurs services Docker pour permettre aux utilisateurs de voter pour les chats ou les chiens.

## Architecture

L'application est composée de plusieurs services :

- **Vote** (Python) : Interface web permettant aux utilisateurs de voter
- **Redis** (Queue) : File d'attente pour stocker les votes temporairement
- **Worker** (C#) : Traite les votes et les sauvegarde dans la base de données
- **DB** (PostgreSQL) : Stocke les résultats des votes
- **Result** (Node.js) : Affiche les résultats en temps réel

## Prérequis

- Docker
- Docker Compose

## Installation et Démarrage

1. Clonez le dépôt :
```bash
git clone --branch base-sans-swarm --single-branch https://github.com/xSupr3me/Docker-Cats-Dogs.git
cd Docker-Cats-Dogs
```

2. Lancez l'application avec Docker Compose :
```bash
docker-compose up -d
```

## Accès aux Applications

- Interface de vote : http://localhost:8080
- Interface des résultats : http://localhost:8888

