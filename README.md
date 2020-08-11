# Installation de PHP & Cie sur Ubuntu by Alex32

Avant tout, tout ce qui va suivre m'a été expliqué par **Alex32**.
Je tiens tout personnellement à le remercier pour son implication.

> **Alex32** m'a expliqué pas à pas ce qu'il faut faire pour ré-installer PHP et compagnie. Avec son accord je reprends tout son processus pour avoir une sauvegarde en cas de ré-installation d'un système Ubuntu 20+.
> Vous êtes libre d'utiliser ce tutoriel au cas où si vous rencontre un quelconque problème.

## Etape 1 - Installation d'apache2

- Installer apache2
  - `sudo apt install -y apache2 apache2-utils`
- Vérifier l'état une fois installé
  - `systemctl status apache2`
- Si apache2 n'est pas démarré
  - `sudo systemctl start apache2`
- Pour que Apache2 démarre à chaque fois au démarrage de l'ordinateur
  - `sudo systelctl enable apache2`

> A partir de maintenant, il est possible de saisir dans la barre d'adresse d'un navigateur, **127.0.0.1** ainsi la page par défaut sera visible.

## Etape 2 - Changé le propriétaire de la racine du document

- Définir le user apache comme propriétaire de la racine du document
  - `sudo chown www-data:www-data /var/www/html/ -R`
- Petite modification pour accéder à localhost, vu que pour le moment nous pouvons accéder uniquement à **127.0.0.1**
  - `sudo nano /etc/apache2/conf-available/servername.conf`
- Dans ce fichier on rajoute ceci :
  - `ServerName localhost`
    - On enregistre avec **CTRL** + **S**
    - On quitte avec **CTRL** + **X**
- On active la modification
  - `sudo a2enconf servername.conf`
- On relance le server Apache
  - `sudo systemctl reload apache2`

> Voilà, à ce stade, il est tout à fait possible d'utiliser dans le navigateur: **localhost** au lieu de **127.0.0.1**

## Etape 3 - MariaDB

- Installation de MariaDB
  - `sudo apt install mariadb-server mariadb-client`
- Vérification de l'état
  - `systemctl status mariadb`
- Si le service n'est pas démarré
  - `sudo systemctl start mariadb`
- Le démarrer automatiquement au démarrage de l'ordinateur
  - `sudo systemctl enable mariadb`
- (Bonus) - Stopper le service
  - `sudo systemctl stop mariadb`
  - ou `sudo systemctl stop apache2`

> A ce moment, mariaDB est installé sauf qu'il n'est pas sécurisé, on y vient dans l'étape -

## Etape 4 - MariaDB (Sécurisé)

- Configuration de mariaDB
  - `sudo mysql_secure_installation`
- 1ère ligne appuyer sur entrée pour valider. Le mot de passe (vide).
- Appuyer sur la touche "**Y**" pour définir un mot de passe.
  - Le (mot de passe ne s'affiche pas sur Linux)
  - Une confirmation vous sera demandée
- Ensuite, on appuira plusieurs fois sur la touche "**Y**" pour accepter toutes les étapes qui suivent.
  - Suppression des utilisateurs anonyme
  - Désactive la connexion root à distance
  - Supprimer la base de donnée test
- Connexion en root
  - `sudo mariadb -u root`
- Pour quitter on saisira **exit** et on validera par la touche entrer du clavier.

> Par défaut mariadb tu te connecte avec ton nom utilisateur et mot de passe de ton système, le mieux c'est de ce connecter en root sans mot de passe. (__le mot de passe sert a rien puisque tu es en local et que tu es en root,__).
> Nous pouvons passer à la configuration de PHP.

## Etape 5 - PHP & ces modules

- installation php et tout les modules.
  - `sudo apt install php7.4 libapache2-mod-php7.4 php7.4-mysql php-common php7.4-cli php7.4-common php7.4-json php7.4-opcache php7.4-readline`
- On active PHP
  - `sudo a2enmod php7.4`
- Re-démarrer les services
  - `sudo systemctl restart apache2`

> Pour voir si les fichiers PHP sont pris en compte, on va créer un fichier **phpinfo.php** et lui définir un contenu.

- Création du fichier et édition de celui-ci en ligne de commande grâce à nano
  - `sudo nano /var/www/html/phpinfo.php`
- On ajoute ce contenu : **<?php phpinfo();**
- On sauvegarde avec **CTRL** + **S**
- On quitte l'éditeur nano avec **CTRL** + **X**

> On peut voir le contenu du fichier dans le navigateur. Si ça fonctionne c'est que tout est bon pour le moment. **localhost/phpinfo.php**
> Il faut supprimer ce fichier car c'est un peut dangereux de laisser ces informations visible par n'importe qui.

- Suppression du fichier **phpinfo.php**
  - `sudo rm -rf /var/www/html/phpinfo.php`

> Voilà PHP est installé on peut installer maintenant **phpmyadmin** et le configurer traquillement.

## Etape 6 - PHPMYADMIN

> Par défaut si on ne fait rien, le nom d'utilisateur sera **phpmyadmin** et le mot de passe sera celui qui aura été créer juste après.
> Il faut connaître son nom d'utilisateur. Pour le savoir dans un terminal, moi j'ai ceci: **zyrass@zyrass-dev:~$** donc mon nom d'utilistaeur correspond à **zyrass**. Adaptez avec le vôtre.

- Être propriétaire du dossier tout en faisant parti du groupe pour apache2
  - `sudo chown -R zyrass /var/www/html`
- Installation de phpmyadmin
  - `sudo apt install phpmyadmin`
- Donc à ce moment, on appuie sur la touche **ESPACE** pour avoir un petit **astérisque**..
- On appuie sur la touche **TAB** et on valide la sélection par la touche **ENTER**
- On saisit le même mot de passe qu'on a définit précédemment (__pour éviter de se tromper__).

> Nous pouvons tester la connexion à **phpmyadmin** via cette URL : **localhost/phpmyadmin**.

- Pour rappel, le nom d'utilisateur est :
  - **phpmyadmin**
- le mot de passe
  - Celui qui a été définit.

## Etape 7 - Créer un compte admin sur phpmyadmin

> Vous comprendrez très vite que le nom d'utilisateur "**phpmyadmin**" n'est pas vraiment terrible terrible... Allez on va changer ça.

- On se connecte en root à mysql
  - `sudo mysql -u root`
- On saisit la commande suivante pour créer un utilisateur "**admin**" (A savoir : **'xxxxxxxxx'** est à remplacer par votre mdp)
  - `create user admin@localhost identified by 'xxxxxxxx';`
- On saisit cette ligne de commande pour lui attribuer tout les privilèges
  - `grant all privileges on *.* to admin@localhost with grant option;`
- On enchaîne immédiatement par :
  - `flush privileges;`
- Pour quitter on saisira : **exit**

## Améliorer les performances de PHP

> On va utiliser **php-fpm** pour améliorer les performances de PHP.

- On désactive le module php7.4 (**No stress**)
  - `sudo a2dismod php7.4`
- On install php-fpm
  - `sudo apt install php7.4-fpm` (On ne re-démarre pas encore le server apache2)
- On active le module proxy et environnement. (On ne re-démarre toujours pas le server apache2)
  - `sudo a2enmod proxy_fcgi setenvif`
- Enregistrement de la configuration
  - `sudo a2enconf php7.4-fpm`
- **On peut re-démarrer le server apache2**
  - `sudo systemctl restart apache2`

> Vous pouvez recréer un fichier phpinfo.php et regarder le contenu de celui-ci.
> En effet, tu verras que le module est bien activé (en 3 eme ligne).

## Remerciements

**Alex32** pour avoir passé un petit moment avec moi, pour chacune de ces étapes. Merci t'es super.
