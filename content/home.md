---
title: "contrôle parental, virtualisation et logiciel libre"
date: 2018-01-28T13:06:33Z
draft: true
---


# contrôle parental, virtualisation et logiciel libre

Le monde de l’éducation utilise depuis longtemps le logiciel libre pour assurer le filtrage de l’internet dans les établissements scolaires.

Nous allons mettre en oeuvre une variante de cette solution pour votre réseau local.

La mise en oeuvre du contrôle parental peut se faire de plusieurs façons. La méthode la plus simple consiste à installer un logiciel dédié, logprotect par exemple. 
Cette technique peut être contournée très facilement. De plus, les foyers ont maintenant plus d’un ordinateur sur leur réseau ce qui multiplie les manipulations !

Nous allons mettre en oeuvre une technique dérivée de celle utilisée par les projets développés pour l’éducation : SLIS, Abuledu.

Nous allons utiliser serveur Linux dans lequel nous allons installer les logiciels squid et dansguardian. Dans un deuxième temps, nous implémenterons les listes noires gérées par l’académie de toulouse : **ftp://ftp.univ-tlse1.fr/pub/reseau/cache/squidguard_contrib/.**

La mise en oeuvre de ce serveur peut se faire par le biais d’une vieille machine 64 Mo de RAM, 700 Mo de disque dur ou plus simplement en utilisant une machine virtuelle déjà prête.

Une fois l’hyperviseur installé, dans notre cas Vmware Player, vous ouvrez le fichier .vmx... votre serveur démarre et directement opérationnel. 
Vous configurez les navigateurs internet des ordinateurs du réseau pour utiliser le proxy : **192.168.1.200:8080**. Le filtrage sera ainsi opérationnel et totalement transparent.

**Le serveur**
Ubuntu server 7.10
**Le service proxy**
Pour installer Squid tapez dans un terminal :

    sudo aptitude install squid

**Configurer le proxy**

La configuration de Squid se fait en éditant le fichier **/etc/squid/squid.conf**

    sudo nano /etc/squid/squid.conf

**Nommer le proxy**

Squid a besoin de connaître le nom de la machine. Pour cela, repérez la ligne visible_hostname. Par exemple, si la machine s’appelle ’ubuntu’, mettez :

    visible_hostname ubuntu

**Choisir le port**

Par défaut, le serveur proxy sera en écoute sur le port 3128. Pour choisir un autre port, repérez la ligne :

    http_port 3128

**Choisir l’interface**

Par défaut le serveur proxy sera en écoute sur toutes les interfaces. Pour des raisons de sécurité, il ne faut le mettre en écoute que sur votre réseau local. Par exemple si la carte réseau reliée à votre LAN a l’IP 10.0.0.1, modifiez la ligne :

    http_port 192.168.1.200:3128

**Définir les droits d’accès**

Par défaut, personne n’est autorisé à se connecter au serveur proxy, sauf votre machine elle-même. Il faut créer une liste d’autorisations. Par exemple, on va définir un groupe englobant tout le réseau local.

Repérez la ligne du fichier commençant par acl localhost... A la fin de la section, ajoutez :

    acl monreseau src 10.0.0.0/255.255.255.0

(**monreseau** est un nom arbitraire que nous avons choisi).

**Autoriser le groupe**

Maintenant que le groupe est défini, nous allons l’authoriser à utiliser le proxy. Repérez la ligne http_access allow... et ajoutez en dessous (avant la ligne **http_access deny all**)

    http_access allow monreseau

**Le service de filtrage**
Installation de DansGuardian

    sudo aptitude install dansguardian

L’installation de « DansGuardian entraîne l’installation de l’antivirus « Clamav ». Dans ce document, nous n’activerons pas cet antivirus car il n’est pas indispensable sous Linux et consomme pas mal de ressources.

Configuration de DansGuardian

DansGuardian utilise plusieurs fichiers de configuration enregistrés dans « /etc/dansguardian ».

Configuration de « **/etc/dansguardian/dansguardian.conf** »

    sudo nano /etc/dansguardian/dansguardian.conf

Pour activer la configuration, il faut commenter ou supprimer cette ligne en début de fichier :

**UNCONFIGURED**

Cette ligne permet d’indiquer le port utilisé par le proxy :

**proxyport = 8080**

Cette ligne permet d’avoir les messages en français en cas de blocage des sites :

**language = 'french'**

Cette ligne permet de désactiver l’antivirus :

**virusscan = off**

Configuration de « **/etc/dansguardian/dansguardianf1.conf** »

    sudo nano /etc/dansguardian/dansguardianf1.conf

La ligne suivante permet de paramétrer le filtrage en fonction de l’âge des personnes concernées :

**naughtynesslimit = 80**

La valeur par défaut de « 50 » donne un filtrage très dur et peu de sites sont accessibles. Plus cette valeur est élevée et plus le filtrage est faible. Personnellement, j’ai fait quelques tests et j’ai passé cette valeur à « 80 ».

Cette ligne permet de désactiver l’antivirus :

**virusscan = off**
