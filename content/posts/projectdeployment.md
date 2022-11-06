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
## Introduction

Ce tutoriel a pour but de monter une infrastructure Cloud grâce à des outils tels que terraform ou kubernetes. Nous allons déployer notre projet sur [Oracle Cloud Infrastructure](https://www.oracle.com/cloud/compute/) qui va nous permettre de déployer gratuitement 4 noeud kubernetes basés sur une machine fonctionnant sous ARM. Chaque noeud dispose d'un 1 OCPU et de 6 Go de Mémoire et de 50 Go d'espace disque (Arm-based Ampere A1 cores and 24 GB of memory usable as 1 VM or up to 4 VMs with 3,000 OCPU hours and 18,000 GB hours per month). Pour créer un compte gratuit, cliquez [ici](https://www.oracle.com/cloud/free/)

## Installation des outils

Nous allons avoir besoin de kubernetes, de terraform et de OCI CLI

### Installation de terraform

Le détail de l'installation est disponible [ici](https://blog.6adm.in/posts/terraform/)

### Installation de kubernetes

### Installation de OCI CLI

Pour installer OCI CLI qui est l'outil en ligne de commande d'administration d'Oracle Cloud infrastructure, nous allons ouvrir un terminal et lancer la commande suivante:

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

On peut vérifier que l'installation s'est bien passée avec la commande suivante:

```bash
oci --version
```

## Déploiement de l'infrastructure sur le cloud Oracle via Terraform

### Prérequis

En plus de terraform CLI et OCI CLI, nous avons besoin de connaître notre tenancy OCI, c'est le nom que vous avez défini à la création de votre compte.

Nous allons configurer l'OCI CLI depuis notre terminal

```bash
oci session authenticate
```

Ce qui donne:

```bash
Enter a region by index or name(e.g.
1: af-johannesburg-1, 2: ap-chiyoda-1, 3: ap-chuncheon-1, 4: ap-dcc-canberra-1, 5: ap-hyderabad-1,
6: ap-ibaraki-1, 7: ap-melbourne-1, 8: ap-mumbai-1, 9: ap-osaka-1, 10: ap-seoul-1,
11: ap-singapore-1, 12: ap-sydney-1, 13: ap-tokyo-1, 14: ca-montreal-1, 15: ca-toronto-1,
16: eu-amsterdam-1, 17: eu-dcc-milan-1, 18: eu-frankfurt-1, 19: eu-madrid-1, 20: eu-marseille-1,
21: eu-milan-1, 22: eu-paris-1, 23: eu-stockholm-1, 24: eu-zurich-1, 25: il-jerusalem-1,
26: me-abudhabi-1, 27: me-dcc-muscat-1, 28: me-dubai-1, 29: me-jeddah-1, 30: mx-queretaro-1,
31: sa-santiago-1, 32: sa-saopaulo-1, 33: sa-vinhedo-1, 34: uk-cardiff-1, 35: uk-gov-cardiff-1,
36: uk-gov-london-1, 37: uk-london-1, 38: us-ashburn-1, 39: us-gov-ashburn-1, 40: us-gov-chicago-1,
41: us-gov-phoenix-1, 42: us-langley-1, 43: us-luke-1, 44: us-phoenix-1, 45: us-sanjose-1):
```

On choisit la région que nous avons défini lors de la création du compte. Le navigateur se lance pour que vous puissiez vous authentifier et ensuite, il nous indique "Authorization completed! Please close this window and return to your terminal to finish the bootstrap process."

Le terminal affiche le message suivant:

```bash
Config written to: /home/richard/.oci/config

    Try out your newly created session credentials with the following example command:

    oci iam region list --config-file /home/richard/.oci/config --profile DEFAULT --auth security_token
```

Nous poursuivons avec la création d'un profil.

```bash
oci session authenticate
```
il nous demande à nouveau la région et lance le navigateur pour qu'on s'authentifie.

Le terminal affiche le message suivant:

```bash
    Completed browser authentication process!
Enter the name of the profile you would like to create: 
```
On renseigne un nom de profile et on valide, il nous renvoie le message suivant:

```bash
Enter the name of the profile you would like to create: deployinfra
Config written to: /home/richard/.oci/config

    Try out your newly created session credentials with the following example command:

    oci iam region list --config-file /home/richard/.oci/config --profile deployinfra --auth security_token
```
il nous indique ou est stocké le token. il faut savoir que la durée de vie de ce token est de 1 heure. Quand il aura expiré, on pourra le réinitialiser avec la commande suivante:

```bash
oci session refresh --profile deployinfra
```

### Notre premier fichier terraform

Nous allons commencer par créer un dossier qui va contenir le fichier

```bash
mkdir deployinfra
```

On rentre dans le dossier et on crée un fichier main.tf dans lequel nous allons copier le contenu suivant:

```tf
terraform {
  required_providers {
    oci = {
      source = "oracle/oci"
    }
  }
}

provider "oci" {
  region              = "eu-paris-1"
  auth                = "SecurityToken"
  config_file_profile = "deployinfra"
}

resource "oci_core_vcn" "internal" {
  dns_label      = "internal"
  cidr_block     = "172.16.0.0/20"
  compartment_id = "ocid1.tenancy.oc1..aaaaaaaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  display_name   = "My internal VCN"
}
```
Avec region qui correspond à la région que vous avez choisi à la création de votre cloud, config_file_profile est évidemment le nom du profile que nous avons choisi précedement et compartment_id est disponible en cliquant sur l'icone du profile en haut à droite de la console OCI et en sélectionnant Location:VotreUsername

Pour mettre en place notre infrastrcture, on lance la commande terraform init qui va installer les plugins dont nous avons besoin.

```bash
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of oracle/oci...
- Installing oracle/oci v4.96.0...
- Installed oracle/oci v4.96.0 (signed by a HashiCorp partner, key ID 1533A49284137CEB)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has made some changes to the provider dependency selections recorded
in the .terraform.lock.hcl file. Review those changes and commit them to your
version control system if they represent changes you intended to make.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

On peut vérifier notre configuration

```bash
terraform validate
Success! The configuration is valid.
```

Et enfin, on applique notre configuration

```bash
richard@S30:~/Documents/deployinfra$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_vcn.internal will be created
  + resource "oci_core_vcn" "internal" {
      + byoipv6cidr_blocks               = (known after apply)
      + cidr_block                       = "172.16.0.0/20"
      + cidr_blocks                      = (known after apply)
      + compartment_id                   = "ocid1.tenancy.oc1..aaaaaaaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + default_dhcp_options_id          = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_list_id         = (known after apply)
      + defined_tags                     = (known after apply)
      + display_name                     = "My internal VCN"
      + dns_label                        = "internal"
      + freeform_tags                    = (known after apply)
      + id                               = (known after apply)
      + ipv6cidr_blocks                  = (known after apply)
      + ipv6private_cidr_blocks          = (known after apply)
      + is_ipv6enabled                   = (known after apply)
      + is_oracle_gua_allocation_enabled = (known after apply)
      + state                            = (known after apply)
      + time_created                     = (known after apply)
      + vcn_domain_name                  = (known after apply)

      + byoipv6cidr_details {
          + byoipv6range_id = (known after apply)
          + ipv6cidr_block  = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

oci_core_vcn.internal: Creating...
oci_core_vcn.internal: Creation complete after 1s [id=ocid1.vcn.oc1.eu-paris-1.amaaaaaa5nytwhqaaxxxxxxxxxxxxggrxzvpemqfaq5sb66fmzpvq]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
Et si on se rend sur la console OCI, on constate la création de notre réseau cloud virtuel.

![Oracle](/img/oraclenetwork.png)

Quand vous lancez la commande terraform apply, Terraform écrit les données dans un fichier intitulé terraform.tfstate.

Le fichier terraform.tfstate est le seul moyen pour Terraform de suivre les ressources qu'il gère et contient souvent des informations sensibles. Vous devez donc stocker votre fichier d'état en toute sécurité et le distribuer uniquement aux membres de l'équipe de confiance qui doivent gérer votre infrastructure.

On peut inspecter l'état courant avec la commande

```bash
terraform show
```

On peut aussi afficher les ressources utilisées dans notre projet avec la commande

```bash
terraform state list
```

Ce qui donne dans notre cas:

```bash
terraform state list
oci_core_vcn.internal
```
Pour supprimer les ressources du projet, on va utiliser la commande:

```bash
terraform destroy
```
### Premiers tests

On repart de 0.

#### Création de clés RSA

Nous allons maintenant créer des clés RSA pour se connecter via l'API à notre compte OCI.

Nous allons créer un dossier .oci dans notre home directory.

```bash
mkdir $HOME/.oci
```

Nous générons une clé privée de 2048 bit au format PEM.

```bash
openssl genrsa -out $HOME/.oci/<your-rsa-key-name>.pem 2048
```

On change les permissions afin que ce soit uniquement vous qui puisse lire et ecrire la clé privée

```bash
chmod 600 $HOME/.oci/<your-rsa-key-name>.pem
```

On génère la clé publique.

```bash
openssl rsa -pubout -in $HOME/.oci/<your-rsa-key-name>.pem -out $HOME/.oci/<your-rsa-key-name>_public.pem
```

On va copier le contenu de notre clé publique et l'ajouter à notre compte Oracle. Pour cela, on se connecte à son compte, on clique sur notre avatar en haut à droite et on va sur "Mon profil".

![oci](/img/oci1.png)

On a un menu en bas à gauche dans lequel nous avons une rubrique "Clés d'API".

![oci](/img/oci2.png)

On clique dessus et dans la nouvelle fenêtre, on clique sur "Ajouter une clé API". Dans la nouvelle fenêtre, on clique sur "Coller une clé publique" et on copie colle le contenu de notre clé en gardant les lignes BEGIN PUBLIC KEY et END PUBLIC KEY.

![oci](/img/oci3.png)

On clique sur ajouter.

![oci](/img/oci4.png)

Il nous indique comment configurer le fichier config du dossier .oci qui se trouve dans notre home directory.

On peut dès lors se connecter à notre compte via les clés RSA.


#### Liste des informations requises

Pour utiliser Terraform sur notre infrastructure Cloud Oracle, nous devons recueillir certaines informations disponibles depuis la console Oracle Cloud Infrastructure.

- tenancy_ocid: On retrouve cette information à partir de notre avatar, on va sur Location : \<your-tenancy> et on copie l'OCID.
- user_ocid: On retrouve cette information en cliquant sur notre utilisateur.

![oci](/img/oci14.png)

- private_key_path: chemin d'accès à la clé privée RSA que vous avez créée plus haut. Exemple : $HOME/.oci/\<your-rsa-key-name>.pem.
- fingerprint : On retrouve cette information à partir de notre avatar, on va sur "Mon profil", on clique sur clés d'API et on copie l'empreinte associée à la clé publique RSA que vous avez créée plus haut. Le format est : xx:xx:xx…xx.
- region : Vous trouverez votre région dans la bannière supérieure de la console Oracle. Si vous cliquez dessus, il vous donnera le terme exact relatif à votre région, ex: eu-paris-1

#### Authentification

Nous allons créer trois fichiers pour 
- L'authentification
- Récuperer les données de notre compte
- imprimer les outputs

##### Création des fichiers

On va créer un dossier qui va contenir nos fichiers terraform.

On y crée un fichier nommé provider.tf comme ci-dessous en remplacant les données entre crochets par les valeurs récuperées ci-dessus tout en gardant les guillemets.

```
provider "oci" {
  tenancy_ocid = "<tenancy-ocid>"
  user_ocid = "<user-ocid>" 
  private_key_path = "<rsa-private-key-path>"
  fingerprint = "<fingerprint>"
  region = "<region-identifier>"
}
```

##### Test

Afin de tester notre authentification, nous allons afficher les domaines disponibles dans notre tenant: [oci_identity_availability_domains](https://registry.terraform.io/providers/oracle/oci/latest/docs/data-sources/identity_availability_domains)

Nous créons donc un fichier domaine-dispo.tf avec le contenu suivant

```
data "oci_identity_availability_domains" "ads" {
compartment_id = "ocid1.tenancy.oc1..aaaaaaaauxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxc2fc2b4qya"
}
```

Et on crée un fichier outputs.tf avec le contenu suivant:

```
output "show-ads" {
  value = "${data.oci_identity_availability_domains.ads.availability_domains}"
}
```

Ce qui donne:

```bash
 terraform apply
data.oci_identity_availability_domains.ads: Reading...
data.oci_identity_availability_domains.ads: Read complete after 2s [id=IdentityAvailabilityDomainsDataSource-367745787]

Changes to Outputs:
  + show-ads = [
      + {
          + compartment_id = "ocid1.tenancy.oc1...aaaaaaaauxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxc2fc2b4qya"
          + id             = "ocid1.availabilitydomain.oc1..aaaaaaaaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx7weba4e5ga"
          + name           = "Tgrg:EU-PARIS-1-AD-1"
        },
    ]

You can apply this plan to save these new output values to the Terraform state, without changing any real infrastructure.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes


Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

show-ads = tolist([
  {
    "compartment_id" = "ocid1.tenancy.oc1...aaaaaaaauxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxc2fc2b4qya"
    "id" = "ocid1.availabilitydomain.oc1..aaaaaaaaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx7weba4e5ga"
    "name" = "Tgrg:EU-PARIS-1-AD-1"
  },
])
```

Dans le cas présent, nous avons un seul domaine mais surtout nous constatons que notre authentification fonctionne.

### Création d'une instance
#### Création manuelle d'une instance.

Dans cette partie, nous allons déployer une instance composée d'une image ubuntu 22.04 sur une VM standard A1 FLex.

![oci](/img/oci15.png)

![oci](/img/oci16.png)

Dans la partie réseau, nous avons une adresse ip privée et une adresse ip publique grâce à laquelle nous allons pouvoir nous y connecter en ssh et aussi exposer un site web.

![oci](/img/oci17.png)

Ensuite, nous avons la possibilité d'uploader notre clé publique ssh pour se connecter directement à notre VM en ssh.

![oci](/img/oci19.png)

On a aussi une rubrique concernant le boot volume.

![oci](/img/oci20.png)

Enfin, on a une rubrique qui nous propose la mise en place d'un script d'initialisation.

![oci](/img/oci20b.png)

Quand c'est fini et que nous faisons create, nous disposons d'une machine à laquelle nous pouvons nous connecter en ssh après avoir récuperé l'IP publique dans la console d'administration.

Nous allons donc nous connecter pour y installer un serveur web grâce aux commandes suivantes:

```bash
sudo apt-get update
sudo apt-get install apache2
```

Par contre, si nous lancons un navigateur et tentons de renseigner l'adresse IP publique de notre serveur, cela ne fonctionne pas. Il faut autoriser le port 80 dans notre console d'administration Oracle.

Pour cela, on se rend sur Networking puis Virtual Cloud Networks. On clique sur notre VCN

![oci](/img/oci21.png)

Puis notre subnet

![oci](/img/oci22.png)

Et enfin sur notre security list

![oci](/img/oci23.png)

On arrive sur les Ingress Rule, on clique sur Add Ingress Rule

![oci](/img/oci24.png)

Et dans la nouvelle fenêtre, on autorise la connexion sur le port 80 comme suit:

![oci](/img/oci25.png)

On refait un test de connexion à notre serveur web mais ca ne fonctionne toujours pas.

Nous devons aussi autoriser la connexion sur notre serveur au niveau iptables. On se connecte donc sur notre machine et on lance les commandes suivantes:

```bash
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo netfilter-persistent save
```
Si nous essayons de nouveau d'accéder à notre serveur web, cela fonctionne:

![oci](/img/oci26.png)

Maintenant que nous connaissons les étapes pour créer un serveur web basé sur ubuntu fonctionnant sur une machine ARM, nous allons pouvoir créer des fichiers terraform pour arriver au même résultat.

#### Création d'une instance hébergeant un serveur web via terraform.

Au vu de l'étape précedente, nous savons de quoi nous avons besoin pour créer notre machine via terraform.

Commencons par créer un [VCN](https://registry.terraform.io/providers/oracle/oci/latest/docs/resources/core_vcn)

```
resource "oci_core_vcn" "vcn" {
  cidr_block = "10.1.0.0/16"
  compartment_id = var.compartment_ocid
  display_name = "vcn-${var.hostname}"
  dns_label = "vcn${var.hostname}"
}
```

On stockera les variables dans le fichier variables.tf

```
variable "compartment_ocid" {
  # OCID of your OCI Account compartment
  default = "xxxxxx"
}

variable "hostname" {
  default = "xxxxxx"
}
```

Voici ce que nous avons actuellement suite à la création via l'interface

![oci](/img/oci27.png)

Et voici le résultat de la commande

```bash
terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_vcn.vcn will be created
  + resource "oci_core_vcn" "vcn" {
      + byoipv6cidr_blocks               = (known after apply)
      + cidr_block                       = "10.1.0.0/16"
      + cidr_blocks                      = (known after apply)
      + compartment_id                   = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3mozxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx4qya"
      + default_dhcp_options_id          = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_list_id         = (known after apply)
      + defined_tags                     = (known after apply)
      + display_name                     = "vcn-kube"
      + dns_label                        = "vcnkube"
      + freeform_tags                    = (known after apply)
      + id                               = (known after apply)
      + ipv6cidr_blocks                  = (known after apply)
      + ipv6private_cidr_blocks          = (known after apply)
      + is_ipv6enabled                   = (known after apply)
      + is_oracle_gua_allocation_enabled = (known after apply)
      + state                            = (known after apply)
      + time_created                     = (known after apply)
      + vcn_domain_name                  = (known after apply)

      + byoipv6cidr_details {
          + byoipv6range_id = (known after apply)
          + ipv6cidr_block  = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

oci_core_vcn.vcn: Creating...
oci_core_vcn.vcn: Creation complete after 2s [id=ocid1.vcn.oc1.eu-paris-1.amaaaaaa5nytwhqa62exmbafamzgcdqhf6uuoqogygcdatdttxnm6azufidq]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
Ce qui nous donne au niveau de l'interface

![oci](/img/oci28.png)

Paramtétrage des accés sortants

Nous avons besoin d'une passerelle internet et d'une table de routage pour les accès externes.

```
resource "oci_core_internet_gateway" "internet_gateway" {
  compartment_id = var.compartment_ocid
  display_name = "ig-${var.hostname}"
  vcn_id = oci_core_vcn.vcn.id
}
```

```
resource "oci_core_default_route_table" "default_route_table" {
  manage_default_resource_id = oci_core_vcn.vcn.default_route_table_id
  display_name = "rt-${var.hostname}"

  route_rules {
    destination = "0.0.0.0/0"
    destination_type = "CIDR_BLOCK"
    network_entity_id = oci_core_internet_gateway.internet_gateway.id
  }
}
```

Ensuite, nous créons un Network Security Group.

```
resource "oci_core_network_security_group" "nsg" {
  compartment_id = var.compartment_ocid
  vcn_id = oci_core_vcn.vcn.id
  display_name = "nsg-${var.hostname}"
}
```

Nous allons autoriser tous les accès sortants

```
resource "oci_core_network_security_group_security_rule" "nsg_outbound" {
  network_security_group_id = "${oci_core_network_security_group.nsg.id}"
  direction = "EGRESS"
  protocol = "all"
  description = "nsg-${var.hostname}-outbound"
  destination = "0.0.0.0/0"
  destination_type = "CIDR_BLOCK"
}
```

Si nous faisons un terraform plan à ce stade, il nous dit:

```
$ terraform plan
oci_core_vcn.vcn: Refreshing state... [id=ocid1.vcn.oc1.eu-paris-1.amaaaaaa5nytwhqa62exmbafamzgcdqhf6uuoqogygcdatdttxnm6azufidq]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_default_route_table.default_route_table will be created
  + resource "oci_core_default_route_table" "default_route_table" {
      + compartment_id             = (known after apply)
      + defined_tags               = (known after apply)
      + display_name               = "rt-kube"
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + manage_default_resource_id = "ocid1.routetable.oc1.eu-paris-1.aaaaaaaatk4wewk3gfz2voan577y2sl7a46jf5v7y4f7gwqfxrhcshnqdyma"
      + state                      = (known after apply)
      + time_created               = (known after apply)

      + route_rules {
          + cidr_block        = (known after apply)
          + description       = (known after apply)
          + destination       = "0.0.0.0/0"
          + destination_type  = "CIDR_BLOCK"
          + network_entity_id = (known after apply)
          + route_type        = (known after apply)
        }
    }

  # oci_core_internet_gateway.internet_gateway will be created
  + resource "oci_core_internet_gateway" "internet_gateway" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yjsjav47jx4l4ic2fc2b4qya"
      + defined_tags   = (known after apply)
      + display_name   = "ig-kube"
      + enabled        = true
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = "ocid1.vcn.oc1.eu-paris-1.amaaaaaa5nytwhqa62exmbafamzgcdqhf6uuoqogygcdatdttxnm6azufidq"
    }

  # oci_core_network_security_group.nsg will be created
  + resource "oci_core_network_security_group" "nsg" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yjsjav47jx4l4ic2fc2b4qya"
      + defined_tags   = (known after apply)
      + display_name   = "nsg-kube"
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = "ocid1.vcn.oc1.eu-paris-1.amaaaaaa5nytwhqa62exmbafamzgcdqhf6uuoqogygcdatdttxnm6azufidq"
    }

  # oci_core_network_security_group_security_rule.nsg_outbound will be created
  + resource "oci_core_network_security_group_security_rule" "nsg_outbound" {
      + description               = "nsg-kube-outbound"
      + destination               = "0.0.0.0/0"
      + destination_type          = "CIDR_BLOCK"
      + direction                 = "EGRESS"
      + id                        = (known after apply)
      + is_valid                  = (known after apply)
      + network_security_group_id = (known after apply)
      + protocol                  = "all"
      + source_type               = (known after apply)
      + stateless                 = (known after apply)
      + time_created              = (known after apply)
    }

Plan: 4 to add, 0 to change, 0 to destroy.
```

On retrouve nos infos dans l'interface

![oci](/img/oci30.png)

![oci](/img/oci31.png)

![oci](/img/oci32.png)

Poursuivons en créant un sous-réseau.

```
resource "oci_core_subnet" "subnet" {
  availability_domain = data.oci_identity_availability_domain.ad.name
  cidr_block = "10.1.0.0/24"
  display_name = "subnet-${var.hostname}"
  dns_label = "subnet${var.hostname}"
  security_list_ids = [
    oci_core_security_list.empty_security_list.id]
  compartment_id = var.compartment_ocid
  vcn_id = oci_core_vcn.vcn.id
  route_table_id = oci_core_vcn.vcn.default_route_table_id
  dhcp_options_id = oci_core_vcn.vcn.default_dhcp_options_id
}
```

 Et une security list qui sera vide par défaut.

```
# create empty security list to avoid using 'default' with open 22
resource "oci_core_security_list" "empty_security_list" {
  compartment_id = var.compartment_ocid
  vcn_id = oci_core_vcn.vcn.id
  display_name = "seclist-${var.hostname}"
}
```

Nous configurons ensuite les accès entrants


On autorise le ssh

```
resource "oci_core_network_security_group_security_rule" "nsg_inbound_ssh" {
  network_security_group_id = "${oci_core_network_security_group.nsg.id}"
  direction = "INGRESS"
  protocol = "6" # TCP
  description = "nsg-${var.hostname}-inbound-ssh"
  source = "${data.dns_a_record_set.bastion-host.addrs[0]}/32"
  source_type = "CIDR_BLOCK"
  destination = "${module.vminst.public_ip}/32"
  destination_type = "CIDR_BLOCK"
  tcp_options {
    destination_port_range {
      min = 22
      max = 22
    }
  }
}
```

et le http

```
resource "oci_core_network_security_group_security_rule" "nsg_inbound_http" {
  network_security_group_id = "${oci_core_network_security_group.nsg.id}"
  direction = "INGRESS"
  protocol = "6" # TCP
  description = "nsg-${var.hostname}-inbound-http"
  source = "${data.dns_a_record_set.bastion-host.addrs[0]}/32"
  source_type = "CIDR_BLOCK"
  destination = "${module.vminst.public_ip}/32"
  destination_type = "CIDR_BLOCK"
  tcp_options {
    destination_port_range {
      min = 80
      max = 80
    }
  }
}
```

<!---
https://www.lightenna.com/tech/2020/create-oracle-cloud-vm-using-terraform/
-->
