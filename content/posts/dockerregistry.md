---
title: "Installation d'un docker registry privé avec authentification"
summary: Voici la procédure pour installer un registry docker privé avec une authentification basique et sécurisé via des certificats https letsencrypt géré par traefik.
date: 2022-11-08T00:24:42+02:00
draft: false
cover:
    image: img/dockerregistry.png
    alt: "docker" # alt text
tags: ["docker", "devops", "registry"]
ShowToc: true
---

## Installation

### Le docker-compose

Pour installer notre registry, nous allons créer un dossier registry dans lequel, nous allons créer un fichier docker-compose.yml avec le contenu suivant:

```docker
version: "3"
services:
  registry:
      image: registry:2
      ports:
        - 5000:5000
      environment:
        - "REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data"
        - "REGISTRY_AUTH=htpasswd"
        - "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"
        - "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd"
      restart: unless-stopped
      networks:
        - dockernetwork
      volumes:
        - ./data:/data
        - /home/richard/registry/auth:/auth
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.registry.entrypoints=web,websecure"
        - "traefik.http.routers.registry.rule=Host(`registry.mydomain.com`)"
        - "traefik.http.services.registry.loadbalancer.server.port=5000"
        - "traefik.http.routers.registry.tls=true"
        - "traefik.http.routers.registry.tls.certresolver=richard"

networks:
  dockernetwork:
    external: true
```

Pour que traefk demande à letsencrypt de générer un certificat https, on déclare des labels dans notre docker-compose ou on lui spécifie les entrypoints, le nom du domaine de notre registry, le port qui expose le service, l'option tls à true pour le certificat et le nom de notre resolver (Un article sur traefik est en cours de préparation).

### L'authentification

Pour l'authentification, nous allons créer un dossier auth dans notre répertoire registry.

Nous lancons ensuite la commande suivante pour générer une utilisateur et un mot de passe associé:

```bash
docker run --entrypoint htpasswd  httpd:2 -Bbn richard monsupermotdepasse > auth/htpasswd
```

## Lancement de notre registry

On lance notre container avec la commande suivante:

```bash
docker-compose up -d
```
Si on n'a pas de message d'erreur, on va pouvoir essayer de se connecter depuis un serveur distant avec la commande: docker login url_du_repo

```bash
richard@docker:~$ docker login https://registry.mydomain.com
Username: richard
Password:
WARNING! Your password will be stored unencrypted in /home/richard/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```
On se déconnecte avec la commande: docker logout

```bash
richard@docker:~$ docker logout
Removing login credentials for https://index.docker.io/v1/
```

Nous avons dorénavant un registry privé sur lequel nous pouvons pousser et récuperer les images docker que nous aurons créer.

