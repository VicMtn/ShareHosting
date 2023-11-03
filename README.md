
  - [ ] PHP-FPM configuration :  php-fpm-nginx //// 
      -  Configure fpm pool : in the file etc/nginx/sites-available/default
      -  Add the index.php in the list of indexes
            - Uncomment the following lines :
            ```
                -   #pass PHP scripts on Nginx to FastCGI (PHP-FPM) server
                  location ~ \.php$ {
                  include snippets/fastcgi-php.conf;

                  #Nginx php-fpm sock config:
                  fastcgi_pass unix:/run/php/php8.1-fpm.sock;
                  #Nginx php-cgi config :
                  #Nginx PHP fastcgi_pass 127.0.0.1:9000;
                  }

                  #deny access to Apache .htaccess on Nginx with PHP, 
                  #if Apache and Nginx document roots concur
                  location ~ /\.ht {
                  deny all;
                  }
               ```
      - validate the nginx config file :
          - su - nginx -t : output nginx the configuration file /etc/nginx/nginx.conf syntax is ok
          configuration file /etc/nginx/nginx.conf test is succesful
      - restart the service : su - systemctl restart nginx
  - [ ] MariaDB
      - su apt update 
      - su apt upgrade
      - apt show mariadb-server
      - su apt install mariadb-server -y
      - su mariadb-secure-installation
            - [ ] Enter current password for root (enter for none) : root
            - [ ] Switch to unix_socket authentication [Y/n] n
            - [ ] Change the root password ? [Y/n] n
            - [ ] Remove anonymous users ? [Y/n] Y
            - [ ] Disallow root login remotely ? [Y/n] n
            - [ ] Remove test database and acces to it ? [Y/n] n
            - [ ] Reload privilege tables now [Y/n] Y
# Share hosting
## Config VM
- Version : Debian 12.1.0
- Nom : sharehosting
- Nom de domaine : cpnv
- Root password : root
- Name of the user : noah
- Username : noah
- User password : cpnv
- Partion guidée pour l'entier du disque
- Tous les fichiers dans une seul partition
- Mirror country : Suisse
- Archive mirror : deb.debian.org
- HTTP proxy : none

## Installation Nginx
 ```
su -
apt update
apt upgrade
apt install ngnix

 ```
 ## Installation PHP
  ```
su -
apt update
apt upgrade
apt install php

 ```
 ### Installation de php-fpm et php-mysql
 ```
 apt install php-fpm php-mysql -y
 # Vérification
 systemctl status php8.2-fpm
 ```
 ## Configuration PHP-FPM
 Rendez-vous dans le fichier : etc/nginx/sites-available/default
```
nano /etc/nginx/sites-available/default

server {
  # Example PHP Nginx FPM config file
  listen 80 default_server;
  listen [::]:80 default_server;
  root /var/www/html;

  # Add index.php to setup Nginx, PHP & PHP-FPM config
  index index.php index.html index.htm index.nginx-debian.html;

  server_name _;

  location / {
    try_files $uri $uri/ =404;
  }

  # pass PHP scripts on Nginx to FastCGI (PHP-FPM) server
  location ~ \.php$ {
    include snippets/fastcgi-php.conf;

    # Nginx php-fpm sock config:
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    # Nginx php-cgi config :
    # Nginx PHP fastcgi_pass 127.0.0.1:9000;
  }

  # deny access to Apache .htaccess on Nginx with PHP, 
  # if Apache and Nginx document roots concur
  location ~ /\.ht {
    deny all;
  }
} 
```
Ensuite validé la configuration
```
nginx -t
```
Ensuite relancé le service
```
systemctl restart nginx
``` 
## Installation MariaDB
```
su -
apt update 
apt upgrade
apt show mariadb-server
apt install mariadb-server -y
```
Lancer ensuite la commande suivante et suiver les instructions 
```
     su mariadb-secure-installation
            - Enter current password for root (enter for none) : root
            - Switch to unix_socket authentication [Y/n] n
            - Change the root password ? [Y/n] n
            - Remove anonymous users ? [Y/n] Y
            - Disallow root login remotely ? [Y/n] Y
            - Remove test database and acces to it ? [Y/n] Y
            - Reload privilege tables now [Y/n] Y
```
# Création de l'utilisateur et attribution des droits
## Création de l'utilisateur

```
su -
useradd client -m -d /home/client

passwd client

```
## Création du groupe pour l'utilisation de ssh
```
addgroup ssh_client
```

## Ajouter votre utilisateur
```
usermod -aG ssh_client <USERNAME>
```

## Modification de la config ssh
Ajouter la ligne AllowGroups ss
```
nano /etc/ssh/sshd_config

AllowGroups ssh_users
```
Relancer SSH
```
systemctl restart sshd
```

## Ajout de l'utilisateur dans le groupe

```
usermod -aG ssh_client client
```

## Creation du répertoire et attribiution des droits
```
mkdir /home/client/www

touch /home/client/www/index.php

chown client:client /home/client
chmod 711 /home/client

echo "<?php phpinfo(); ?>" > /home/client/www/index.php
chmod 710 /home/client/www/index.php

chown -R client:www-data /home/client/www/
chmod 2750 /home/client/www/
```

# Configuring php-fpm et création de la pool
On va copier la configuration de base et ensuite la moodifier
```
cp /etc/php/8.2/fpm/pool.d/www.conf /etc/php/8.2/fpm/pool.d/client.conf

nano /etc/php/8.2/fpm/pool.d/client.conf
```

```
[client]

user = client
group = client

listen = /var/run/php/php8.2-fpm-client.sock

listen.owner = www-data
listen.group = www-data

env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

Ensuite relancé php fpm

```

systemctl restart php8.2-fpm
```

# Configuration de ngnix

```
nano /etc/nginx/sites-available/client
```

Copier le code suivant

```
server {
        listen 80;

        server_name client.ch www.client.ch;
        root /home/client/www;
        index index.php index.html index.html;

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass unix:/var/run/php/php8.2-fpm-client1.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}
```
Crée ensuite un lien symbolique et ensuite tester la configuration si tout est bon redémarer  

```
ln -s /etc/nginx/sites-available/client /etc/nginx/sites-enabled/

nginx -t

systemctl restart nginx

systemctl restart php8.2-fpm
```

# Base de donnée
Crée la base de donnée l'utilisateur et donné lui les droits
```
mariadb

CREATE DATABASE client;

CREATE USER 'client'@'localhost' IDENTIFIED BY 'password';

GRANT ALL PRIVILEGES ON client.* TO 'client'@'localhost' IDENTIFIED BY 'password';

FLUSH PRIVILEGES;
```
# Changé le host sur la machine client
sur windows rendez-vous ici :
`C:\Windows\System32\drivers\etc\host`

```
ip_machine client.ch
```

## Envoyé les fichiers
Etant donnée que les utilisateurs n'ont pas de bash il faut utiliser cette commande pour ajouter des fichier au site
```
scp -p ./<FILE>/* <USERNAME>@<HOSTNAME>:~/www
```

# Sources
- https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Nginx-PHP-FPM-config-example
- https://www.digitalocean.com/community/tutorials/
- https://itslinuxfoss.com/install-nginx-debian-12-linux/
