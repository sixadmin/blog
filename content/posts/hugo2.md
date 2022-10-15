---
title: "Hugo: Déploiement continu"
date: 2022-10-15T01:00:00+02:00
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

      - name: Update theme
        # (Optional)If you have the theme added as submodule, you can pull it and use the most updated version
        run: git submodule update --init --recursive

      - name: Installing Homebrew and Hugo
        run: |
          /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
          test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
          test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
          echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
          echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >>~/.bashrc
          echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >>~/.profile
          echo "/home/linuxbrew/.linuxbrew/bin" >> $GITHUB_PATH
          source ~/.bashrc
          source ~/.profile
          brew --version
          brew install hugo

      - name: Build
        run: hugo
  
      - name: Install SSH Key
        run: |
          install -m 600 -D /dev/null ~/.ssh/id_rsa
          echo "${{ secrets.PRIVATE_SSH_KEY }}" > ~/.ssh/id_rsa
          echo "${{ secrets.KNOWN_HOSTS }}" > ~/.ssh/known_hosts

      - name: Deploy
        run: rsync --archive --delete --stats -e 'ssh -p 2227' 'public/' richard@${{ secrets.REMOTE_DEST }}:/var/www/html/blog
```

## Explication du workflow