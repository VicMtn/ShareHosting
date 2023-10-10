## TODO : 
- [ ] Choisir la distribution
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
   - 
- [ ] Choisir le poste d'installation
- [ ] Installer la distribution
- [ ] Installer les services
  - [ ] SSH
  - [ ] Nginx
        - [ ] su apt-get update
        - [ ] su apt-get upgrade
        - [ ] su apt install php
              - [ ] check the version with : php -v --> version 8.2.7 installed
        - [] su apt install php-fpm php-mysql -y
              - [] check the installed modeul with : systemctl status php8.2-fpm
              - 
     lien vers l'install : https://itslinuxfoss.com/install-nginx-debian-12-linux/
  - [ ] PHP-FPM configuration :  https://www.digitalocean.com/community/tutorials/php-fpm-nginx //// https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Nginx-PHP-FPM-config-example
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
      
- [ ] Configurer les services
- [ ] Créer les utilisateurs
- [ ] Créer les bases de données
- [ ] Créer les sites web
- [ ] Créer les fichiers de configuration
- [ ] Créer les scripts de création d'utilisateurs
- [ ] Tester les scripts
- [ ] Tester les sites web
- [ ] Tester les bases de données
- [ ] Ecrire le rapport