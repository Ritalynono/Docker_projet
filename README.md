# Docker_projet

Ce projet consiste à conteneuriser une application de vote distribuée à l'aide de Docker. L'application originale est lancée manuellement via des scripts bash et n'utilise Docker que pour les instances PostgreSQL et Redis.

## L'objectif

Conteneuriser chaque module (vote, worker, result) avec des Dockerfiles respectant les bonnes pratiques.
Orchestrer l'ensemble avec Docker Compose (réseaux, volumes, dépendances, healthchecks).
Déployer l'application sur un cluster Docker Swarm (1 manager, 2 workers).
L'application permet à une audience de voter entre deux propositions. Un vote ne peut être émis qu'une seule fois par navigateur, mais il est modifiable. Les résultats sont mis à jour en temps réel.

## Installation

- Prérequis :

    - VirtualBox 7.2.x (Extension Pack requis)

    - Vagrant ≥ 2.4

- Mise en place du projet :

    - À la racine du projet :

        ```bash
        vagrant up
        ```

