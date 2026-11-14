# Docker_projet

Ce projet consiste à conteneuriser une application de vote distribuée à l'aide de Docker. L'application originale est lancée manuellement via des scripts bash et n'utilise Docker que pour les instances PostgreSQL et Redis.

## Objectif

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

## Initialisation du cluster Swarm

1. **Initialiser le manager** :

    ```bash
    vagrant ssh manager1
    docker swarm init --advertise-addr 192.168.99.100
    ```

2. **Joindre les workers** :

    ```bash
    vagrant ssh worker1
    docker swarm join --token <TOKEN> 192.168.99.100:2377

    vagrant ssh worker2
    docker swarm join --token <TOKEN> 192.168.99.100:2377
    ```

    > Le token est obtenu sur le manager lors du `docker swarm init ...` ou via `docker swarm join-token -q worker`.

    > Pour sortir du ssh : `exit`

3. **Vérifier le cluster** :

    ```bash
    vagrant ssh manager1
    docker node ls
    ```

    Sortie attendue :

    ```
    ID                  NAME    STATUS    AVAILABILITY    MANGER STATUS    ENGINE VERSION
    xxx                 manager Ready     Active          Leader           xx.x.x
    yyy                 worker1 Ready     Active                           xx.x.x
    zzz                 worker2 Ready     Active                           xx.x.x
    ```

## Build et push des images

1. **Démarrer un registry local** (pour les images personnalisées) :

    ```bash
    vagrant ssh manager1
    cd /vagrant
    docker service create --name registry --publish 5000:5000 registry:2
    ```

2. **Construire les images** :

    ```bash
    docker compose build
    ```

3. **Pousser les images vers le registry local** :

    ```bash
    docker push localhost:5000/vote:1.0
    docker push localhost:5000/worker:1.0
    docker push localhost:5000/result:1.0
    ```

## Déploiement de la stack

1. **Exporter les variables d'environnement** :

    ```bash
    export $(cat .env | xargs)
    ```

2. **Déployer la stack** :

    ```bash
    docker stack deploy --compose-file docker-compose.yml voting-app
    ```

3. **Vérification** :

    ```bash
    docker stack services voting-app
    docker stack ps voting-app
    ```

## Accès à l'application

| Application | URL |
|-------------|-----|
| Vote | `http://192.168.99.100:8080/` |
| Result | `http://192.168.99.100:8081/` |

## Arrêt

```bash
# Supprimer la stack
docker stack rm voting-app

# Supprimer le registry
docker service rm registry

# Quitter le Swarm (sur chaque nœud)
docker swarm leave --force   # workers
docker swarm leave --force   # manager (après les workers)  
```
