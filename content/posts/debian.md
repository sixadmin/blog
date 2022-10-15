---
title: "Debian"
date: 2022-10-07T21:24:42+02:00
draft: false
cover:
    image: img/debian.png
    alt: "debian" # alt text
   #  caption: "<text>" # display caption under cover
   #  relative: false # when using page bundles set this to true
   #  hidden: true # only hide on current single page
---

# <a name="tutorial-create-and-share-a-docker-app-with-visual-studio-code"></a>Didacticiel : créer et partager une application Docker avec Visual Studio Code

Ce didacticiel est le début d’une série en trois parties présentant l’ancrage à l’aide de Visual Studio Code (VS Code).
Vous apprendrez à créer et exécuter des conteneurs, à conserver des données et à déployer votre application en conteneur dans Azure.

Dans ce premier didacticiel, vous apprendrez à créer et à déployer des applications Dockr.
Vous pouvez ensuite mettre à jour et partager votre application en conteneur.

Les conteneurs sont des environnements virtualisés compacts, tels que des machines virtuelles, qui fournissent une plate-forme pour la création et l’exécution d’applications.
Les conteneurs n’ont pas besoin de la taille et de la surcharge d’un système d’exploitation complet.
[Dockr](https://www.docker.com) est un fournisseur de conteneurs standard et un système de gestion de conteneur tiers.

Le Bureau de l’arrimeur s’exécute sur votre ordinateur et gère vos conteneurs locaux.
des outils de développement comme Visual Studio et VS Code offrent des extensions qui vous permettent de travailler avec un service bureau local de station d’accueil.
Vous pouvez créer des applications en conteneur, déployer des applications dans des conteneurs et déboguer des applications qui s’exécutent sur vos conteneurs.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> - Créez un conteneur.
> - Générez une image conteneur.
> - Démarrez un conteneur d’application.
> - Mettez à jour le code et remplacez le conteneur.
> - Partagez votre image.
> - Exécutez l’image sur une nouvelle instance.

## <a name="prerequisites"></a>Prérequis

- [Visual Studio Code](https://code.visualstudio.com/download).
- [Extension de VS Code d’ancrage](https://code.visualstudio.com/docs/containers/overview).
- [Bureau](https://docs.docker.com/desktop/)de l’ancrage.
- Un compte d' [ancrage Hub](https://hub.docker.com/signup) . Vous pouvez créer un compte gratuitement.

## <a name="create-a-container"></a>Créez un conteneur.

Un conteneur est un processus sur votre ordinateur.
Il est isolé de tous les autres processus sur l’ordinateur hôte.
Cette isolation utilise des espaces de noms de noyau et des groupes de contrôle.

Un conteneur utilise un système de fichiers isolé.
Ce système de fichiers personnalisé est fourni par une *image de conteneur*.
L’image contient tout ce qui est nécessaire pour exécuter une application, comme toutes les dépendances, la configuration, les scripts et les binaires.
L’image contient également d’autres configurations pour le conteneur, telles que des variables d’environnement, une commande par défaut à exécuter et d’autres métadonnées.

après avoir installé l’extension de l’amarrage pour VS Code, vous pouvez travailler avec des conteneurs dans VS Code.
En plus des menus contextuels dans le volet de l’ancrage, vous pouvez sélectionner **Terminal**  >  **nouveau terminal** pour ouvrir une fenêtre de ligne de commande.
Vous pouvez également exécuter des commandes dans une fenêtre bash.
sauf indication contraire, toute commande nommée **bash** peut s’exécuter dans une fenêtre bash ou le terminal VS Code.

1. dans VS Code, sélectionnez **terminal**  >  **nouveau terminal**.

1. Exécutez cette commande dans la fenêtre de terminal ou une fenêtre bash.

   ```bash
   docker run -d -p 80:80 docker/getting-started
   ```

   Cette commande contient les paramètres suivants :

   - `-d` Exécutez le conteneur en mode détaché, en arrière-plan.
   - `-p 80:80` Mappez le port 80 de l’hôte au port 80 dans le conteneur.
   - `docker/getting-started` Spécifie l’image à utiliser.

   > [!TIP]
   > Vous pouvez combiner des indicateurs à un seul caractère pour raccourcir la commande complète.
   > Par exemple, la commande ci-dessus peut être écrite comme suit :
   >
   > ```bash
   > docker run -dp 80:80 docker/getting-started
   > ```

1. dans VS Code, sélectionnez l’icône d’ancrage sur la gauche pour afficher l’extension de l’ancrage.

   ![Capture d’écran montre l’extension de l’ancrer avec le didacticiel de démarrage/d’ancrage en cours d’exécution.](media/vs-tutorial-docker-extension.png)

   l’Extension de VS Code d’ancrage affiche les conteneurs en cours d’exécution sur votre ordinateur.
   Vous pouvez accéder aux journaux de conteneur et gérer le cycle de vie des conteneurs, tels que arrêter et supprimer.

   Le nom du conteneur, **modest_schockly** dans cet exemple, est créé de manière aléatoire.
   Le vôtre aura un nom différent.

1. Cliquez avec le bouton droit sur **ancrage/Getting Started** pour ouvrir un menu contextuel.
   Sélectionnez **Ouvrir dans le navigateur**.

   Au lieu de cela, ouvrez un navigateur et entrez `http://localhost/tutorial/` .

   Vous verrez une page, hébergée localement, à propos de DockerLabs.

1. Cliquez avec le bouton droit sur **ancrage/Getting Started** pour ouvrir un menu contextuel.
   Sélectionnez **supprimer** pour supprimer ce conteneur.

   Pour supprimer un conteneur à l’aide de la ligne de commande, exécutez la commande suivante pour obtenir son ID de conteneur :

   ```bash
   docker ps
   ```

   Ensuite, arrêtez et supprimez le conteneur :

   ```bash
   docker stop <container-id>
   docker rm <container-id>
   ```

1. Actualisez votre navigateur.
   La page de Prise en main que vous avez vu il y a un instant a disparu.

## <a name="build-a-container-image-for-the-app"></a>Créer une image conteneur pour l’application

Ce didacticiel utilise une application todo simple.

![Capture d’écran montre l’exemple d’application avec plusieurs éléments ajoutés et une zone de texte et un bouton pour ajouter de nouveaux éléments.](media/todo-list-sample.png)

L’application vous permet de créer des éléments de travail et de les marquer comme terminés ou de les supprimer.

Pour générer l’application, créez un *fichier dockerfile*.
Un fichier dockerfile est un script textuel d’instructions qui est utilisé pour créer une image de conteneur.

1. Accédez au référentiel de l' [ancrage prise en main](https://github.com/docker/getting-started) , puis sélectionnez **code**  >  **Download zip**.
   Extrayez le contenu dans un dossier local.

   ![La capture d’écran montre une partie du site github, avec le bouton de code vert et l’option Télécharger le ZIP en surbrillance.](media/download-zip.png)

1. dans VS Code, sélectionnez **fichier**  >  **ouvrir le dossier**.
   Accédez au dossier de l' *application* dans le projet extrait et ouvrez ce dossier.
   Vous devez voir un fichier nommé *Package. JSON* et deux dossiers nommés *src* et *spec*.

   ![capture d’écran de Visual Studio Code montrant le fichier package. json ouvert avec l’application chargée.](media/ide-screenshot.png)

1. Créez un fichier nommé *fichier dockerfile* dans le même dossier que le fichier *Package. JSON* avec le contenu suivant.

   ```dockerfile
   FROM node:12-alpine
   WORKDIR /app
   COPY . .
   RUN yarn install --production
   CMD ["node", "/app/src/index.js"]
   ```

   > [!NOTE]
   > Assurez-vous que le fichier n’a pas d’extension de fichier comme `.txt` .

1. dans l’explorateur de fichiers, sur la gauche de VS Code, cliquez avec le bouton droit sur *fichier dockerfile* , puis sélectionnez **générer l’Image**.
   Entrez *Getting-Started* comme balise pour l’image dans la zone de saisie de texte.

   La balise est un nom convivial pour l’image.

   Pour créer une image conteneur à partir de la ligne de commande, utilisez la commande suivante.

    ```bash
    docker build -t getting-started .
    ```

    > [!NOTE]
    > Dans une fenêtre bash externe, accédez `app` au dossier qui contient le *fichier dockerfile* pour exécuter cette commande.

Vous avez utilisé *fichier dockerfile* pour créer une image de conteneur.
Vous avez peut-être remarqué que de nombreuses « couches » ont été téléchargées.
Le *fichier dockerfile* démarre à partir de l' `node:12-alpine` image.
À moins qu’il ne soit déjà sur votre ordinateur, cette image devait être téléchargée.

Une fois l’image téléchargée, *fichier dockerfile* copie votre application et utilise `yarn` pour installer les dépendances de votre application.
La `CMD` valeur de *fichier dockerfile* spécifie la commande par défaut à exécuter lors du démarrage d’un conteneur à partir de cette image.

Le `.` à la fin de la `docker build` commande indique à l’ancrage qu’il doit rechercher le *fichier dockerfile* dans le répertoire actif.

## <a name="start-your-app-container"></a>Démarrer votre conteneur d’application

Maintenant que vous avez une image, vous pouvez exécuter l’application.

1. Pour démarrer votre conteneur, utilisez la commande suivante.

   ```bash
   docker run -dp 3000:3000 getting-started
   ```

   Le `-d` paramètre indique que vous exécutez le conteneur en mode détaché, en arrière-plan.
   La `-p` valeur crée un mappage entre le port hôte 3000 et le port de conteneur 3000.
   Sans le mappage de port, vous ne seriez pas en mesure d’accéder à l’application.

1. après quelques secondes, dans VS Code, dans la zone ancrer, sous **conteneurs**, cliquez avec le bouton droit sur **getting-started** , puis sélectionnez **ouvrir dans le navigateur**.
   Vous pouvez ouvrir votre navigateur Web à `http://localhost:3000` la place.

   Vous devez voir l’application en cours d’exécution.

   ![Capture d’écran montre l’exemple d’application sans élément et le texte aucun élément n’est encore ajouté ci-dessus.](media/todo-list-empty.png)

1. Ajoutez un ou deux éléments et vérifiez qu’il fonctionne comme prévu.
   Vous pouvez marquer les éléments comme étant terminés et supprimer des éléments.
   Votre serveur frontal stocke correctement les éléments dans le serveur principal.

## <a name="update-the-code-and-replace-the-container"></a>Mettre à jour le code et remplacer le conteneur

À ce stade, vous avez un gestionnaire de liste de tâches en cours d’exécution avec quelques éléments.
À présent, nous allons apporter quelques modifications et apprendre à gérer vos conteneurs.

1. Dans le fichier, mettez à jour la `src/static/js/app.js` ligne 56 pour utiliser cette nouvelle étiquette de texte :

   ```diff
   - <p className="text-center">No items yet! Add one above!</p>
   + <p className="text-center">You have no todo items yet! Add one above!</p>
   ```

    Enregistrez vos modifications.

1. Arrêtez et supprimez la version actuelle du conteneur.
   Plusieurs Contains ne peuvent pas utiliser le même port.

   Cliquez avec le bouton droit sur le conteneur de **prise en** main, puis sélectionnez **supprimer**.

   ![Capture d’écran montre l’extension de l’ancrage avec un conteneur sélectionné et un menu contextuel avec supprimer les sélectionnés.](media/vs-remove-container.png)

   Ou, à partir de la ligne de commande, utilisez la commande suivante pour récupérer l’ID de conteneur.

   ```bash
   docker ps
   ```

   Ensuite, arrêtez et supprimez le conteneur :

   ```bash
   docker stop <container-id>
   docker rm <container-id>
   ```

1. Générez la version mise à jour de l’image.
   Dans l’Explorateur de fichiers, cliquez avec le bouton droit sur *fichier dockerfile*, puis sélectionnez **générer l’image**.

   Ou, pour générer sur la ligne de commande, utilisez la même commande que celle que vous avez utilisée auparavant.

    ```bash
    docker build -t getting-started .
    ```

1. Démarrez un nouveau conteneur qui utilise le code mis à jour.

    ```bash
    docker run -dp 3000:3000 getting-started
    ```

1. Actualisez votre navigateur `http://localhost:3000` pour voir votre texte d’aide mis à jour.

   ![Capture d’écran montre l’exemple d’application avec le texte modifié, décrit ci-dessus.](media/todo-list-updated-empty-text.png)

## <a name="share-your-image"></a>Partager votre image

Maintenant que vous avez créé une image, vous pouvez la partager.
Pour partager des images de station d’accueil, utilisez un registre d’ancrage.
Le registre par défaut est Dockr Hub, où sont issues toutes les images que nous avons utilisées.

Pour envoyer une image, vous devez d’abord créer un référentiel sur le hub d’ancrage.

1. Accédez au [hub d’ancrage](https://hub.docker.com) et connectez-vous à votre compte.

1. Sélectionnez **créer un référentiel**.

1. Pour le nom de référentiel, entrez `getting-started` .
   Assurez-vous que la **visibilité** est **publique**.

1. Sélectionnez **Create** (Créer).

   À droite de la page, vous verrez une section intitulée commandes de l' **ancrage**.
   Cette section fournit un exemple de commande à exécuter pour effectuer un push vers ce référentiel.

   ![Capture d’écran affiche la page du Hub de l’Ancreur avec une commande d’ancrage recommandée.](media/push-command.png)

1. dans la vue d’ancrage de VS Code, sous **IMAGES**, cliquez avec le bouton droit sur la balise d’image, puis sélectionnez **envoyer**.
   sélectionnez **Connecter registre** , puis **ancrer Hub**.

   Vous devez entrer votre compte, votre mot de passe et un espace de noms de Hub Dockr.

Pour effectuer une transmission de type push vers le hub d’ancrage à l’aide de la ligne de commande, utilisez cette procédure.

1. Connectez-vous au hub de l’Ancreur :

   ```bash
   docker login -u <username>
   ```

1. Utilisez la commande suivante pour attribuer un nouveau nom à l’image de *prise en* main.

    ```bash
    docker tag getting-started <username>/getting-started
    ```

1. Utilisez la commande suivante pour pousser votre conteneur.

    ```bash
    docker push <username>/getting-started
    ```

## <a name="run-the-image-on-a-new-instance"></a>Exécuter l’image sur une nouvelle instance

Maintenant que votre image a été générée et envoyée dans un registre, essayez d’exécuter l’application sur une toute nouvelle instance qui n’a jamais vu cette image de conteneur.
Pour exécuter votre application, utilisez lire avec l’ancrage.

1. Ouvrez votre navigateur pour [jouer avec l’ancrage](http://play-with-docker.com).

1. Connectez-vous avec votre compte d’ancrage Hub.

1. Sélectionnez **Démarrer** , puis cliquez sur le lien **+ Ajouter une nouvelle instance** dans la barre latérale gauche.
   Après quelques secondes, une fenêtre de terminal s’ouvre dans votre navigateur.

   ![Capture d’écran présente le lien lire avec un site d’accueil avec un lien ajouter une nouvelle instance.](media/play-with-docker-add-new-instance.png)

1. Dans le terminal, démarrez votre application.

    ```bash
    docker run -dp 3000:3000 <username>/getting-started
    ```

    La fonction lire avec l’Ancreur attire votre image et la démarre.

1. Sélectionnez le badge **3000** , en regard d' **ouvrir le port**.
   Vous devez voir l’application avec vos modifications.

   Si le badge **3000** ne s’affiche pas, sélectionnez **ouvrir le PORT** et entrez 3000.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Gardez tout ce que vous avez fait jusqu’à présent pour continuer cette série de didacticiels.

## <a name="next-steps"></a>Étapes suivantes

Vous avez terminé ce didacticiel.
Vous avez appris à créer des images de conteneur, à exécuter une application en conteneur, à mettre à jour votre code et à exécuter votre image sur une nouvelle instance.

Voici quelques ressources qui peuvent vous être utiles :

- [Intégration du Cloud de l’ancreur](https://github.com/docker/compose-cli)
- [Exemples](https://github.com/docker/awesome-compose)

Ensuite, essayez le didacticiel suivant de cette série :

> [!div class="nextstepaction"]
> [Rendre persistantes les données et les couches d’une application Dockr](tutorial-persist-data-layer-docker-app-with-vscode.md)
