# Docker-Cats-Dogs

Une application de vote distribuée utilisant Docker et Docker Swarm pour comparer la popularité entre les chats et les chiens.

## Architecture du Projet

L'application est composée de plusieurs microservices :
- **Vote** : Interface web permettant aux utilisateurs de voter (Python/Flask)
- **Redis** : File d'attente pour stocker les votes temporairement
- **Worker** : Traite les votes et les enregistre dans la base de données (C#)
- **DB** : Base de données PostgreSQL pour le stockage permanent
- **Result** : Interface web affichant les résultats en temps réel (Node.js)

## Prérequis

- VMware Desktop
- Vagrant
- Docker installé sur la machine hôte

## Configuration de l'Environnement

Le projet utilise Vagrant pour créer trois machines virtuelles :
- `manager` (192.168.33.11) : Nœud manager Docker Swarm
- `worker1` (192.168.33.12) : Premier nœud worker
- `worker2` (192.168.33.13) : Second nœud worker

## Installation et Démarrage

1. Cloner le repository :
```bash
git clone https://github.com/xSupr3me/Docker-Cats-Dogs.git
cd Docker-Cats-Dogs
```

2. Installer le plugin vmware pour vagrant :
```bash
vagrant plugin install vagrant-vmware-desktop
```

3. Démarrer l'environnement Vagrant :
```bash
vagrant up
```

3. L'environnement se configure automatiquement :
   - Installation de Docker sur chaque machine
   - Initialisation du cluster Swarm
   - Construction des images Docker
   - Déploiement des services

## Accès aux Applications

- Interface de vote : http://localhost:8080
- Interface des résultats : http://localhost:8888

## Structure des Services

### Service de Vote
- Port : 8080
- Technologie : Python/Flask
- Permet aux utilisateurs de voter pour les chats ou les chiens

### Service de Résultats
- Port : 8888
- Technologie : Node.js
- Affiche les résultats en temps réel

### Worker
- Technologie : .NET
- Traite les votes depuis Redis
- Enregistre les résultats dans PostgreSQL

### Base de Données
- PostgreSQL
- Stocke les votes de manière persistante

### Redis
- Cache temporaire
- File d'attente pour les votes

## Gestion des Conteneurs

Commandes utiles sur le nœud manager :

```bash
# Voir l'état des services
docker service ls

# Voir sur quelle noeud est un service
docker service ps <service_name>

# Voir les logs d'un service
docker service logs <service_name>

```

## Architecture de Déploiement

- Utilisation de Docker Swarm pour l'orchestration
- Contraintes de placement pour optimiser la distribution des services
- Réplication des services pour la haute disponibilité

## Réseau

- Réseau overlay `app-net` pour la communication inter-services
- Isolation des services par conteneur
- Communication sécurisée entre les nœuds

## Développement

Pour modifier le projet :
1. Modifier les fichiers source dans les dossiers respectifs
2. Reconstruire les images Docker
3. Mettre à jour les services via Docker Swarm

## Arrêt de l'Environnement

```bash
vagrant halt    # Pour mettre en pause
vagrant destroy -f # Pour supprimer complètement
```

## Accès a la base sans SWARM 

Pour acceder a la branche sans swarm :
```bash
git clone --branch base-sans-swarm --single-branch https://github.com/xSupr3me/Docker-Cats-Dogs.git
cd Docker-Cats-Dogs
```