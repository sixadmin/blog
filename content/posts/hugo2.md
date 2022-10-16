---
title: "Hugo: Déploiement continu"
date: 2022-10-15T01:00:00+02:00
draft: false
cover:
    image: img/hugo2.png
    alt: "hugo" # alt text
tags: ["hugo", "CI/CD", "DevOps"]  
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

On édite le fichier build.yaml et on y rajoute le contenu suivant

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

```yaml
name: build

on:
  push:
    branches:
    - main
```

Ici, le workflow s'éxecutera quand il y aura un push sur la branche main.

```yaml
jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:

      - name: Checkout
        uses: actions/checkout@v2
```

Ensuite, on part sur une machine ubuntu et on y déploie les sources de notre repository

```yaml
      - name: Update theme
        # (Optional)If you have the theme added as submodule, you can pull it and use the most updated version
        run: git submodule update --init --recursive
```

On installe les sous-modules. Dans ce cas, cela correspond au theme PaperMod

```yaml
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
```

Dans cette étape, on télécharge le script d'installation de homebrew et on rajoute les binaires dans le PATH de notre machine. On conclut par l'installation de hugo

```yaml
      - name: Build
        run: hugo
```

On lance la génération de notre site, il va créer un dossier public qui va contenir les pages générées.

```yaml
      - name: Install SSH Key
        run: |
          install -m 600 -D /dev/null ~/.ssh/id_rsa
          echo "${{ secrets.PRIVATE_SSH_KEY }}" > ~/.ssh/id_rsa
          echo "${{ secrets.KNOWN_HOSTS }}" > ~/.ssh/known_hosts
```

Dans cette partie, on crée un fichier de clé privé id_rsa avec les droits adéquats et on y copie la clé privé qu'on a généré sur notre ordinateur sachant qu'on a copié la clé publique sur le serveur qui va hébérger notre site. On renseigne le fichier know_hosts afin que l'on n'ai pas d'interactivité quand on lancera le rsync.

Les valeurs PRIVATE_SSH_KEY et KNOWN_HOSTS correspondent à des secrets qu'on a générés dans Github dans Settings\Secrets\Actions

![Secret](/img/secrets.png)

```yaml
      - name: Deploy
        run: rsync --archive --delete --stats -e 'ssh -p 2227' 'public/' richard@${{ secrets.REMOTE_DEST }}:/var/www/html/blog
```

Enfin, on synchronise le dossier public qui vient d'être généré avec le root directory apache de notre site internet.