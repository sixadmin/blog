---
title: "Traefik"
summary: Traefik
date: 2022-11-09T01:24:42+02:00
draft: false
cover:
    image: img/traefik.png
    alt: "traefik" # alt text
tags: ["docker", "devops", "traefik"]
ShowToc: true
---

<!--
https://www.digitalocean.com/community/tutorials/how-to-use-traefik-v2-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04
-->

# Pourquoi Traefik ?

Un reverse proxy est un type de serveur proxy qui dirige les demandes des clients vers le serveur approprié. Imaginons que vous avez un nas accessible via une interface web ainsi qu'une webcam (qui a aussi une interface web) sur votre réseau et vous souhaitez y accéder depuis internet.
Une solution serait de configurer votre routeur en [PAT](https://waytolearnx.com/2018/07/difference-entre-nat-et-pat.html). Si notre IP publique est XX.XX.XX.XX, on pourrait faire en sorte que l'URL XX.XX.XX.XX:8080 redirige vers l'IP local du NAS et que l'URL XX.XX.XX.XX:8081 redirige vers l'IP local de la webcam.
Cela répond à notre besoin mais ce n'est pas la méthode la plus élégante.
C'est pour cela que nous allons utiliser traefik. Traefik est un reverse proxy écrit en Go. l'interêt de Traefik par rapport à ses concurrents Nginx, HAproxy ou encore Apache, c'est qu'il permet surtout aussi d'exposer des containers docker. Il propose des options très intéressants comme la gestion des certificats letsencrypt ou la possibilité de mettre en place une whitelist pour filtrer les accès à nos services.

# Mise en place

## Le docker-compose

Pour utiliser traefik, nous allons créer un dossier traefik et y créer un fichier docker-compose.yaml avec le contenu suivant:

```docker
version: '3'

networks:
  dockernetwork:
    external: true
services:
  reverse-proxy:
    restart: always
    image: traefik:v2.8
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - /home/richard/traefik/traefik.toml:/etc/traefik/traefik.toml
      - /home/richard/traefik/services.toml:/etc/traefik/services.toml
      - /home/richard/traefik/acme.json:/acme.json
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - dockernetwork
    labels:
      - "traefik.http.routers.api.rule=Host(`traefik.mydomain.com`)"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls=true"
      - "traefik.http.routers.api.tls.certresolver=richard"
      - "traefik.http.routers.api.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=richard:$$apr1$$qTjRvKhq$$pfulncMRf2Hkv5SUgUtvh1"
      - "traefik.http.middlewares.myAccess.ipWhiteList.sourceRange=192.168.0.0/24, XX.XX.XX.XX"
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=websecure"

      - "traefik.http.routers.nas.entrypoints=websecure"
      - "traefik.http.routers.nas.rule=Host(`nas.mydomain.com`)"
      - "traefik.http.routers.nas.service=nas@file"
      - "traefik.http.routers.nas.tls=true"
      - "traefik.http.routers.nas.tls.certresolver=richard"
      - "traefik.http.routers.nas.middlewares=myAccess@docker"
```

Plusieurs informations sont déclarées dans ce fichiers.

En tout premier, nous avons un network. Ici le network est dockernetwork. Si vous avez des containers qui fonctionnent sur d'autres networks et que vous souhaitez les exposer via traefik, il faudra aussi les déclarer dans ce fichier.

Ensuite, nous retrouvons les options classiques qu'on retrouve dans un fichier docker-compose comme l'image utilisée et les ports via lesquels on accéde à ce container (en l'occurence, ici ce sont les ports 80 et 443).
La partie volume fait du mapping sur différents fichiers:

```docker
     - /var/run/docker.sock:/var/run/docker.sock
```

Ici, on donne l'accès au socket interne de Docker. Traefik pourra donc accéder à l'activité de docker. Il pourra détécter le lancement d'un nouveau container par exemple...

```docker
      - /home/richard/traefik/acme.json:/acme.json
```

Comme dit plus haut, Traefik sait gérer les certificats https. Le fichier acme.json est destiné à stocker les certificats générées.
Quand on crée ce fichier dans le dossier traefik, il faut veiller à lui donner les bons droits.

```bash
chmod 600 acme.json
```

```docker
      - /home/richard/traefik/traefik.toml:/etc/traefik/traefik.toml
```

### traefik.toml

Voici le contenu de notre fichier traefik.toml

```toml
[entryPoints]
  [entryPoints.web]
  address = ":80"
  
  [entryPoints.websecure]
  address = ":443"

[api]

[providers.docker]
  endpoint = "unix:///var/run/docker.sock"

[providers.file]
  filename = "/etc/traefik/services.toml"

[certificatesResolvers.richard.acme]
  email = "richardxxxxx@gmail.com"
  storage = "acme.json"
  [certificatesResolvers.richard.acme.httpChallenge]
    entryPoint = "web"
```

On déclare donc deux entryPoints. L'entryPoints web qui "écoute" sur le port 80 et l'entryPoints websecure qui "écoute" sur le port 443. L'adresse définit le port et optionnelement l'hostname sur lesquels on écoute les connections entrantes. On peut spécifier un quel protocole utilisé (TCP ou UDP). Si on ne spécifie pas le protocole, il utilise TCP par défaut.

On a une rubrique api qui met à disposition un dashboard traefik, on retrouve le détail de sa configuration dans la partie labels du docker-compose.yaml

```toml
[api]
```

Ensuite, on déclare le socket Unix que le démon Docker écoute par défaut:

```toml
[providers.docker]
  endpoint = "unix:///var/run/docker.sock"
```

Nous avons une section sépcifique pour les certificats https:

```toml
[certificatesResolvers.richard.acme]
  email = "richardxxxxx@gmail.com"
  storage = "acme.json"
  [certificatesResolvers.richard.acme.httpChallenge]
    entryPoint = "web"
```

On indique un email de contact, on lui dit aussi le nom du fichier qui va stocker les certificats acme.json

Ensuite, nous avons le section provider.file ou on lui spécifie le chemin vers un fichier services.toml qui va contenir les services qu'on va exposer via traefik

```toml
[providers.file]
  filename = "/etc/traefik/services.toml"
```

On mappe donc ce fichier pour que traefik puisse y accéder

```docker
      - /home/richard/traefik/services.toml:/etc/traefik/services.toml
```

### services.toml

Et voici le contenu de notre fichier services.toml:

```toml
[http]
  [http.services]
    [http.services.nas]
      [http.services.nas.loadBalancer]
        [[http.services.nas.loadBalancer.servers]]
          url = "http://192.168.0.100:5000/"
```

Ici, l'URL http://192.168.0.100:5000/ correspond à un NAS synology. Donc, on a créé un service qui s'appelle nas.

Si on se refère à notre fichier docker-compose.yaml, on a une section spécifique pour accéder au NAS:

```docker
      - "traefik.http.routers.nas.entrypoints=websecure"
      - "traefik.http.routers.nas.rule=Host(`nas.mydomain.com`)"
      - "traefik.http.routers.nas.service=nas@file"
      - "traefik.http.routers.nas.tls=true"
      - "traefik.http.routers.nas.tls.certresolver=richard"
      - "traefik.http.routers.nas.middlewares=myAccess@docker"
```

- On lui spécifie qu'on accède à ce service via l'entrypoints websecure (donc sur le port 443)
- On lui donne quel nom de domaine on doit taper pour rediriger vers l'interface du nas
- On lui donne le nom du service as@file.Le provider file a été déclaré dans le fichier traefik.toml et service nas a été déclaré dans le fichier services.toml. le nom du service est sous la forme [nom du service]@[provider].
- On lui dit qu'on veut du https
- On lui indique le nom de notre resolver qu'on a défini dans le fichier traefik.toml (ici: richard)
- Enfin, on lui dit que l'on utilise le middleware myAccess défini dans le fichier docker-compose.yaml ou on lui spécifie des adresses IP autorisées. Bien sûr, si on veut que son service soit accessible de partout, on n'utilise pas cette option.

### networks

Poursuivons l'analyse de notre fichier docker-compose.yaml

Nous avons donc une partie networks

```docker
    networks:
      - dockernetwork
```

On lui indique les réseaux utilisés par docker et comme vu plus haut, si on est amené à exposer un container qui tourne sur un network différent, il faut l'ajouter à cette liste.

### les labels

On a ensuite la partie labels

```docker
    labels:
      - "traefik.http.routers.api.rule=Host(`traefik.mydomain.com`)"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls=true"
      - "traefik.http.routers.api.tls.certresolver=richard"
      - "traefik.http.routers.api.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=richard:$$apr1$$qTjRvKhq$$pfulncMRf2Hkv5SUgUtvh1"
      - "traefik.http.middlewares.myAccess.ipWhiteList.sourceRange=192.168.0.0/24, XX.XX.XX.XX"
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=websecure"

      - "traefik.http.routers.nas.entrypoints=websecure"
      - "traefik.http.routers.nas.rule=Host(`nas.mydomain.com`)"
      - "traefik.http.routers.nas.service=nas@file"
      - "traefik.http.routers.nas.tls=true"
      - "traefik.http.routers.nas.tls.certresolver=richard"
      - "traefik.http.routers.nas.middlewares=myAccess@docker"
```

On a déjà vu la dernière partie pour exposer un service défini dans un fichier. Regardons la première partie.

```docker
    labels:
      - "traefik.http.routers.api.rule=Host(`traefik.mydomain.com`)"
      - "traefik.http.routers.api.service=api@internal"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls=true"
      - "traefik.http.routers.api.tls.certresolver=richard"
      - "traefik.http.routers.api.middlewares=auth"
      - "traefik.http.middlewares.auth.basicauth.users=richard:$$apr1$$qTjRvKhq$$xxxxxxxxxxxh1"
      - "traefik.http.middlewares.myAccess.ipWhiteList.sourceRange=192.168.0.0/24, XX.XX.XX.XX"
      - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
      - "traefik.http.routers.http-catchall.entrypoints=web"
      - "traefik.http.routers.http-catchall.middlewares=redirect-to-https"
      - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=websecure"
```

Les premières lignes font références à l'api traefik qui permet d'accéder au dashbord de traefik

- On indique le nom de domaine.
- On lui spécifie qu'on doit y accéder via https (websecure)
- On dit qu'on veut de l'https
- On précise qu'on utilise le resolver richard
- Et on lui dit aussi d'utiliser le middleware auth

Le middleware auth est renseigné à la ligne juste en dessous. Cela correspond à une authentification basique. On a généré un user et un mot de passe avec la commande suivante:

```bash
htpasswd -nB richard
```

Si le binaire est absent, on l'installe via la commande suivante:

```bash
sudo apt install apache2-utils
```

**TIPS**
Quand vous générez le mot de passe via htpasswd, lorsque vous le renseignez dans le fichier docker-compose.yaml, tous les symboles $ doivent être échappés avec un autre $.

Dans la ligne suivante, on crée un middleware myAccess dans lequel on défnit une liste d'IP autorisées pour accéder à nos services.

Les 4 dernières lignes permettent de faire la redirection https.



