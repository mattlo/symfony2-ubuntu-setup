# Symfony2 on Ubuntu

**Created in 2014 August**

Documentation for setting up a bare Symfony2 application on Ubuntu with PHP. A portion of this is ported from DigitalOcean's documentation for both Ubuntu LAMP setup and Symfony2 setup. This project assumes you already have Symfony2 bare application outside the box. Setup also assumes this box will eventually be production grade.

**Note:** Replace `your-project-name` with your project name.

## Installation Steps

### Update Ubunut's repository pointers

```
sudo apt-get update
```

### Install Apache Server

```
sudo apt-get install apache2
```

Add application document root directory

```
/var/www/vhost/your-project-name
```

Remove default vhost configuration

```
rm /etc/apache2/sites-enabled/000-default.conf
```

Add in a new configuration

```
vim /etc/apache2/sites-enabled/your-project-name.conf
```

Paste the following config

```
<VirtualHost *:80>
        DocumentRoot /var/www/vhost/hazchem-sms/web
        #ServerName mydomain.com
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/vhost/hazchem-sms/web>
                AllowOverride All
                Order allow,deny
                Allow from all
        </Directory>
</VirtualHost>
```

Optionally you can configure Apache to listen to a port by adding this above the config file and updating the virtual host tag

```
listen *:9010
<VirtualHost *:9010>
...
```


### Install DB instance (e.g.: MySQL)

```
sudo apt-get install mysql-server php5-mysql
```

```
sudo mysql_install_db
```

```
sudo mysql_secure_installation
```

**Don't change the root password**, but say yes for all other options.

### Create Database

```
mysql -u root -p
```

```
CREATE DATABASE `your-project-name`;
```

type `exit;` to escape mysql

### Install PHP 5.5+

```
sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt php-apc php5-curl php5-cli php5-json
```

### Install Composer

Package manager service.


```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

### Update php.ini

First find the PHP ini location

```
php -i | grep 'Configuration File'
```

Then `vim` into the first line item

```
vim /etc/php5/cli/php.ini
```

Then search for `date.timezone`, uncomment it, and set the value to `America/Chicago`. E.g.:

```
date.timezone = America/Chicago
```

Move the pointer to the end of the file, add a new line, then copy the configs below:

```
short_open_tag = Off
magic_quotes_gpc = Off
register_globals = Off
session.auto_start = Off
```

### Swap Memory for <1GB RAM boxes

**For smaller instances only**, follow the steps below (this is so composer doesn't die while installing)

Switch to root and follow these steps to add the swap space -

Type the following command with count being equal to the desired block size:

```
dd if=/dev/zero of=/swapfile bs=1M count=1024
```

Setup the swap file with the command:

```
mkswap /swapfile
```

To enable the swap file immediately but not automatically at boot time:

```
swapon /swapfile
```

To enable it at the boot time, add the following entry into /etc/fstab:

```
/swapfile swap swap defaults 0 0
```

### Mount the application

This step assumes you can easily access the code base and can easily be deployed via SCP, SFTP/FTP, GIT, etc...

Point the application code base to `/var/www/vhost/your-project-name`

### Permissions

```
cd /var/www/vhost/your-project-name
chown -R root:www-data app/cache
chown -R root:www-data app/logs
chown -R root:www-data app/config/parameters.yml
```

#### On OS X
```
sudo chmod +a "_www allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
```

### Symfony2 Configuration

This file should mimic your `app/config/parameters.yml` but obviously not have the same values

### Symfony2 Install

```
SYMFONY_ENV=prod composer install --optimize-autoloader --no-dev
php app/console doctrine:schema:update --force --env=prod
php app/console router:dump-apache -e=prod --no-debug
php app/console cache:clear --env=prod
php app/console assets:install web --symlink --env=prod
```

```
chmod -R 777 /var/www/vhost/your-project-name
```

## References
- https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04
- https://www.digitalocean.com/community/tutorials/how-to-install-and-get-started-with-symfony-2-on-an-ubuntu-vps
- http://yourstory.com/2012/02/adding-swap-space-to-amazon-ec2-linux-micro-instance-to-increase-the-performance/