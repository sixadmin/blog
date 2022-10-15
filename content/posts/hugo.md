---
title: "Hugo: Installation et configuration"
date: 2022-10-14T01:00:00+02:00
draft: false
cover:
    image: img/hugo.png
    alt: "hugo" # alt text
tags: ["hugo"]  
ShowToc: true
---
## Installation de brew

Un pré requis à l'installation de homebrew est la présence d'un compilateur.

```bash
sudo apt install build-essential
```

Nous téléchargeons ensuite l'installateur

```bash
curl -fsSL -o install.sh https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh
```

Et nous l'executons

```bash
/bin/bash install.sh
```

Enfin, on indique à notre système le chemin vers le binaire

```bash
echo '# Set PATH, MANPATH, etc., for Homebrew.' >> /home/richard/.profile
```

```bash
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/richard/.profile
```

```bash
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

## Installation de Hugo

```bash
brew install hugo
```

```bash
hugo new site monblog -f yml
Congratulations! Your new Hugo site is created in /home/richard/Documents/dev/hugo/monblog.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

## Installation du thème Papermod

On se refère à la documentation:

https://github.com/adityatelange/hugo-PaperMod/wiki/Installation

On choisit l'option avec les sous-modules

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```
Et si on reclone notre projet, il faut lancer la commande suivante

```bash
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```

Nous allons ensuite éditer le fichier config.yml à la racine de notre site pour lui indiquer le thème utilsé (entre autres)

```yaml
baseURL: ""
languageCode: fr-fr
title: Blog 6adm.in
theme: PaperMod
```

On peut dès lors lancer notre site avec la commande suivante:

```bash
hugo server
```

On accède à notre site via l'url suivante: http://localhost:1313/

## Ajouter du contenu à notre site

Le site fonctionne mais il n'a aucun contenu à afficher.

Le dossier content est destiné à contenir nos pages. Nous allons donc lancer la commande suivante

```bash
hugo new posts/docker.md
```

Cette commande va créer un dossier posts qui contient le fichier markdown docker.md

Si on l'édite, on voit le contenu suivant:

```markdown
---
title: "Docker"
date: 2022-10-09T21:24:42+02:00
draft: true
---
```

Il est indiqué le titre de notre document, la date de création et il est actuellement en mode brouillon.

Si on souhaite que la page apparaisse, on modifie la ligne draft: true en draft: false

## Ajouter une image à un post

Pour ajouter une image, nous allons dans un premier temps créer un dossier img dans le dossier static et nous y déposerons nos images d'illustration.

Il suffira ensuite de mettre un lien vers cette image dans l'entête de notre page au format markdwon, ex:

```yaml
---
title: "Installation de docker et de docker-compose sous Debian 11 (Bullseye) / Debian 10 (Buster)"
date: 2022-10-14T00:24:42+02:00
draft: false
cover:
    image: img/docker.png
    alt: "docker"
---
```

## Ajout d'un menu

Chaque thème Hugo a ses spécificités, le thème Papermod permet d'ajouter un menu.
Il suffir d'éditer le fichier config.yaml et d'ajouter la section suivante:

```yaml
menu:
  main:
    - identifier: tags
      name: tags
      url: /tags/
      weight: 10
    - identifier: archives
      name: archives
      url: /archives/
      weight: 20
    - identifier: 6adm
      name: 6adm.in
      url: https://6adm.in
      weight: 30
```

Nous avons dorénavant un menu avec une entrée tags, une entrée archives et une autre entrée qui nous renvoie vers la page https://6adm.in

## Création de la page archives

Si on clique sur le lien archives du menu créé ci-dessus, la page ne renvoie vers rien du tout, nous devons la créer.

Pour cela, nous allons créer un fichier archives.md dans le dossier content et nous allons rajouter le contenu suivant:

```yaml
---
title: "Archives"
layout: "archives"
url: "/archives"
summary: "archives"
---
```

Dorénavant, si on clique sur le lien, on se retrouve avec une page qui liste nos posts chronologiquement.

En fait, on utilise le template (layout) qui se trouve dans le dossier themes\PaperMod\layouts\_default.
On a une page archives.html qui contient la structure de la page générée.

Nous avons la possibilité de modifier ce template pour l"adapter à nos besoins mais si nous mettons à jour le thème, nous perdrons nos changements.

La solution est de copier cette page et de la mettre dans le dossier layouts à la racine de notre site en respectant exactement le même chemin, c'est à dire dans ce cas concret que nous allons créer un dossier _default et y copier le fichier archives.html que nous pourrons modifier à notre guise.

## Rajouter des tags à nos pages.

Pour rajouter des tags, c'est très simple, il suffit de rajouter une option tags dans l'entête de notre pages avec un tableau en paramètre qui contient nos tags, ex:

```yaml
---
title: "Installation de docker et de docker-compose sous Debian 11 (Bullseye) / Debian 10 (Buster)"
date: 2022-10-14T00:24:42+02:00
draft: false
cover:
    image: img/docker.png
    alt: "docker" # alt text
tags: ["docker", "devops"]  
---
```
Quand on clique sur la page en question, on retrouve nos tags en bas de page, ex:

![Resume](/img/tags1.png)

 Et si nous nous rendons dans la page tags, nous avons nos tags qui sont listés avec le nombre de fois. Si on clique sur un des tag, il nous liste tout les posts qui ont ce tag.

 ![Resume](/img/tags2.png)

On peut procéder de la même manière pour les catégories, il suffit de rajouter l'option categories en plus ou au lieu de l'option tags. On met aussi un tableau en paramètre qui contient nos catégories.

## Options supplémentaires

PaperMod offre la possibilité de paramétrer notre site à notre goût et offre de nombreuses option, il est indispensable de consulter la documentation pour voir quelles sont les possibilités: https://themes.gohugo.io/themes/hugo-papermod/


Par exemple, nous avons la possibilté de créer un résumé juste au dessus de la liste de nos posts

```yaml
params:
    homeInfoParams:
        Title: Bienvenue
        Content: Ce site a pour vocation de me servir de pense-bête. J’espère qu’à vous aussi, il vous sera utile …
    socialIcons:
        - name: Github
          url: "https://github.com/sixadmin"
        - name: gitlab
          url: "https://gitlab.com/ModernMenu"
        - name: cv
          url: "https://6adm.in/wp-content/uploads/2022/04/Richard_CRUZ_CV.pdf"
        - name: linkedin
          url: "https://www.linkedin.com/in/richard-cruz-7a978687/"
```

Ce qui donne:

![Resume](/img/resume.png)

Il existe d'autres options intéressantes:

On peut proposer une option de partage du post via les réseaux sociaux en rajoutant l'option suivante dans le fichier config.yaml

```yaml
params:
    ShowShareButtons: true
```
On peut indiquer le temps de lecture:

```yaml
Params:
    ShowReadingTime: true
```

On peut rajouter une option qui permet de copier coller le code

```yaml
params:
    ShowCodeCopyButtons: true
```

On peut aussi rajouter une table des matières à une page en rajoutant cette option dans l'entête.

```yaml
ShowToc: true
```

## Génération du site

Pour générer notre site statique, on utilise la commande suivante

```bash
hugo
```

Il crée un dossier public qui contient notre site.

Ce tutoriel est la traduction de l'excellent tuto de la chaine [Envato Tuts+](https://www.youtube.com/c/tutsplus)

{{< youtube hjD9jTi_DQ4 >}}


Le formateur conclut sa vidéo par la publication de son site sur le service Netlify, nous procéderons différement en déployant notre site sur un VPS en SSH via Github Action.