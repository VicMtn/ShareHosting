# Share hosting

# Création de l'utilisateur et attribution des droits
## Création de l'utilisateur

```
su -
useradd client1 -m -d /home/client

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

chown client:www-data -R /home/client

chmod 750 -R /home/client1
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
# restart php7.4
systemctl restart php8.2-fpm
```

# Configuration de ngnix

```
nano /etc/nginx/sites-available/client
```

Copy the following :

```
server {
        listen 80;

        server_name client.ch www.client.ch;
        root /home/client/www;
        index index.php index.html index.html;

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass unix:/var/run/php/php8.4-fpm-client1.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
}
```

The above code shows a common configuration for a server block in nginx.

- Web root is /home/client1/www
- The server name uses the domaine client1.ch
- fastcgi_pass specifies the handler for the php files. For every site you should use a different unix socket such as /var/run/php/php7.4-fpm-client1.sock

To enable the above site you have to create a symlink to it in the directory /etc/nginx/sites-enabled/

```
# Enable the website
sudo ln -s /etc/nginx/sites-available/client1 /etc/nginx/sites-enabled/

# Test the configuration
sudo nginx -t

# Restart nginx
sudo systemctl restart nginx

# Restart php
sudo systemctl restart php7.4-fpm
```

### Database

```
# Connect to mariadb
sudo mariadb

# Add a new database
CREATE DATABASE client1;

# Add a user
CREATE USER 'client1'@localhost IDENTIFIED BY 'password';

# Give the user the rights to his database
GRANT ALL PRIVILEGES ON client1.* TO 'client1'@localhost IDENTIFIED BY 'password';

# Refresh rights
FLUSH PRIVILEGES;
```

## Deploy website (Scp)

It is possible to transfer files to the website with SCP as follows .

```
# from client side
scp -pr ./<FOLDER>/* <USERNAME>@<HOSTNAME>:~/www
```

## Hosts file (client side)

You have to change the host file because there is no DNS.

### For windows

Open the hosts file with a text editor (with administrator rights) which is located here `C:\Windows\System32\drivers\etc`
Add the last line as below

```
ip_machine client1.ch www.client1.ch
```

### For MAC OS

1. Open the hosts file with a text editor located here `/etc/hosts`
1. Add, as in case 1, a line with the ip and domain names

You can now access the site from your browser by connecting to client1.ch

# Sources

- https://wiki.debian.org/fr/SSH
- https://grafikart.fr/tutoriels/nginx-692
- https://ostechnix.com/allow-deny-ssh-access-particular-user-group-linux/
- https://www.digitalocean.com/community/tutorials/how-to-host-multiple-websites-securely-with-nginx-and-php-fpm-on-ubuntu-14-04
- https://www.ionos.fr/digitalguide/serveur/configuration/fichier-host/