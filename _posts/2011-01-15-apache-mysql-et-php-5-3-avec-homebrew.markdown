---
comments: true
date: 2011-01-15 13:11:08
layout: post
slug: apache-mysql-et-php-5-3-avec-homebrew
title: Apache, mySQL et PHP 5.3 avec Homebrew
wordpress_id: 271
categories:
- Geek
tags:
- apache
- homebrew
- howto
- mac os x
- macports
- mysql
- php
---

Il y a quelques mois, j'ai publié un article sur l'installation d'[Apache, mySQL et PHP 5.3 via le gestionnaire de paquets MacPorts](http://www.vincentlemaire.com/apache-mysql-et-php-5-3-avec-macports), je vous propose aujourd'hui un nouveau billet sur le même sujet mais avec un autre gestionnaire de paquets : [Homebrew](http://mxcl.github.com/homebrew/) !


# Pourquoi Homebrew ?


J'ai découvert Homebrew il y a quelques semaines en discutant avec mes collègues [Clever Agiens](http://www.clever-age.com). Comme j'avais eu quelques soucis lors d'une mise à jour de MacPorts, j'ai décidé de voir ce qu'avait ce nouvel outil dans le ventre et s'il était si "simple" et "flexible" que son auteur le clame !

Je partais déjà du constat suivant : MacPorts c'est bien mais ce n'est pas la panacée. Certes, l'outil est puissant mais il a plusieurs gros défauts pour moi : les installations sont affreusement lentes et j'ai eu régulièrement des problèmes lors de la mise à jour des paquets (sans compter le temps qu'elles prennent, cf. premier point).

Cet type d'outil était relativement "vital" pour un développeur web, il faut prendre un minimum de recul sur le projet en lui-même :



	
  * le projet est "ouvert", son code est disponible via Github... Premier bon point.

	
  * 3 500 suiveurs du projet, 1 500 forks et [plusieurs commits](https://github.com/mxcl/homebrew/commits/master) sur la dernière journée... Le projet est actif, deuxième bon point.


Le projet semble actif et bien maintenu, ce qui est donc rassurant. Direction le Dock, clic sur Terminal et c'est parti pour l'installation.


# Installation de Homebrew


Préalablement à l'installation de Homebrew, il est conseillé de faire un peu le ménage sur son Mac. Comme toujours, n'oubliez pas de faire une sauvegarde et d'avoir, par exemple, votre disque Time Machine à portée, ça peut toujours servir ;-)

Nous allons donc supprimer notre installation de MacPorts et faire le ménage dans nos dossiers pour l'arrivée du petit dernier :

    
    $ sudo rm -rf /usr/local/include
    $ sudo rm -rf /usr/local/lib
    $ sudo rm -rf /opt/local /Applications/DarwinPorts /Applications/MacPorts /Library/LaunchDaemons/org.macports.* /Library/Receipts/DarwinPorts*.pkg /Library/Receipts/MacPorts*.pkg /Library/StartupItems/DarwinPortsStartup /Library/Tcl/darwinports1.0 /Library/Tcl/macports1.0 ~/.macports


Une fois cela fait, l'accouchement n'est qu'à la prochaine ligne :

    
    $ ruby -e "$(curl -fsSL https://raw.github.com/gist/323731)"


Alors que l'on s'attend à devoir partir prendre un café pour échapper au bruit assourdissant du ventilateur de sa machine, rien de tout ça : l'installation est d'une rapidité déconcertante. En quelques secondes, l'outil est prêt à l'emploi. C'est comme l'effet [wahou de Windows Vista](http://www.youtube.com/watch?v=9RFXoupKDko), Windows et Flavie en moins ;-)

On en profite pour installer quelques petits outils pratiques (wget, git et bash-completion) qui nous permettront de contempler la rapidité d'exécution de Homebrew.

    
    $ brew install wget git bash-completion




# Installation de mySQL


On continue avec l'installation de notre serveur mySQL :

    
    $ brew install mysql
    $ unset TMPDIR
    $ mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp


On ajoute un fichier my.cnf (à placer dans /etc/) pour définir le socket :

    
    [mysqld]
    socket = /tmp/mysql.sock
    
    [client]
    socket = /tmp/mysql.sock


On copie le fichier de démarrage pour que launchd puisse le lancer au démarrage du Mac :

    
    $ mkdir -p ~/Library/LaunchAgents
    $ cp /usr/local/Cellar/mysql/5.5.15/com.mysql.mysqld.plist ~/Library/LaunchAgents/
    $ launchctl load -w ~/Library/LaunchAgents/com.mysql.mysqld.plist


Concernant l'avant dernière ligne, j'attire votre attention sur la version installée : n'oubliez pas de modifier le path :)

On sécurise notre installation :

    
    $ mysql_secure_installation


Et le tour est jour ;-)


# Installation de PHP et configuration d'Apache


Homebrew ayant la volonté de ne pas proposer d'outil déjà intégré au sein de Mac OS X, il faut utiliser un dépôt alternatif qui propose un certain nombre de paquets avec des versions plus à jour que celles intégrées au système :

    
    $ brew install https://github.com/adamv/homebrew-alt/raw/master/duplicates/php.rb --with-apache --with-mysql
    $ chmod -R ug+w /usr/local/Cellar/php/5.3.8/lib/php
    $ pear config-set php_ini /usr/local/etc/php.ini


A noter que si vous avez besoin de pouvoir switcher simplement entre PHP 5.2 et 5.3, Bastnic propose une [solution élégante](http://www.bastnic.info/index.php/post/2011/01/22/Switcher-entre-PHP-5.2-et-PHP-5.3-sur-Mac-OS-X-et-Homebrew).

Comme nous utilisons le serveur Apache fourni par Mac OS X, il faut modifier le fichier de configuration de ce dernier afin qu'il utilise notre version de PHP :

    
    $ vim /etc/apache2/httpd.conf
    + LoadModule php5_module    /usr/local/Cellar/php/5.3.8/libexec/apache2/libphp5.so


Afin que la CLI de PHP soit celle de notre version de PHP, nous ajoutons une petite ligne à notre fichier de profil Bash :

    
    $ echo 'export PATH='`brew --prefix php`'/bin:$PATH' >> .bash_profile


On peut aussi installer Xdebug via la commande suivante :

    
    $ brew install xdebug


On ajoutera également la ligne suivante à notre php.ini (n'oubliez pas de modifier le path par la version installée) :

    
    $ vim /usr/local/etc/php.ini
    + zend_extension="/usr/local/Cellar/xdebug/2.1.0/xdebug.so"


On en profite également pour définir quelques valeurs par défaut pour notre php.ini :

    
    $ sudo -i
    $ cd /usr/local/etc/
    $ defSock=`mysql_config --socket`
    $ cat php.ini | sudo sed \
    -e "s#pdo_mysql\.default_socket.*#pdo_mysql\.default_socket=${defSock}#" \
    -e "s#mysql\.default_socket.*#mysql\.default_socket=${defSock}#" \
    -e "s#mysqli\.default_socket.*#mysqli\.default_socket=${defSock}#" > php.ini-tmp
    $ grep default_socket php.ini-tmp
    $ mv php.ini-tmp php.ini
    $ exit




# C'est fini


Les principaux services dont nous avions besoin sont installés. Il reste encore un peu de tuning à faire pour activer le server-status d'Apache, définir notre premier Virtualhost ou configurer phpMyAdmin. Tout étant expliqué en détail dans [mon post sur MacPorts](http://www.vincentlemaire.com/apache-mysql-et-php-5-3-avec-macports), je vous invite à suivre les explications ;-).


# Sources





	
  * [Install PHP 5.3 with Homebrew on 10.6 Snow Leopard](http://notfornoone.com/2010/07/install-php53-homebrew-snow-leopard/)

	
  * [Installer Apache, MySQL, PHP avec Homebrew sur MacOSX](http://www.remi-montagu.fr/blog/developpement/installer-apache-mysql-php-avec-homebrew-macosx/)


