---
title: "Installation de terraform sous Debian"
summary: Retrouvez dans ce tuto les commandes pour installer terraform sous Debian
date: 2022-10-13T01:24:42+02:00
draft: false
cover:
    image: img/terraform.png
    alt: "terraform" # alt text
tags: ["terraform", "devops"]
ShowToc: true
---

##  Mise à jour des sources list debian

On met à jour la liste des paquets disponibles



On met à jour notre système si besoin

```bash
sudo apt upgrade
```

##  Installation des dépendances dont terraform a besoin

```bash
sudo apt-get install -y gnupg software-properties-common curl
```

## Ajout du dépôt officiel de Hashicorp

On commence par ajouter la clé GPG de Hashicorp

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
```

on ajoute le dépôt officiel

```bash
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
```
Et on lance une mise à jour de la liste des paquets disponibles pour prendre en compte ce dépôt.

```bash
sudo apt update
```

## Installation de terraform CLI

```bash
sudo apt-get install terraform
```
On peut vérifier la version installé

```bash
terraform version
```

Ce qui nous donne:

```bash
Terraform v1.3.2
on linux_amd64
```