---
title: "Présentation de terraform"
summary: Dans cette article, vous trouverez une présentation global de terraform
date: 2022-10-23T01:00:00+02:00
draft: false
cover:
    image: img/terraform.png
    alt: "terraform" # alt text
tags: ["devops", "terraform", "cloud"]  
ShowToc: true
---

# Introduction

HashiCorp Terraform est un outil d'infrastructure ascode qui vous permet de définir à la fois des ressources cloud et on premise à partir de fichiers de configuration lisibles par l'homme que vous pouvez versionner, réutiliser et partager. Vous pouvez ensuite utiliser un flux de travail cohérent pour provisionner et gérer l'ensemble de votre infrastructure tout au long de son cycle de vie. Terraform peut gérer des composants de bas niveau comme les ressources de calcul, de stockage et de mise en réseau, ainsi que des composants de haut niveau comme les entrées DNS et les fonctionnalités SaaS.

## Comment fonctionne Terraform ?

Terraform crée et gère des ressources sur des plates-formes cloud et d'autres services via leurs interfaces de programmation d'applications (API). Les fournisseurs permettent à Terraform de fonctionner avec pratiquement n'importe quelle plate-forme ou service avec une API accessible.

![terraform](/img/terraformapi.png)

HashiCorp et la communauté Terraform ont déjà écrit plus de 1700 fournisseurs pour gérer des milliers de types de ressources et de services différents, et ce nombre continue de croître. Vous pouvez trouver tous les fournisseurs accessibles au public sur le registre [Terraform](https://registry.terraform.io/) , y compris Amazon Web Services (AWS), Azure, Google Cloud Platform (GCP), Kubernetes, Helm, GitHub, Splunk, DataDog et bien d'autres.

Le workflow principal de Terraform se compose de trois étapes :

- **Écrire** : vous définissez des ressources, qui peuvent se trouver sur plusieurs fournisseurs et services cloud. Par exemple, vous pouvez créer une configuration pour déployer une application sur des machines virtuelles dans un réseau Virtual Private Cloud (VPC) avec des groupes de sécurité et un équilibreur de charge.
- **Plan** : Terraform crée un plan d'exécution décrivant l'infrastructure qu'il va créer, mettre à jour ou détruire en fonction de l'infrastructure existante et de votre configuration.
- **Appliquer** : après approbation, Terraform exécute les opérations proposées dans le bon ordre, en respectant toutes les dépendances de ressources. Par exemple, si vous mettez à jour les propriétés d'un VPC et modifiez le nombre de machines virtuelles dans ce VPC, Terraform recréera le VPC avant de redimensionner les machines virtuelles.

![workflow terraform](/img/workflow.png)

## Pourquoi utiliser Terraform ?

### Gérer n'importe quelle infrastructure
Trouvez des fournisseurs pour de nombreuses plates-formes et services que vous utilisez déjà dans le [registre Terraform](https://registry.terraform.io/) . Vous pouvez également écrire [le vôtre](https://developer.hashicorp.com/terraform/plugin) . Terraform adopte une approche immuable de l'infrastructure , réduisant la complexité de la mise à niveau ou de la modification de vos services et de votre infrastructure.

### Suivez votre infrastructure
Terraform génère un plan et vous demande votre approbation avant de modifier votre infrastructure. Il garde également une trace de votre infrastructure réelle dans un [fichier d'état](https://developer.hashicorp.com/terraform/language/state) , qui agit comme une source de vérité pour votre environnement. Terraform utilise le fichier d'état pour déterminer les modifications à apporter à votre infrastructure afin qu'elle corresponde à votre configuration.

### Automatisez les changements
Les fichiers de configuration Terraform sont déclaratifs, ce qui signifie qu'ils décrivent l'état final de votre infrastructure. Vous n'avez pas besoin d'écrire des instructions détaillées pour créer des ressources, car Terraform gère la logique sous-jacente. Terraform construit un graphique de ressources pour déterminer les dépendances des ressources et crée ou modifie des ressources non dépendantes en parallèle. Cela permet à Terraform de provisionner efficacement les ressources.

### Standardiser les configurations
Terraform prend en charge des composants de configuration réutilisables appelés modules qui définissent des collections configurables d'infrastructure, ce qui permet de gagner du temps et d'encourager les meilleures pratiques. Vous pouvez utiliser des modules accessibles au public à partir du registre Terraform ou écrire les vôtres.

### Collaborer
Étant donné que votre configuration est écrite dans un fichier, vous pouvez la valider dans un système de contrôle de version (VCS) et utiliser Terraform Cloud pour gérer efficacement les flux de travail Terraform entre les équipes. Terraform Cloud exécute Terraform dans un environnement cohérent et fiable et fournit un accès sécurisé aux données d'état et secrètes partagées, des contrôles d'accès basés sur les rôles, un registre privé pour partager les modules et les fournisseurs, et plus encore.

# HCL


# Les providers
# Les actions
# Dépendances des ressources
# Les provisionners
# Les variables
