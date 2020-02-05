Pour les néophytes en [Markdown](https://www.markdownguide.org/basic-syntax/)

Un outil de prévisualisation utile [StackEdit](https://stackedit.io/) 

# Descriptions des demandes :
*Contexte client à ne pas prendre en compte, nous sommes ici pour prendre du plaisir/s'entrainer sur des technos qui pourront
nous être utile*

## Entrée technique à prendre en compte 
_Infra en 2 partie LAN/WAN :_
* __Infra LAN :__
  * DNS : serveur ayant pour but de faire les résolution DNS sur le réseau Local entreprise
  * Web : Demande WordPress préconisation Léo : Wekan
  * BDD : à mettre en place pour gérer le service WEB
  * Serveur annuaire : permettra l'authentification sur le serveur
  * Serveur pour les sauvegardes
  * Reverse proxy pour accéder au service Web
  * Aucun serveur du réseau local n'a directement accès a internet, les dépots seront stockés sur un serveur qui gérera le cache ( à configurer dans les sources.list des différents serveurs)
  * Outil de provisionnement
* __Infra WAN :__
  * Passerelle de sortie vers internet
  * Serveur d'externalisation de sauvegarde (facultatif)
* __Supervision :__
  * Solution capable de superviser la partie LAN/WAN (pas des terminaux)
  * Métriques
  * Alertes
  * Automatisation d'actions mineures (facultatif) 
* __Sécurisation :__
  * _Mettre en place une solution permettant la sécurisation des éléments suivants :_
  * LDAP
  * DNS
  * HTTP
  * BDD

### Techno à mettre en place pour le projet Linux

Ici je détail les différentes techno que nous allons étudier/mettre en place pour ce labo
* __Minimum attendu :__
  * OS serveur : [__CentOS 7 1810__](http://repos-va.psychz.net/centos/7.6.1810/isos/x86_64/)
  * OS Client : [__Ubuntu 18.04.3 LTS__](https://ubuntu.com/download/desktop)
  * DNS  : [__Zentyal__](https://wiki.zentyal.org/wiki/Installation_Guide)
  * LDAP : [__Zentyal__](https://wiki.zentyal.org/wiki/Installation_Guide)
  * DHCP : [__Zentyal__](https://wiki.zentyal.org/wiki/Installation_Guide)
  * BDD : [__MariaDB__](https://linuxize.com/post/install-mariadb-on-centos-7/)
  * WEB : [__Nginx__](https://www.cyberciti.biz/faq/how-to-install-and-use-nginx-on-centos-7-rhel-7/)
  * NFS : [__NFS Utils__](https://blog.microlinux.fr/serveur-nfs-centos/)
  * SSH (au travers d'un proxy) : [__Proxy Command__](https://www.cyberciti.biz/faq/linux-unix-ssh-proxycommand-passing-through-one-host-gateway-server/)
  * Sauvegarde : [__Borgbackup__](https://borgbackup.readthedocs.io/en/stable/installation.html)
  * Gestion de configuration : [__Puppet__](https://puppet.com/try-puppet/puppet-enterprise/?ls=Campaigns&lsd=Paid-Search&cid=7010f00000232RV&utm_medium=paid-search&utm_campaign=Q3FY20_EMEA_SEMEA_DEMAND_SER_ADWRDS_pe-dwnld&utm_source=google&utm_content=pe-10-nodes&obility_id=78002805358&gclid=EAIaIQobChMIyMiM3Law5wIVw_ZRCh2MIA8FEAAYASAAEgLny_D_BwE)

* __Services optionnels :__
  * FAil2ban
  * Reverse Proxy
  * OSSEC
  * Supervision/métrologie
  * Ansible
  * SpaceWalk
  * Docker
  
## Carac des VMs

Chaque VM contient 1 coeur, 1go de ram et 80go de DD

## Diverses convention de nommage

Nom de domaine : inline.corp

Différentes IP : 192.168.x.x

Nommage des machines : 

Pour la convention de nommage de nos machines/VMs nous utiliserons la convention suivante : 

Type de machine-Nom de domaine-Service ou numéro incrémentielle

_Explications_ :

Le type de machine : 2 types de machine sont dispo : __SRV__ pour les serveurs ou __CLI__ pour les utilisateurs finaux

Nom de domaine : Le nom de domaine pour ce TP est Inline, nous utilisons donc le trigramme suivant : __INL__

Service ou numéro incrémentielle : 

* Ce trigramme sera utilisé de 2 façons différentes :
  * Dans le cas des serveurs, ceux-ci sont dédiés à un type de service, le trigramme correspondra au service présent sur chaque serveur
  * Dans le cas des machines clients, celle-ci sont génériques et ne seront donc différenciées que par un numéro incrémentiel qui commence a 001 et finit à 999
  
## Infra Finale
  
  Voici le visio de l'infrastructure final que nous allons utiliser 
  ![alt tag](https://user-images.githubusercontent.com/58468543/73590526-b7caaf00-44e3-11ea-8d55-fa1d940cfb94.png)
  
## Explications technologie 

_Rebond SSH_

Le rebond SSH est là pour protéger les accès au serveur de back. Le rebond se fait par une machine dans une zone neutre qui peut communiquer sur les 2 plages IP avec 2 cartes réseau différentes. Cette machine sert de relais et de gestionnaire d'accès sans laisser filtrer les informations sensibles d'un côté comme de l'autre.
![alt tag](https://user-images.githubusercontent.com/58468543/73590935-d0899380-44e8-11ea-92b9-28855a27d0cd.png)

## BDD

_Présentation_

L'outil MariaDB est là pour nous permettre de gérer la base de données qui vas nous servir à stocker les sauvegarde faites avec Borgbackup.

_Installation_

Nous faisons l'installation du Paquet MariaDB ainsi que la sécurisation de mysql 

	sudo yum install mariadb-server
	sudo mysql_secure_installation

Ensuite nous ajoutons un le depo MariaDB pour pouvoir installer le server et le client MariaDB. Pour ce faire nous créons le fichier MariaDB.repo dans yum.repos.

Ensuite nous installons le Serveurs et le client MariaDB avec 

	sudo yum install MariaDB-server MariaDB-client
 
 Derniere étape de l'installation, nous lançons le script de sécurisation sql

	sudo mysql_secure_installation
	
 A chaque étape démarré, authorisé et vérifier le status de MariaDB.

	sudo systemctl enable mariadb
	sudo systemctl start mariadb
	sudo systemctl status mariadb
	
_Login sur MariaDB_

Pour se connecter à MariaDB nous utilisons la commande de base qui nous permet de lancer MariaDB en root avec le mot de passe précisé plus tôt avec le script de sécurisation MySQL  :

	mysql -u root -p
	
_Diverses utilisation de la table SQL_

Pour créer une base de donnée SQL : 

	CREATE DATABASE WebServiceSave;
	
Et ensuite pour vérifier que la base est bien présente : 
	
	SHOW DATABASES;
	
et vous aurez une réponse du style : 

	+--------------------+
	| Database           |
	+--------------------+
	| WebServiceSave     |
	| information_schema |
	| mysql              |
	| performance_schema |
	+--------------------+

Les bases de donnée information_schéma, mysql et performance_schema sont des bases installée par SQL lors de l'installation et elles stockent des informations à propos de toutes les bases de données, la configuration systeme, les utilisateurs, les authorisations ainsi que d'autres importantes données.

Pour supprimer une base de donnée SQL :

	DROP DATABASE WebServiceSave;
	
_Gestion des utilisateurs SQL_

Nous allons créer l'utilisateur AdminBorg capable de se connecter sur localhost avec le mot de passe à définir ensuite et remplacer 

	CREATE USER 'AdminBorg'@'localhost' IDENTIFIED BY 'user_password';
	
Pour changer le mot de passe d'un utilisateur MySQL nous utilisons cette commande :

	

Pour lister les utilisateurs présent sur le MySQL nous utilisons ceci : 

	SELECT user, host FROM mysql.user;
	
Nous pouvons avoir le résultat suivant 

	+-----------+-----------------------+
	| user      | host                  |
	+-----------+-----------------------+
	| root      | 127.0.0.1             |
	| root      | ::1                   |
	| AdminBorg | localhost             |
	| root      | localhost             |
	| root      | localhost.localdomain |
	+-----------+-----------------------+
	5 rows in set (0.000 sec)


## Zentyal

_Présentation_

Zentyal est une plateforme clefs en main pour PME et TPE, qui permet la mise en place de différents services et rôles : Contrôleur de domaine, DNS, systèmes de partage de fichiers, pare-feu … 
Ici nous utiliserons la plateforme en tant que serveur DNS, serveur DHCP pour les clients, contrôleur de domaine. Nous prendrons la version 6.1 de Zentyal, qui tourne sous ubuntu 18.04.

_Installation_

![alt tag](https://user-images.githubusercontent.com/58468543/73642897-d7dca880-4672-11ea-8118-38c315dfee4c.png)

1ere sélection des services après l’installation de la plateforme. Ici nous nous servirons de Zentyal en tant que serveur DNS, LDAP, Contrôleur de domaine et serveur DHCP. Il y aura toujours la possibilité d’installer d’autres services par la suite car la plateforme propose tout un panel de services.

*Configuration matérielle de la machine :
  * -	20 GB stockage
  * -	1024 Mo RAM
  * -	1 carte réseau NAT pour accès à internet au démarrage. 
  * -	1 carte réseau pour joindre les clients, via laquelle le protocole DHCP transitera pour allouer dynamiquement des adresses IP aux clients.
  * -	1 carte réseau pour joindre les serveurs de l’infrastructure. 
  
![alt tag](https://user-images.githubusercontent.com/58468543/73642909-dc08c600-4672-11ea-8498-16022cd49125.png)

![alt tag](https://user-images.githubusercontent.com/58468543/73642918-df03b680-4672-11ea-9f0a-3ffca14bbfb9.png)

![alt tag](https://user-images.githubusercontent.com/58468543/73642926-e32fd400-4672-11ea-8019-864b703ae1bc.png)

![alt tag](https://user-images.githubusercontent.com/58468543/73642934-e6c35b00-4672-11ea-98ad-1c9caaf41850.png)

Ici nous configurerons les noms d’hôtes de nos serveurs. 

Ne pas oublier de faire le paramétrage sur les serveurs et les clients dans /etc/hostname., ainsi que dans le fichier /etc/resolv.conf afin de déclarer le domaine inline.corp et le contrôleur de domaine inline-dc01 associé à l’adresse IP 192.168.2.1/

![alt tag](https://user-images.githubusercontent.com/58468543/73642944-e925b500-4672-11ea-87ca-2515dc61da97.png)

Ici on renseignera les noms d’hôtes des serveurs avec leur adresse IP. Nous pouvons également configurer des alias. Exemple avec inline-ssh1 en ssh1.

![alt_tag](https://user-images.githubusercontent.com/58468543/73642947-eb880f00-4672-11ea-8cd3-e8cff1cbb1b7.png)

_DHCP_

![alt tag](https://user-images.githubusercontent.com/58468543/73642986-fd69b200-4672-11ea-986b-0929b0897f07.png)

Configuration DHCP via les interfaces réseau du serveur. 1 Seule interface délivre le service DHCP, celle qui joint les clients.

![alt_tag](https://user-images.githubusercontent.com/58468543/73643000-00fd3900-4673-11ea-9555-ef9b9284b8f6.png)


Configuration d’une plage DHCP sur l’interface qui communiquera avec les clients.  Nous configurerons la plage de 192.168.2.50 à 192.168.2.253 (192.168.2.254 étant la passerelle).

![alt_tag](https://user-images.githubusercontent.com/58468543/73643009-035f9300-4673-11ea-8046-32d6ad918534.png)

Configuration des interfaces réseau. 

![alt_tag](https://user-images.githubusercontent.com/58468543/73642958-efb42c80-4672-11ea-9e76-e1007f9830c9.png)

![alt_tag](https://user-images.githubusercontent.com/58468543/73642963-f2af1d00-4672-11ea-89fb-f1e540b26b66.png)

Configuration de l'interface réseau client reliée au client. Cette interface délivrera les adresses IP de la plage configurée aux clients présents sur le même sous réseau que l'interface eth1.

![alt_tag](https://user-images.githubusercontent.com/58468543/73642963-f2af1d00-4672-11ea-89fb-f1e540b26b66.png)

## Serveur de rebond SSH

![alt_tag](https://user-images.githubusercontent.com/58468543/73643021-08bcdd80-4673-11ea-89ea-789565c4300a.png)

Le service SSH doit bien entendu être activé et exécuté  sur toutes les machines utilisées.
Ici nous utiliserons la méthode via ProxCommand et l’activation de ForwardAgent, pour rebondir sur le serveur inline-ssh2 afin d’accèder au serveur inline-ssh3. L’agent gèrera le rebond de l’authentification entre le serveur cible (inline-ssh3) et la machine cliente (ici inline-ssh1).
La variable %h nous permet d’indiquer le port (ici le port par défaut, 22). Le paramètre -W nous permet d’utiliser le netcat mode.	

## Ajout d'un client au Controleur de domaine Zentyal

Pré requis :

- Machine cliente sous CentOS 7
- Un contrôleur de domaine (DC) sous Zentyal
- Configuration de l’interface joignant le contrôleur de domaine
- Identifiants d’un compte administrateur du domaine
- Nom de domaine : inline.corp

Configuration du nom d’hôte de la machine cliente dans le fichier /etc/hostname.
	
Ici : 

	inl-cli03.inline.corp
	
Nous renseignons le nom de domaine et  son adresse IP dans le fichier /etc/resolv.conf :


	search inline.corp

	nameserver 192.168.2.1
	
Installation des paquets suivants : 

	sudo yum install samba samba-winbind 

Le paquet fournira les outils nécessaires pour intégrer le domaine Zentyal.

	sudo yum install krb5-workstation

Nous passons ensuite à la modification du fichier /etc/krb5.conf afin de déclarer notre domaine : 

![alt_tag](https://user-images.githubusercontent.com/51946423/73828583-d392c580-4801-11ea-8420-2149d3d45a6b.png)


Le paquet Kerberos Workstation client fournira une authentification chiffrée basée sur Key Distribution Center (KDC).

Pour lister les royaumes disponibles, 

Côté client, pour joindre la machine au domaine, utiliser la commande suivante: 

	realm join <nom du domaine> -U <AdministrateurDuDomaine>

Pour connecter un utilisateur du domaine, En mode terminal, entre la commande suivante : 

	 su – inline.corp\\<User>
	 


## Sauvegarde - BorgBackup

C'est un programme de sauvegarde qui fourni une méthode sécurisée pour savegarder des données. 

_Installation des dépendances et  pré requis_
Pour installer BorgBackup nous devons installer les dépendances : 

	Yum install python3 openssl-devel cython gcc gcc-c++ libzstd libb2 python3-devel libacl-devel

Nous installerons les paquets via les commandes suivantes : 
Nous téléchargeons et installons tout d'abord le package pip. C'est un utilitaire permettant notamment d'installer des packages Python.
	curl https://bootstrap.pypa.io/get-pip.py -o

	get-pip.py


Virtualenv est un outil pour créer des environnements virtuels Python isolés. virtualenv crée un dossier qui contient tous les exécutables nécessaires pour utiliser les paquets qu’un projet Python pourrait nécessiter. En effet, Borgbackup fonctionne avec Python.
Nous installerons Virtualenv via la commande suivante : 

Nous allons créer un environnement nommé « borg-env » et utilisant python 3.

	virtualenv --python=python3 borg-env


Nous pouvons activer ou désactiver cet environnement à notre guise avec les commandes :

	source borg-env/bin/activate

Ou

	source borg-env/bin/desactivate

Nous finalisons l'installation de BorgBackup : 

	Pip install borgbackup

_Effectuer une sauvegarde_

Sauvegarde locale

Nous allons tout d’abord créer deux dossiers : 1 dépôt et 1 pour effectuer les tests de restauration.

	mkdir -p /data/backup/borg_backup /data/backup/test`

Nous initialisons ensuite le dépôt. Nous allons le déclarer à Borg et chiffrer le dépôt.

	borg init --encryption=keyfile /data/backup/borg_backup

Il faut ensuite choisir une passphrase en plus du chiffrement. Il faudra donc la clé de déchiffrement plus la passphrase pour déchiffrer les sauvegardes.
	
_Gestion de la sauvegarde en ligne de commande :_

Pour effectuer une sauvegarde en ligne de commande : 

	borg create -v --stats /data/backup/borg_backup::{hostname}_{now:%d.%m.%Y}
	
Pour vérifier toutes les sauvegardes, nous entrons la commande : 
	
borg list  /data/backup/borg_backup

Pour consulter une sauvegarde : 

	 borg list  /data/backup/borg_backup::localhost_dd.mm.yyyy

Pour restaurer une sauvegarde : 

	 borg extract  /data/backup/borg_backup:: localhost_dd.mm.yyyy

_Paramétrage pour la sauvegarde distante_

Pour faire de la sauvegarde distante il faut que le client ait borgbackup ainsi qu’openssh-server d’installé. En effet le client initie une connexion ssh avec la machine hébergeant le dépôt distant. Borg sait nativement gérer les connexions ssh.

Créons un utilisateur pour borgbackup sur le serveur, et générons une clef SSH.

	useradd borg --create-home --home-dir /var/lib/borg-backups/

	ssh-keygen
Copiez la clef sur la machine cliente au serveur Borg.
Nous allons maintenant initier le dépôt distant sur le serveur : 

	borg init --encryption=keyfile **borg@borg:**/var/lib/borg-backups/

Les commandes sont trés légèrement différentes par rapport à la sauvegarde locale, du fait de la connexion avec une machine distante. Il faut rajouter le compte avec lequel nous réalisons la sauvegarde (borg) ainsi que le nom d’hôte ou l’ip de la machine (dans le cas actuel le hostname est « borg »).

Pour effectuer une sauvegarde à distance de notre serveur web nous NGINX, nous pouvons effectuer la commande suivante : 

	borg create -v –stats [borg@borg:/var/lib/borg-backups/::{hostname}_{now:%d.%m.%Y}](mailto:borg@borg:/var/lib/borg-backups/::%7bhostname%7d_%7bnow:%25d.%25m.%25Y%7d) / usr/share/nginx/html /etc/nginx






	 

