---
title: "Mise en place d'un cluster kubernetes sur des raspberry pi"
summary: Dans ce tutoriel, nous allons mettre en place un cluster kubernetes sur des rapsberry pi.
date: 2022-11-14T00:00:00+00:00
draft: false
cover:
    image: img/raspberry.png
    alt: "traefik" # alt text
tags: ["kubernetes", "devops", "raspberry pi"]
ShowToc: true
---

# Le matériel


- 1 Raspberry Pi 4 Model B 4Gb
- 5 Raspberry Pi 4 Model B 2Gb
- 1 Anker PowerPort 10 Chargeur 60W 10 Ports USB
![anker](/img/anker.jpg)
- 6 câbles USB courts
![cable](/img/cable.jpg)
- Boitier pour raspberry
![boitier](/img/boitier.jpg)
- 6 cartes SD
![cable](/img/sdcard.jpg)
- 1 switch gigabit 8 ports
- 7 cables réseaux

# Configuration des raspberry

Pour installer le système d'exploitation sur une carte SD, nous allons utiliser le programme [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

![rpimanager](/img/rpimanager.png)

On clique sur le bouton "Choisissez l'OS" puis sur Raspberry PI OS (Other) et dans la liste, on choisit Raspberry PI OS Lite (64-bit). Nous prenons la version lite car nous n'avons pas besoin de l'interface graphique.

Quand nous avons séléctionné notre image, nous avons un bouton d'option qui est apparu en bas à droite de la fenêtre.

![rpimanager](/img/rpi2.png)

Quand on clique dessus, nous avons la possibilité de configurer différenetes options:

![rpimanager](/img/rpi3.png)

On peut spécifier le hostname, activer le ssh, renseigner le username et le mot de passe et paramétrer les options régionales.


Comme, nous allons configurer plusieurs carte SD, nous allons sélectionner l'option "always use" pour ne pas avoir à ressaisir les mêmes informations à chaque fois.

On peut dès lors, séléctionner notre carte SD et cliquer sur Ecrire pour copier le système d'exploitation sur la carte. Il ne reste plus qu'à l'inserer dans notre raspberry.

Lorsque notre premier raspberry est branché au réseau, on l'allume. Il va automatiquement recevoir une adresse IP. Nous allons configurer une IP statique. Mais avant tout, nous devons connaitre l'IP actuelle pour ensuite se connecter à notre raspberry et faire la modification. Pour cela, nous allons scanner notre réseau avec la commande nmap

```bash
richard@mint:~$ nmap 192.168.1.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-15 19:42 CET
...

Nmap scan report for 192.168.1.29
Host is up (0.011s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh

...

```

On se connecte ensute à notre raspberry en SSH et nous allons éditer le fichier /etc/dhcpcd.conf et nous y rajoutons le contenu suivant.

```
interface eth0
static ip_address=192.168.1.200/24
static routers=192.168.1.254
static domain_name_servers=8.8.8.8 192.168.1.254
```

Ici, nous donnons l'IP 192.168.1.200 à la carte réseau filaire.

Ensuite, si vous souhaitez changer le hostname, vous pouvez le faire avec la commande suivante:

```bash
sudo raspi-config
```

Vous avez aussi la possibilité d'agrandir la partition systéme.

On va procéder de la même manière pour l'ensemble de nos rapsberry...

Nous avons donné le hostname master au raspberry qui a 4 Go de RAM et worker1 à worker5 pour les autres raspberry qui ont 2Go.

Nous allons ensuite installer docker sur le master afin de générer les images docker que nous allons pousser sur le registry. En effet, notre raspberry a un processeur arm. Il est possible de générer des images déstinée au processeur arm depuis un PC sous x86 avec le logiciel docker-desktop mais nous comme nous avons une plateforme ARM autant en profiter...

```bash
sudo apt-get update && sudo apt-get upgrade
```

On télécharge le script d’installation qui va installer Docker sur Raspberry Pi

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

puis

```bash
sudo sh get-docker.sh
```

On donne les droits à notre utilisateur courant

```bash
sudo usermod -aG docker pi
```

Nous allons ensuite passer sur chacunes des machines et modifier le fichier /etc/hosts avec le contenu suivant:

```
192.168.1.200 master
192.168.1.201 worker1
192.168.1.202 worker2
192.168.1.203 worker3
192.168.1.204 worker4
192.168.1.205 worker5
```

Ensuite, nous allons éditer le ficher /boot/cmdline.txt et rajouter ceci à la suite de la ligne existante, il ne faut pas faire de retour à la ligne:

```bash
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

On va ensuite installer k3s

```bash
curl -sfL https://get.k3s.io | sh -
```

```bash
pi@master:~ $ curl -sfL https://get.k3s.io | sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.25.3+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.25.3+k3s1/sha256sum-arm64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.25.3+k3s1/k3s-arm64
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
```

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
K10eb52c63750982053d2aebfe5dbe03dd0954866e0aaa118a19d906f7fa42274e2::server:a41d055bed4d29426c79316d8fb53207
```

```bash
sudo su -
curl -sfL https://get.k3s.io | K3S_TOKEN="K10eb52c63750982053d2aebfe5dbe03dd0954866e0aaa118a19d906f7fa42274e2::server:a41d055bed4d29426c79316d8fb53207" K3S_URL="https://192.168.1.200:6443" K3S_NODE_NAME="k3s-node1" sh -
```

```bash
root@worker1:~# curl -sfL https://get.k3s.io | K3S_TOKEN="K10eb52c63750982053d2aebfe5dbe03dd0954866e0aaa118a19d906f7fa42274e2::server:a41d055bed4d29426c79316d8fb53207" K3S_URL="https://192.168.1.200:6443" K3S_NODE_NAME="k3s-node1" sh -[INFO]  Finding release for channel stable
[INFO]  Using v1.25.3+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.25.3+k3s1/sha256sum-arm64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.25.3+k3s1/k3s-arm64
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-agent-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent
```

```bash
pi@master:~ $sudo kubectl get nodes
NAME        STATUS   ROLES                  AGE     VERSION
k3s-node3   Ready    <none>                 5m4s    v1.25.3+k3s1
k3s-node4   Ready    <none>                 4m16s   v1.25.3+k3s1
master      Ready    control-plane,master   112m    v1.25.3+k3s1
k3s-node1   Ready    <none>                 8m13s   v1.25.3+k3s1
k3s-node5   Ready    <none>                 2m6s    v1.25.3+k3s1
k3s-node2   Ready    <none>                 6m1s    v1.25.3+k3s1
```

Création des images à pousser sur le registry docker

```bash
docker build -t dashboard .
```

```bash
docker tag dashboard registry.cruz.im/dashboard:v1.1.0
```

```bash
docker push registry.cruz.im/dashboard:v1.1.0
```