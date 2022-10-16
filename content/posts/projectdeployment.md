---
title: "Déploiement sur le cloud d'un cluster kubernetes"
summary: La finalité de ce tutoriel est de mettre en place un process d'automatisation de création d'une infrastructure sur le cloud et d'y déployer un projet personnel.
date: 2022-10-16T01:00:00+02:00
draft: false
cover:
    image: img/devops.png
    alt: "devops" # alt text
tags: ["devops", "terraform", "kubernetes", "cloud"]  
ShowToc: true
---
# Introduction

Ce tutoriel a pour but de monter une infrastructure Cloud grâce à des outils tels que terraform ou kubernetes. Nous allons déployer notre projet sur [Oracle Cloud Infrastructure](https://www.oracle.com/cloud/compute/) qui va nous permettre de déployer gratuitement 4 noeud kubernetes basés sur une machine fonctionnant sous ARM. Chaque noeud dispose d'un 1 OCPU et de 6 Go de Mémoire et de 50 Go d'espace disque (Arm-based Ampere A1 cores and 24 GB of memory usable as 1 VM or up to 4 VMs with 3,000 OCPU hours and 18,000 GB hours per month). Pour créer un compte gratuit, cliquez [ici](https://www.oracle.com/cloud/free/)
