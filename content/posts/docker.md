---
title: "Installation de docker et de docker-compose sous Debian 11 (Bullseye) / Debian 10 (Buster)"
date: 2022-10-14T00:24:42+02:00
draft: false
cover:
    image: img/docker.png
    alt: "docker" # alt text
tags: ["docker", "devops"]  
---

## Installation de Docker

On installe les paquets dont docker va avoir besoin

```bash
sudo apt update
sudo apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

On ajoute le clé GPG de Docker utilisée pour signer les paquets

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
```

On ajoute le dépôt Docker qui contient les dernières versions stables de Docker

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

On met à jour la liste de nos paquets

```bash
sudo apt update
```

On peut dès lors installer Docker

```bash
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

Pour démarrer et activer le service docker

```bash
sudo systemctl enable --now docker
```

Pour permettre de lancer Docker sans être root

```bash
sudo usermod -aG docker $USER
newgrp docker
```

## Installation de Docker Compose

On met à jour nos paquets et nous allons avoir besoin de curl et de wget

```bash
sudo apt update
sudo apt install -y curl wget
```
On télécharge la dernière version de Docker Compose

```bash
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url  | grep docker-compose-linux-x86_64 | cut -d '"' -f 4 | wget -qi -
```

On rend notre binaire éxecutable

```bash
chmod +x docker-compose-linux-x86_64
```

On déplace le fichier dans le PATH

```bash
sudo mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
```