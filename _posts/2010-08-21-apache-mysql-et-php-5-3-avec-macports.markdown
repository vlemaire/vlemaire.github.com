---
comments: true
date: 2010-08-21 11:46:26
layout: post
slug: apache-mysql-et-php-5-3-avec-macports
title: Apache, mySQL et PHP 5.3 avec MacPorts
wordpress_id: 237
categories:
- Geek
tags:
- apache
- howto
- mac os x
- macports
- mysql
- php
---

Ayant récemment changé mon Mac, il m'a fallu repasser par l'étape (pénible) de réinstallation complète du système afin de repartir sur une base saine. Je ne détaillerai pas ici pourquoi je n'ai pas souhaité migrer mon ancien système sur le nouveau Mac mais je vais vous livrer un petit tutorial rapide pour installer un environnement de développement LAMP avec l'excellent gestionnaire de paquets [MacPorts](http://www.macports.org).

Ce billet repose principalement sur le [howto proposé par MacPorts](https://trac.macports.org/wiki/howto/MAMP) modifié par mes soins pour coller à mes besoins. Vos remarques sont d'ailleurs les bienvenues en commentaire, tout étant évidemment perfectible !


# Première étape : installation de MacPorts


Au cas où vous ne l'auriez pas fait, [téléchargez MacPorts](http://www.macports.org/install.php) en veillant bien à choisir la version de votre système d'exploitation. Lancez-le, prenez un café (au passage, prévoyez-en un stock pour la suite, ça sera utile) et patientez :)


# Deuxième étape : installation d'Apache


Dans un terminal, lancez la commande suivante :

    
    $ sudo port install apache2


N'oubliez pas de faire un tour dans les préférences système de Mac OS X pour désactiver le "Partage Web", _aka_ l'Apache fourni avec le système, histoire d'éviter tout conflit (bien qu'il soit possible de facilement faire tourner Apache sur un autre port que celui par défaut).

Afin qu'Apache se lance à chaque démarrage du Mac, il faut spécifier l'emplacement du fichier de lancement à _launchd_, le gestionnaire de lancement de programmes de Mac OS X. MacPorts fourni une commande pour faire cela aisément :

    
    $ sudo port load apache2


Passons ensuite à la configuration d'Apache (avec l'activation, entres autres, des _VirtualHosts_) :

    
    $ sudo vim /opt/local/apache2/conf/httpd.conf
    + Include conf/extra/httpd-info.conf
    + Include conf/extra/httpd-vhosts.conf
    + Include conf/extra/httpd-default.conf
    
    $ sudo vim /opt/local/apache2/conf/extra/httpd-info.conf
    
    <Location /server-status>
      SetHandler server-status
      Order deny,allow
      Deny from all
    + Allow from 127.0.0.1
    </Location>
    <Location /server-info>
      SetHandler server-info
      Order deny,allow
      Deny from all
    + Allow from 127.0.0.1
    </Location>


Afin de pouvoir accéder facilement à la commande de contrôle Apache, nous allons définir un alias dans notre fichier de profile _bash_ :

    
    $ vim .profile
    +  alias apache2ctl='sudo /opt/local/apache2/bin/apachectl'
    
    $ source .profile


Ainsi, vous pouvez démarrer, redémarrer, arrêter, vérifier la syntaxe des fichiers de configuration Apache, connaître le statut du serveur simplement à l'aide de la commande :

    
    $ apache2ctl start/restart/stop/configtest/status


Enfin, je vous conseille d'installer lynx, le navigateur en mode texte afin d'utiliser l'option _status_ de la commande :

    
    $ sudo port install lynx




# Troisième étape : installation de mySQL


Vient ensuite l'installation du SGBD mySQL :

    
    $ sudo port install mysql5-server
    $ sudo port load mysql-server
    $ sudo -u _mysql mysql_install_db5


Enfin, afin de sécuriser l'installation de mySQL, saisissez la commande :

    
    /opt/local/lib/mysql5/bin/mysql_secure_installation


On profite également de l'occasion pour que mySQL utilise par défaut l'UTF8, ça sera toujours un peu de temps de gagné pour la suite :

    
    $ sudo vim /opt/local/etc/mysql5/my.cnf
    + [mysqld]
    + default-character-set=utf8
    + default-collation=utf8_unicode_ci
    + character_set_server=utf8
    + collation_server=utf8_unicode_ci
    + default-storage-engine=INNODB




# Quatrième étape : installation de PHP


MacPorts propose par défaut la dernière version de PHP, soit la 5.3. Si vous souhaitez installer la version 5.2, il faudra installer le paquet php52.

    
    $ sudo port install php5 +apache2
    $ sudo port install php5-apc php5-curl php5-exif php5-ftp php5-gd php5-imagick php5-mbstring php5-memcache php5-mysql php5-soap php5-solr php5-xdebug php5-xmlrpc php5-xsl


Afin que PHP soit reconnu comme module Apache, il est nécessaire de faire une petite bidouille :

    
    $ cd /opt/local/apache2/modules
    $ sudo /opt/local/apache2/bin/apxs -a -e -n "php5" libphp5.so
    
    $ sudo vim /opt/local/apache2/conf/httpd.conf
    + Include conf/extra/mod_php.conf
    <IfModule dir_module>
    +  DirectoryIndex index.php index.html
    </IfModule>


On peut ensuite configurer PHP et définir le socket mySQL utilisé par MacPorts :

    
    $ sudo -i
    $ cd /opt/local/etc/php5
    $ cp php.ini-development php.ini
    $ defSock=`/opt/local/bin/mysql_config5 --socket`
    $ cat php.ini | sudo sed \
    -e "s#pdo_mysql\.default_socket.*#pdo_mysql\.default_socket=${defSock}#" \
    -e "s#mysql\.default_socket.*#mysql\.default_socket=${defSock}#" \
    -e "s#mysqli\.default_socket.*#mysqli\.default_socket=${defSock}#" > tmp.ini
    $ grep default_socket php.ini.tmp
    $ mv php.ini.tmp php.ini
    $ exit




# Cinquième étape : configuration d'un VirtualHost Apache



    
    $ sudo vim /opt/local/apache2/conf/extra/httpd-vhosts.conf
    NameVirtualHost *:80
    
    <VirtualHost *:80>
     DocumentRoot "/path/de/votre/
     ServerName projet.local
    
     <Directory "/path/de/votre/projet">
     Order allow,deny
     Allow from 127.0.0.1
     </Directory>
    </VirtualHost>
    
    $ sudo vim /etc/hosts
    +  127.0.0.1       projet.local




# Etape complémentaire : installation de phpMyAdmin



    
    $ sudo port install phpmyadmin
    $ sudo vim /opt/local/apache2/conf/extra/httpd-vhosts.conf
    <VirtualHost *:80>
      DocumentRoot "/opt/local/www/phpmyadmin"
      ServerName phpmyadmin.local
    
      <Directory "/opt/local/www/phpmyadmin">
        Order allow,deny
        Allow from 127.0.0.1
      </Directory>
    </VirtualHost>
    
    $ sudo vim /etc/hosts
    +  127.0.0.1       phpmyadmin.local
    
    $ apache2ctl restart


Vous avez désormais une plateforme LAMP de développement disponible sur votre Mac. Il existe beaucoup de "ports" disponibles, je vous invite à faire un tour dans la [liste des paquets disponibles](http://guide.macports.org) et à [lire le guide](http://www.macports.org/ports.php) pour que la liste de commande de MacPorts n'ait plus de secret pour vous !
