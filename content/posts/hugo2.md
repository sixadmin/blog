---
title: "Hugo: Déploiement continu"
date: 2022-10-14T01:00:00+02:00
draft: false
cover:
    image: img/hugo.png
    alt: "hugo" # alt text
tags: ["hugo"]  
ShowToc: true
---
## Prérequis

La finalité est de déployer automatiquement notre blog via Github Action dès qu'on pousse une mise à jour sur notre repository. 

Afin de déployer notre projet sur notre VPS, nous allons créér un dossier sur le VPS destiné à hébérger notre site web statique Hugo et donner les droits d'écriture à l'utilsateur courant.

```bash
sudo mkdir /var/www/html/blog
```

```bash
sudo chown $USER:www-data /var/www/html/blog
```

Apache est configuré pour que notre nom de domaine affiche le contenu du dossier /var/www/html/blog

## Création du workflow

Commencons par créer le fichier qui va contenir notre scénario.

```bash
mkdir -p .github/workflows/
```

```bash
touch .github/workflows/build.yml
```

```yaml
name: build

on:
  push:
    branches:
    - main

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:

      - name: Checkout
        uses: actions/checkout@v2

      - name: Installing Homebrew
        run: |
          /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
          test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
          test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
          echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
          echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >>~/.bashrc
          echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >>~/.profile
          source ~/.bashrc
          source ~/.profile
          brew --version
      - name: Verify Homebrew's Installation
        run: |
          cat ~/.profile
          echo "======================="
          cat ~/.bashrc
          echo "======================="
          brew --version
```