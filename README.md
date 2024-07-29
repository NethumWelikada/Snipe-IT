#Install apache2 by using below command
sudo apt install apache2 -y

#Enable and Start apache2 service
systemctl start apache2 && systemctl enable apache2

#check the status of the service
systemctl status apache2

#check the version of apache2 to validate
apache2 -v

#if firewall is enabled you need to open HTTP and HTTPS ports to this server. 
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload

#Snipe-IT needs various extensions, let us enable one which is mandatory
sudo a2enmod rewrite
systemctl restart apache2

#MySql / MariaDB installation
sudo apt install mariadb-server mariadb-client -y

#Enable and Start the service and then check the version and status of the service
sudo systemctl start mariadb && sudo systemctl enable mariadb
sudo systemctl status mariadb

#Before we proceed further let us now secure the installation by running the below command
sudo mysql_secure_installation

#PHP Installation
sudo apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath

#required extensions by snipe-it are also installed which
sudo apt install php php-bcmath php-bz2 php-intl php-gd php-mbstring php-mysql php-zip php-opcache php-pdo php-calendar php-ctype php-exif php-ffi php-fileinfo php-ftp php-iconv php-intl php-json php-mysqli php-phar php-posix php-readline php-shmop php-sockets php-sysvmsg php-sysvsem php-sysvshm php-tokenizer php-curl php-ldap -y

#install PHP composer
sudo curl -sS https://getcomposer.org/installer | php

#move the composer installer to the recommended directory
sudo mv composer.phar /usr/local/bin/composer

#Create Database in MariaDB for Snipe-IT
mysql -u root -p

#create the DB
CREATE DATABASE snipeitdb;
#CREATE USER snipeit@localhost IDENTIFIED BY 'Password';
CREATE USER snipeit@localhost IDENTIFIED BY 'snipeit#12@';
#GRANT ALL PRIVILEGES ON snipeitdb.* TO snipeituser’@’localhost;
#GRANT ALL PRIVILEGES ON snipeitdb.* TO 'snipeituser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON snipeitdb.* TO 'snipeit'@'localhost' IDENTIFIED BY 'snipeit#12@';
FLUSH PRIVILEGES;
EXIT;

#Installation of Snipe-IT
#move to the Website Root Directory.
cd /var/www/

#latest repository of Snipe-IT can be cloned from GIT
git clone https://github.com/snipe/snipe-it snipe-it

#open the folder
cd /var/www/snipe-it

#The main configuration file is .env which is stored as example, we will make a copy of that as below;
sudo cp /var/www/snipe-it/.env.example /var/www/snipe-it/.env

#need to mainly look at the Application and the Database connection details
sudo nano /var/www/snipe-it/.env
#Change mainly the following;
APP_DEBUG=true
APP_URL=null
APP_TIMEZONE='Asia/colombo'
DB_DATABASE=snipeitdb
DB_USERNAME=snipeituser
DB_PASSWORD=test123

#set permission to the directories within the Snipe-IT directories use below commands
sudo chown -R www-data:www-data /var/www/snipe-it
sudo chmod -R 755 /var/www/snipe-it

#using the same folder /var/www/snipe-it. We will install the composer
sudo composer update --no-plugins --no-scripts
sudo composer install --no-dev --prefer-source --no-plugins --no-scripts

#generate the APPKEY which will automatically update the.env file with APP_KEY use below command while remaining in the same directory
php artisan key:generate

#Update Virtual Host
#disable the default virtual host in our web server
sudo a2dissite 000-default.conf

#create a Snipe-IT virtual host configuration
sudo nano /etc/apache2/sites-available/snipe-it.conf
#use below line to create this configuration and modify to meet your requirements.
<VirtualHost *:80>
ServerName snipe-it.syncbricks.com
DocumentRoot /var/www/snipe-it/public
<Directory /var/www/snipe-it/public>
Options Indexes FollowSymLinks MultiViews
AllowOverride All
Order allow,deny
allow from all
</Directory>
</VirtualHost>

#Enable Snipe-it configuration
a2ensite snipe-it.conf

#Folder Permissions for Upload and Storage
sudo chown -R www-data:www-data ./storage
sudo chmod -R 755 ./storage

#Restart Apache Server
systemctl restart apache2
