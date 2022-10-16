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
      source = "hashicorp/oci"
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

```bash
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/oci...
- Installing hashicorp/oci v4.96.0...
- Installed hashicorp/oci v4.96.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

╷
│ Warning: Additional provider information from registry
│ 
│ The remote registry returned warnings for registry.terraform.io/hashicorp/oci:
│ - For users on Terraform 0.13 or greater, this provider has moved to oracle/oci. Please update your source in required_providers.
╵

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

```bash
terraform validate
Success! The configuration is valid.
```

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
      + compartment_id                   = "ocid1.tenancy.oc1..aaaaaaaaujdjzgc3moz6nw2o6d74a6xxlft7yjsjav47jx4l4ic2fc2b4qya"
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
oci_core_vcn.internal: Creation complete after 1s [id=ocid1.vcn.oc1.eu-paris-1.amaaaaaa5nytwhqaayum25cq3ohfal22i47hggrxzvpemqfaq5sb66fmzpvq]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
![Oracle](/img/oraclenetwork.png)