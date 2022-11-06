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
#### Création manuelle d'une instance hébergeant un serveur web.

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

Créons un fichier network.tf avec le contenu suivant:

```
resource "oci_core_vcn" "vcn" {
  cidr_block = "10.1.0.0/16"
  compartment_id = var.compartment_ocid
  display_name = "vcn-${var.hostname}"
  dns_label = "vcn${var.hostname}"
}

resource "oci_core_internet_gateway" "internet_gateway" {
  compartment_id = var.compartment_ocid
  display_name = "ig-${var.hostname}"
  vcn_id = oci_core_vcn.vcn.id
}

resource "oci_core_default_route_table" "default_route_table" {
  manage_default_resource_id = oci_core_vcn.vcn.default_route_table_id
  display_name = "rt-${var.hostname}"

  route_rules {
    destination = "0.0.0.0/0"
    destination_type = "CIDR_BLOCK"
    network_entity_id = oci_core_internet_gateway.internet_gateway.id
  }
}

resource "oci_core_network_security_group" "nsg" {
  compartment_id = var.compartment_ocid
  vcn_id = oci_core_vcn.vcn.id
  display_name = "nsg-${var.hostname}"
}

resource "oci_core_network_security_group_security_rule" "nsg_outbound" {
  network_security_group_id = "${oci_core_network_security_group.nsg.id}"
  direction = "EGRESS"
  protocol = "all"
  description = "nsg-${var.hostname}-outbound"
  destination = "0.0.0.0/0"
  destination_type = "CIDR_BLOCK"
}

resource "oci_core_subnet" "subnet" {
  cidr_block        = "10.1.0.0/24"
  display_name      = "subnet-${var.hostname}"
  dns_label = "subnet${var.hostname}"
  compartment_id    = var.compartment_ocid
  vcn_id = oci_core_vcn.vcn.id
  route_table_id = oci_core_vcn.vcn.default_route_table_id
  dhcp_options_id = oci_core_vcn.vcn.default_dhcp_options_id
  security_list_ids = [oci_core_security_list.sl.id]

  provisioner "local-exec" {
    command = "sleep 5"
  }
}

resource "oci_core_security_list" "sl" {
  compartment_id = var.compartment_ocid
  display_name   = "seclist-${var.hostname}"
  vcn_id         = oci_core_vcn.vcn.id

  egress_security_rules {
    destination = "0.0.0.0/0"
    protocol    = "6"
  }

  ingress_security_rules {

    protocol = "6"
    source   = "0.0.0.0/0"

    tcp_options {
      max = 22
      min = 22
    }
  }

  ingress_security_rules {
    protocol = "6"
    source   = "0.0.0.0/0"

    tcp_options {
      max = 80
      min = 80
    }
  }
}

```

On a aussi un fichier variables.tf avec le contenu suivant :

```
variable "compartment_ocid" {
  # OCID of your OCI Account compartment
  default = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}

variable "hostname" {
  default = "srvweb"
}
```

Faisons un 

```
terraform plan
```

Pour nous assurer que tout est OK

```
terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_default_route_table.default_route_table will be created
  + resource "oci_core_default_route_table" "default_route_table" {
      + compartment_id             = (known after apply)
      + defined_tags               = (known after apply)
      + display_name               = "rt-srvweb"
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + manage_default_resource_id = (known after apply)
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
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "ig-srvweb"
      + enabled        = true
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)
    }

  # oci_core_network_security_group.nsg will be created
  + resource "oci_core_network_security_group" "nsg" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "nsg-srvweb"
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)
    }

  # oci_core_network_security_group_security_rule.nsg_outbound will be created
  + resource "oci_core_network_security_group_security_rule" "nsg_outbound" {
      + description               = "nsg-srvweb-outbound"
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

  # oci_core_security_list.sl will be created
  + resource "oci_core_security_list" "sl" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "seclist-srvweb"
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)

      + egress_security_rules {
          + description      = (known after apply)
          + destination      = "0.0.0.0/0"
          + destination_type = (known after apply)
          + protocol         = "6"
          + stateless        = (known after apply)
        }

      + ingress_security_rules {
          + description = (known after apply)
          + protocol    = "6"
          + source      = "0.0.0.0/0"
          + source_type = (known after apply)
          + stateless   = false

          + tcp_options {
              + max = 22
              + min = 22
            }
        }
      + ingress_security_rules {
          + description = (known after apply)
          + protocol    = "6"
          + source      = "0.0.0.0/0"
          + source_type = (known after apply)
          + stateless   = false

          + tcp_options {
              + max = 80
              + min = 80
            }
        }
    }

  # oci_core_subnet.subnet will be created
  + resource "oci_core_subnet" "subnet" {
      + availability_domain        = (known after apply)
      + cidr_block                 = "10.1.0.0/24"
      + compartment_id             = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags               = (known after apply)
      + dhcp_options_id            = (known after apply)
      + display_name               = "subnet-srvweb"
      + dns_label                  = "subnetsrvweb"
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + ipv6cidr_block             = (known after apply)
      + ipv6cidr_blocks            = (known after apply)
      + ipv6virtual_router_ip      = (known after apply)
      + prohibit_internet_ingress  = (known after apply)
      + prohibit_public_ip_on_vnic = (known after apply)
      + route_table_id             = (known after apply)
      + security_list_ids          = (known after apply)
      + state                      = (known after apply)
      + subnet_domain_name         = (known after apply)
      + time_created               = (known after apply)
      + vcn_id                     = (known after apply)
      + virtual_router_ip          = (known after apply)
      + virtual_router_mac         = (known after apply)
    }

  # oci_core_vcn.vcn will be created
  + resource "oci_core_vcn" "vcn" {
      + byoipv6cidr_blocks               = (known after apply)
      + cidr_block                       = "10.1.0.0/16"
      + cidr_blocks                      = (known after apply)
      + compartment_id                   = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + default_dhcp_options_id          = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_list_id         = (known after apply)
      + defined_tags                     = (known after apply)
      + display_name                     = "vcn-srvweb"
      + dns_label                        = "vcnsrvweb"
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

Plan: 7 to add, 0 to change, 0 to destroy.
```

on peut donc faire un :

```
terraform apply
```

Si on se connecte à l'interface, on retrouve notre Virtual Cloud Network

![oci](/img/oci27.png)

Si on clique dessus, on a:

![oci](/img/oci28.png)

On retrouve les mêmes paramètres que pour notre VCN créé manuellement.

Nous allons poursuivre avec la création de notre compute instance, nous créons alors un fichier intitulé compute.tf et le remplissons avec le contenu suivant:

```
resource "oci_core_instance" "compute_instance" {
  availability_domain = var.availablity_domain_name == "" ? data.oci_identity_availability_domains.ads.availability_domains[0]["name"] : var.availablity_domain_name
  compartment_id      = var.compartment_ocid
  display_name        = "${var.hostname}"
  shape               = var.instance_shape
  fault_domain        = "FAULT-DOMAIN-1"

  shape_config {
    ocpus         = var.instance_ocpus
    memory_in_gbs = var.instance_shape_config_memory_in_gbs
  }

   metadata = {
        ssh_authorized_keys = file("/home/richard/.ssh/oci.pub")
    } 

  create_vnic_details {
    subnet_id                 = oci_core_subnet.subnet.id
    display_name              = "vnic-${var.hostname}"
    assign_public_ip          = true
    assign_private_dns_record = true
  }

  source_details {
    source_type             = "image"
    source_id               = "ocid1.image.oc1.eu-paris-1.aaaaaaaa2kukypyttuyb6vkpjbdrzl5dm2cg7mxniigjdukbfvelwlesrurq"
    boot_volume_size_in_gbs = "50"
  }

  timeouts {
    create = "60m"
  }
}
```

On crée dans la foulée un fichier datasources.tf avec le contenu suivant :

```
data "oci_identity_availability_domains" "ads" {
  compartment_id = var.tenancy_ocid
}
```

ainsi qu'un fichier terraform.tfvars

```
# Authentication
tenancy_ocid         = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Region
region = "eu-paris-1"

# Compartment
compartment_ocid = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

Et on modifie le fichier variables.tf

```
variable "region" {}

variable "compartment_ocid" {
}

variable "tenancy_ocid" {
}

variable "availablity_domain_name" {
  default = ""
}

variable "hostname" {
  default = "srvweb"
}

variable "instance_shape" {
  description = "Instance Shape"
  default     = "VM.Standard.A1.Flex"
}

variable "instance_ocpus" {
  default = 1
}

variable "instance_shape_config_memory_in_gbs" {
  default = 6
}
```
On vérifie que tout est ok avec un :

```
terraform plan
data.oci_identity_availability_domains.ads: Reading...
data.oci_identity_availability_domains.ads: Read complete after 0s [id=IdentityAvailabilityDomainsDataSource-367745787]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_default_route_table.default_route_table will be created
  + resource "oci_core_default_route_table" "default_route_table" {
      + compartment_id             = (known after apply)
      + defined_tags               = (known after apply)
      + display_name               = "rt-srvweb"
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + manage_default_resource_id = (known after apply)
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

  # oci_core_instance.compute_instance will be created
  + resource "oci_core_instance" "compute_instance" {
      + availability_domain                 = "Tgrg:EU-PARIS-1-AD-1"
      + boot_volume_id                      = (known after apply)
      + capacity_reservation_id             = (known after apply)
      + compartment_id                      = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + dedicated_vm_host_id                = (known after apply)
      + defined_tags                        = (known after apply)
      + display_name                        = "srvweb"
      + fault_domain                        = "FAULT-DOMAIN-1"
      + freeform_tags                       = (known after apply)
      + hostname_label                      = (known after apply)
      + id                                  = (known after apply)
      + image                               = (known after apply)
      + ipxe_script                         = (known after apply)
      + is_pv_encryption_in_transit_enabled = (known after apply)
      + launch_mode                         = (known after apply)
      + metadata                            = {
          + "ssh_authorized_keys" = <<-EOT
                ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCXdARPOIcfkmBwhhCDV+YXiG7P97sUWK30ceYCgFobM5A1G/GWJIeEINj57ylfRiWvLzwsVIDC6zd9XW6HHhj4etgOFyi4mxSNHlSWtoj/w6Gj877R7+psuwtVprpQcOFzkbdzn/xnMRovdhyuHx7+BSMQXEnGlwuFECv/REn4VzLbyEOnj5u15U35Xkv1VJoUDX2Eq9OhM4Lo4+o/Nnia49IrwRMfOtSgwILzx/q+DkucM498Wx7TGacNMy2NDD7IbX9zp4nJfr29EkGKq018NQ2gVlt7Tkv1OPdRexH0eJNJUm2MlLLC0a9OM84+2I4U0JrBsd0xSso+ofDNeZTg/aKXia0o3qahwkUEbOlxHfg6namhNm9SqrNQS3qa6CHluQKoSl+ykOYZv+OYPmu32tcKL2yTf0KSFDgyBJb05Cox3YYpOZrq+H6HW6o2OQjGFzXPjxWXojCmGBSyXNquAmhpzFiJ7jlYhOPE9UItx57Vxpr1etxSx0392OodzWM= richard@mint
            EOT
        }
      + private_ip                          = (known after apply)
      + public_ip                           = (known after apply)
      + region                              = (known after apply)
      + shape                               = "VM.Standard.A1.Flex"
      + state                               = (known after apply)
      + subnet_id                           = (known after apply)
      + system_tags                         = (known after apply)
      + time_created                        = (known after apply)
      + time_maintenance_reboot_due         = (known after apply)

      + agent_config {
          + are_all_plugins_disabled = (known after apply)
          + is_management_disabled   = (known after apply)
          + is_monitoring_disabled   = (known after apply)

          + plugins_config {
              + desired_state = (known after apply)
              + name          = (known after apply)
            }
        }

      + availability_config {
          + is_live_migration_preferred = (known after apply)
          + recovery_action             = (known after apply)
        }

      + create_vnic_details {
          + assign_private_dns_record = true
          + assign_public_ip          = "true"
          + defined_tags              = (known after apply)
          + display_name              = "srvweb"
          + freeform_tags             = (known after apply)
          + hostname_label            = (known after apply)
          + private_ip                = (known after apply)
          + skip_source_dest_check    = (known after apply)
          + subnet_id                 = (known after apply)
          + vlan_id                   = (known after apply)
        }

      + instance_options {
          + are_legacy_imds_endpoints_disabled = (known after apply)
        }

      + launch_options {
          + boot_volume_type                    = (known after apply)
          + firmware                            = (known after apply)
          + is_consistent_volume_naming_enabled = (known after apply)
          + is_pv_encryption_in_transit_enabled = (known after apply)
          + network_type                        = (known after apply)
          + remote_data_volume_type             = (known after apply)
        }

      + platform_config {
          + are_virtual_instructions_enabled               = (known after apply)
          + is_access_control_service_enabled              = (known after apply)
          + is_input_output_memory_management_unit_enabled = (known after apply)
          + is_measured_boot_enabled                       = (known after apply)
          + is_secure_boot_enabled                         = (known after apply)
          + is_symmetric_multi_threading_enabled           = (known after apply)
          + is_trusted_platform_module_enabled             = (known after apply)
          + numa_nodes_per_socket                          = (known after apply)
          + percentage_of_cores_enabled                    = (known after apply)
          + type                                           = (known after apply)
        }

      + preemptible_instance_config {
          + preemption_action {
              + preserve_boot_volume = (known after apply)
              + type                 = (known after apply)
            }
        }

      + shape_config {
          + baseline_ocpu_utilization     = (known after apply)
          + gpu_description               = (known after apply)
          + gpus                          = (known after apply)
          + local_disk_description        = (known after apply)
          + local_disks                   = (known after apply)
          + local_disks_total_size_in_gbs = (known after apply)
          + max_vnic_attachments          = (known after apply)
          + memory_in_gbs                 = 6
          + networking_bandwidth_in_gbps  = (known after apply)
          + nvmes                         = (known after apply)
          + ocpus                         = 1
          + processor_description         = (known after apply)
        }

      + source_details {
          + boot_volume_size_in_gbs = "50"
          + boot_volume_vpus_per_gb = (known after apply)
          + kms_key_id              = (known after apply)
          + source_id               = "ocid1.image.oc1.eu-paris-1.aaaaaaaa2kukypyttuyb6vkpjbdrzl5dm2cg7mxniigjdukbfvelwlesrurq"
          + source_type             = "image"
        }

      + timeouts {
          + create = "60m"
        }
    }

  # oci_core_internet_gateway.internet_gateway will be created
  + resource "oci_core_internet_gateway" "internet_gateway" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "ig-srvweb"
      + enabled        = true
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)
    }

  # oci_core_network_security_group.nsg will be created
  + resource "oci_core_network_security_group" "nsg" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "nsg-srvweb"
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)
    }

  # oci_core_network_security_group_security_rule.nsg_outbound will be created
  + resource "oci_core_network_security_group_security_rule" "nsg_outbound" {
      + description               = "nsg-srvweb-outbound"
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

  # oci_core_security_list.sl will be created
  + resource "oci_core_security_list" "sl" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "seclist-srvweb"
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)

      + egress_security_rules {
          + description      = (known after apply)
          + destination      = "0.0.0.0/0"
          + destination_type = (known after apply)
          + protocol         = "6"
          + stateless        = (known after apply)
        }

      + ingress_security_rules {
          + description = (known after apply)
          + protocol    = "6"
          + source      = "0.0.0.0/0"
          + source_type = (known after apply)
          + stateless   = false

          + tcp_options {
              + max = 22
              + min = 22
            }
        }
      + ingress_security_rules {
          + description = (known after apply)
          + protocol    = "6"
          + source      = "0.0.0.0/0"
          + source_type = (known after apply)
          + stateless   = false

          + tcp_options {
              + max = 80
              + min = 80
            }
        }
    }

  # oci_core_subnet.subnet will be created
  + resource "oci_core_subnet" "subnet" {
      + availability_domain        = (known after apply)
      + cidr_block                 = "10.1.0.0/24"
      + compartment_id             = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags               = (known after apply)
      + dhcp_options_id            = (known after apply)
      + display_name               = "subnet-srvweb"
      + dns_label                  = "subnetsrvweb"
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + ipv6cidr_block             = (known after apply)
      + ipv6cidr_blocks            = (known after apply)
      + ipv6virtual_router_ip      = (known after apply)
      + prohibit_internet_ingress  = (known after apply)
      + prohibit_public_ip_on_vnic = (known after apply)
      + route_table_id             = (known after apply)
      + security_list_ids          = (known after apply)
      + state                      = (known after apply)
      + subnet_domain_name         = (known after apply)
      + time_created               = (known after apply)
      + vcn_id                     = (known after apply)
      + virtual_router_ip          = (known after apply)
      + virtual_router_mac         = (known after apply)
    }

  # oci_core_vcn.vcn will be created
  + resource "oci_core_vcn" "vcn" {
      + byoipv6cidr_blocks               = (known after apply)
      + cidr_block                       = "10.1.0.0/16"
      + cidr_blocks                      = (known after apply)
      + compartment_id                   = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + default_dhcp_options_id          = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_list_id         = (known after apply)
      + defined_tags                     = (known after apply)
      + display_name                     = "vcn-srvweb"
      + dns_label                        = "vcnsrvweb"
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

Plan: 8 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take
exactly these actions if you run "terraform apply" now.
```
Tout va bien, on fait un :

```
$ terraform apply
data.oci_identity_availability_domains.ads: Reading...
data.oci_identity_availability_domains.ads: Read complete after 0s [id=IdentityAvailabilityDomainsDataSource-367745787]

Terraform used the selected providers to generate the following execution plan. Resource
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_default_route_table.default_route_table will be created
  + resource "oci_core_default_route_table" "default_route_table" {
      + compartment_id             = (known after apply)
      + defined_tags               = (known after apply)
      + display_name               = "rt-srvweb"
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + manage_default_resource_id = (known after apply)
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

  # oci_core_instance.compute_instance will be created
  + resource "oci_core_instance" "compute_instance" {
      + availability_domain                 = "Tgrg:EU-PARIS-1-AD-1"
      + boot_volume_id                      = (known after apply)
      + capacity_reservation_id             = (known after apply)
      + compartment_id                      = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + dedicated_vm_host_id                = (known after apply)
      + defined_tags                        = (known after apply)
      + display_name                        = "srvweb"
      + fault_domain                        = "FAULT-DOMAIN-1"
      + freeform_tags                       = (known after apply)
      + hostname_label                      = (known after apply)
      + id                                  = (known after apply)
      + image                               = (known after apply)
      + ipxe_script                         = (known after apply)
      + is_pv_encryption_in_transit_enabled = (known after apply)
      + launch_mode                         = (known after apply)
      + metadata                            = {
          + "ssh_authorized_keys" = <<-EOT
                ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCXdARPOIcfkmBwhhCDV+YXiG7P97sUWK30ceYCgFobM5A1G/GWJIeEINj57ylfRiWvLzwsVIDC6zd9XW6HHhj4etgOFyi4mxSNHlSWtoj/w6Gj877R7+psuwtVprpQcOFzkbdzn/xnMRovdhyuHx7+BSMQXEnGlwuFECv/REn4VzLbyEOnj5u15U35Xkv1VJoUDX2Eq9OhM4Lo4+o/Nnia49IrwRMfOtSgwILzx/q+DkucM498Wx7TGacNMy2NDD7IbX9zp4nJfr29EkGKq018NQ2gVlt7Tkv1OPdRexH0eJNJUm2MlLLC0a9OM84+2I4U0JrBsd0xSso+ofDNeZTg/aKXia0o3qahwkUEbOlxHfg6namhNm9SqrNQS3qa6CHluQKoSl+ykOYZv+OYPmu32tcKL2yTf0KSFDgyBJb05Cox3YYpOZrq+H6HW6o2OQjGFzXPjxWXojCmGBSyXNquAmhpzFiJ7jlYhOPE9UItx57Vxpr1etxSx0392OodzWM= richard@mint
            EOT
        }
      + private_ip                          = (known after apply)
      + public_ip                           = (known after apply)
      + region                              = (known after apply)
      + shape                               = "VM.Standard.A1.Flex"
      + state                               = (known after apply)
      + subnet_id                           = (known after apply)
      + system_tags                         = (known after apply)
      + time_created                        = (known after apply)
      + time_maintenance_reboot_due         = (known after apply)

      + agent_config {
          + are_all_plugins_disabled = (known after apply)
          + is_management_disabled   = (known after apply)
          + is_monitoring_disabled   = (known after apply)

          + plugins_config {
              + desired_state = (known after apply)
              + name          = (known after apply)
            }
        }

      + availability_config {
          + is_live_migration_preferred = (known after apply)
          + recovery_action             = (known after apply)
        }

      + create_vnic_details {
          + assign_private_dns_record = true
          + assign_public_ip          = "true"
          + defined_tags              = (known after apply)
          + display_name              = "srvweb"
          + freeform_tags             = (known after apply)
          + hostname_label            = (known after apply)
          + private_ip                = (known after apply)
          + skip_source_dest_check    = (known after apply)
          + subnet_id                 = (known after apply)
          + vlan_id                   = (known after apply)
        }

      + instance_options {
          + are_legacy_imds_endpoints_disabled = (known after apply)
        }

      + launch_options {
          + boot_volume_type                    = (known after apply)
          + firmware                            = (known after apply)
          + is_consistent_volume_naming_enabled = (known after apply)
          + is_pv_encryption_in_transit_enabled = (known after apply)
          + network_type                        = (known after apply)
          + remote_data_volume_type             = (known after apply)
        }

      + platform_config {
          + are_virtual_instructions_enabled               = (known after apply)
          + is_access_control_service_enabled              = (known after apply)
          + is_input_output_memory_management_unit_enabled = (known after apply)
          + is_measured_boot_enabled                       = (known after apply)
          + is_secure_boot_enabled                         = (known after apply)
          + is_symmetric_multi_threading_enabled           = (known after apply)
          + is_trusted_platform_module_enabled             = (known after apply)
          + numa_nodes_per_socket                          = (known after apply)
          + percentage_of_cores_enabled                    = (known after apply)
          + type                                           = (known after apply)
        }

      + preemptible_instance_config {
          + preemption_action {
              + preserve_boot_volume = (known after apply)
              + type                 = (known after apply)
            }
        }

      + shape_config {
          + baseline_ocpu_utilization     = (known after apply)
          + gpu_description               = (known after apply)
          + gpus                          = (known after apply)
          + local_disk_description        = (known after apply)
          + local_disks                   = (known after apply)
          + local_disks_total_size_in_gbs = (known after apply)
          + max_vnic_attachments          = (known after apply)
          + memory_in_gbs                 = 6
          + networking_bandwidth_in_gbps  = (known after apply)
          + nvmes                         = (known after apply)
          + ocpus                         = 1
          + processor_description         = (known after apply)
        }

      + source_details {
          + boot_volume_size_in_gbs = "50"
          + boot_volume_vpus_per_gb = (known after apply)
          + kms_key_id              = (known after apply)
          + source_id               = "ocid1.image.oc1.eu-paris-1.aaaaaaaa2kukypyttuyb6vkpjbdrzl5dm2cg7mxniigjdukbfvelwlesrurq"
          + source_type             = "image"
        }

      + timeouts {
          + create = "60m"
        }
    }

  # oci_core_internet_gateway.internet_gateway will be created
  + resource "oci_core_internet_gateway" "internet_gateway" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "ig-srvweb"
      + enabled        = true
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)
    }

  # oci_core_network_security_group.nsg will be created
  + resource "oci_core_network_security_group" "nsg" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "nsg-srvweb"
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)
    }

  # oci_core_network_security_group_security_rule.nsg_outbound will be created
  + resource "oci_core_network_security_group_security_rule" "nsg_outbound" {
      + description               = "nsg-srvweb-outbound"
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

  # oci_core_security_list.sl will be created
  + resource "oci_core_security_list" "sl" {
      + compartment_id = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags   = (known after apply)
      + display_name   = "seclist-srvweb"
      + freeform_tags  = (known after apply)
      + id             = (known after apply)
      + state          = (known after apply)
      + time_created   = (known after apply)
      + vcn_id         = (known after apply)

      + egress_security_rules {
          + description      = (known after apply)
          + destination      = "0.0.0.0/0"
          + destination_type = (known after apply)
          + protocol         = "6"
          + stateless        = (known after apply)
        }

      + ingress_security_rules {
          + description = (known after apply)
          + protocol    = "6"
          + source      = "0.0.0.0/0"
          + source_type = (known after apply)
          + stateless   = false

          + tcp_options {
              + max = 22
              + min = 22
            }
        }
      + ingress_security_rules {
          + description = (known after apply)
          + protocol    = "6"
          + source      = "0.0.0.0/0"
          + source_type = (known after apply)
          + stateless   = false

          + tcp_options {
              + max = 80
              + min = 80
            }
        }
    }

  # oci_core_subnet.subnet will be created
  + resource "oci_core_subnet" "subnet" {
      + availability_domain        = (known after apply)
      + cidr_block                 = "10.1.0.0/24"
      + compartment_id             = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + defined_tags               = (known after apply)
      + dhcp_options_id            = (known after apply)
      + display_name               = "subnet-srvweb"
      + dns_label                  = "subnetsrvweb"
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + ipv6cidr_block             = (known after apply)
      + ipv6cidr_blocks            = (known after apply)
      + ipv6virtual_router_ip      = (known after apply)
      + prohibit_internet_ingress  = (known after apply)
      + prohibit_public_ip_on_vnic = (known after apply)
      + route_table_id             = (known after apply)
      + security_list_ids          = (known after apply)
      + state                      = (known after apply)
      + subnet_domain_name         = (known after apply)
      + time_created               = (known after apply)
      + vcn_id                     = (known after apply)
      + virtual_router_ip          = (known after apply)
      + virtual_router_mac         = (known after apply)
    }

  # oci_core_vcn.vcn will be created
  + resource "oci_core_vcn" "vcn" {
      + byoipv6cidr_blocks               = (known after apply)
      + cidr_block                       = "10.1.0.0/16"
      + cidr_blocks                      = (known after apply)
      + compartment_id                   = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
      + default_dhcp_options_id          = (known after apply)
      + default_route_table_id           = (known after apply)
      + default_security_list_id         = (known after apply)
      + defined_tags                     = (known after apply)
      + display_name                     = "vcn-srvweb"
      + dns_label                        = "vcnsrvweb"
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

Plan: 8 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

oci_core_vcn.vcn: Creating...
oci_core_vcn.vcn: Creation complete after 1s [id=ocid1.vcn.oc1.eu-paris-1.amaaaaaa5nytwhqa3gysivvqkywxkkbjw6bogmhxtzfiifooygdayidwhtyq]
oci_core_internet_gateway.internet_gateway: Creating...
oci_core_network_security_group.nsg: Creating...
oci_core_security_list.sl: Creating...
oci_core_security_list.sl: Creation complete after 0s [id=ocid1.securitylist.oc1.eu-paris-1.aaaaaaaarwlkmlo7hsbrlgtazpfsfej6fdltqhyacxnets7yuctd77vkbrfa]
oci_core_subnet.subnet: Creating...
oci_core_network_security_group.nsg: Creation complete after 0s [id=ocid1.networksecuritygroup.oc1.eu-paris-1.aaaaaaaaxblbyvb62zswggqwdwputgduuhli524jex5qr3yiu5ho3wxvucbq]
oci_core_network_security_group_security_rule.nsg_outbound: Creating...
oci_core_network_security_group_security_rule.nsg_outbound: Creation complete after 0s [id=26BA23]
oci_core_internet_gateway.internet_gateway: Creation complete after 0s [id=ocid1.internetgateway.oc1.eu-paris-1.aaaaaaaazok5ftokl2ouyqcukcc4nnzz7h7lyd3y3cyxnctwf6ajwaxld4ba]
oci_core_default_route_table.default_route_table: Creating...
oci_core_default_route_table.default_route_table: Creation complete after 1s [id=ocid1.routetable.oc1.eu-paris-1.aaaaaaaaotzsmay6gvdjp7jler7d3tcys65adkoaeo2cf2c3nv6mer67zavq]
oci_core_subnet.subnet: Provisioning with 'local-exec'...
oci_core_subnet.subnet (local-exec): Executing: ["/bin/sh" "-c" "sleep 5"]
oci_core_subnet.subnet: Creation complete after 9s [id=ocid1.subnet.oc1.eu-paris-1.aaaaaaaaverpsr73wqcjpqpisoxjxl76epuo22uvcumdnh3hkqdbc4o7xyla]
oci_core_instance.compute_instance: Creating...
oci_core_instance.compute_instance: Still creating... [10s elapsed]
oci_core_instance.compute_instance: Still creating... [20s elapsed]
oci_core_instance.compute_instance: Still creating... [30s elapsed]
oci_core_instance.compute_instance: Still creating... [40s elapsed]
oci_core_instance.compute_instance: Still creating... [50s elapsed]
oci_core_instance.compute_instance: Still creating... [1m0s elapsed]
oci_core_instance.compute_instance: Still creating... [1m10s elapsed]
oci_core_instance.compute_instance: Still creating... [1m20s elapsed]
oci_core_instance.compute_instance: Still creating... [1m30s elapsed]
oci_core_instance.compute_instance: Still creating... [1m40s elapsed]
oci_core_instance.compute_instance: Still creating... [1m50s elapsed]
oci_core_instance.compute_instance: Still creating... [2m0s elapsed]
oci_core_instance.compute_instance: Still creating... [2m10s elapsed]
oci_core_instance.compute_instance: Still creating... [2m20s elapsed]
oci_core_instance.compute_instance: Still creating... [2m30s elapsed]
oci_core_instance.compute_instance: Still creating... [2m40s elapsed]
oci_core_instance.compute_instance: Creation complete after 2m46s [id=ocid1.instance.oc1.eu-paris-1.anrwiljr5nytwhqc7vd7glgxkin4wtmh32igvea2jo7soaejfbb2otnz24va]

Apply complete! Resources: 8 added, 0 changed, 0 destroyed.
```

Si on se connecte à l'interface d'OCI, on voit bien notre compute instance:

![oci](/img/oci29.png)

On peut se connecter en ssh à notre machine

![oci](/img/oci30.png)

Avant de finaliser l'automatisation, nous allons installer manuellement le serveur apache et ouvrir le port 80 sur notre instance

```bash
sudo apt-get update
sudo apt-get install apache2
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo netfilter-persistent save
```

Et on s'assure que c'est bon en tapant notre ip publique dans un navigateur:

![oci](/img/oci31.png)










