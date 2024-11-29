# dev setup

## Command to get the tools on Linux

If your computer is using a Debian based operating system, you can install all the required packages with the following command:

```sh
 sudo apt-get install php php-curl php-gd php-cli mysql-server php-mysql php-xml php-mbstring
```

## Get & Install Matomo

We'll get the latest version of Matomo's source code using git.

Open a terminal, cd into the directory where you want to install Matomo, and then run the following commands (without the leading $):

```sh
git clone https://github.com/matomo-org/matomo matomo
cd matomo
git submodule update --init
```

## Get composer

Follow the download instructions for Composer in order to get composer on your machine:

Download the installer to the current directory:

```sh
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```

Verify the installer SHA-384:

```sh
php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

Run the installer:

```sh
php composer-setup.php
```

Remove the installer:

```sh
php -r "unlink('composer-setup.php');"
```

(optinoal) put the composer.phar into a directory on your PATH:

```sh
sudo mv composer.phar /usr/local/bin/composer
```

## Install Composer

Then run this command to download all PHP packages that Matomo requires:

```sh
php composer.phar install
```
or 
```sh
composer install
```


On Windows, you will likely need to add an option --no-script:

```sh
php composer.phar install --no-script
```


## Get a web server

If your PHP version is greater than 5.4, you can also use PHP's built-in web server which requires no installation. Simply run the following command:

```sh
php -S 0.0.0.0:8000
```

If you use Apache or Nginx, the specific instructions for configuring your web server depend on the web server itself.

Matomo should now be available at http://localhost:8000/. To stop the web server, just hit Ctrl+C.

Remember that PHP's built in web server is only suitable for development. It should never be used in production.


## Install MySQL 

At first, install packages and security command

```sh
sudo apt install mysql-server mysql-client php-mysql mysql-common
sudo mysql_secure_installation 
```

Then stop mysql service:

```sh
systemctl stop mysql
```

Login to mysql server:

```sh
mysql -u root
```

Then run this command to reload the grant tables:

```sh
FLUSH PRIVILEGES;
```

Now you can be able to set a new password for the root account:

```sh
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
```

## Install phpmyadmin (optional)


First, install phpmyadmin package

```sh
sudo apt install phpmyadmin
```

Then run the following commands to open phpmyadmin in browser (localhost/phpmyadmin):

sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin.conf
sudo systemctl reload apache2.service
```


## create a database

When you install Matomo, at the database creation step, you will need to specify your database user.

1. Connect to your MySQL database:

```sh
mysql -u root -p
```

2. Create a database for Matomo:

```sh
mysql> CREATE DATABASE matomo_db_name_here;
```

3. Create a user called matomo, if you are using MySQL 5.7 or MySQL 8 or newer:

```sh
mysql> CREATE USER 'matomo'@'localhost' IDENTIFIED WITH mysql_native_password BY 'my-strong-password-here';
```

Or if you are using an older version such as MySQL 5.1, MySQL 5.5, MySQL 5.6:

```sh
mysql> CREATE USER 'matomo'@'localhost' IDENTIFIED BY 'my-strong-password-here';

```

4. Grant this user matomo the permission to access your matomo_db_name_here database:


```sh
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, INDEX, DROP, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES ON matomo_db_name_here.* TO 'matomo'@'localhost';
```

It is important to grant the user the following privileges: SELECT, INSERT, UPDATE, DELETE, CREATE, INDEX, DROP, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES

5. (OPTIONAL) Grant this user matomo the FILE global privilege: (if enabled, reports will be archived faster thanks to the LOAD DATA INFILE feature):

```sh
mysql> GRANT FILE ON *.* TO 'matomo'@'localhost';
```

In these instructions:

*replace matomo_db_name_here with the name of your MySQL database (if possible, this database should only have the Matomo tables installed).

*replace my-strong-password-here by a strong secure password.

*replace matomo by your chosen MySQL username (or simply use matomo).

*done

## Install Matomo

next from welcome level and system check

localhost:8000


```js
<!-- Matomo -->
<script>
  var _paq = window._paq = window._paq || [];
  /* tracker methods like "setCustomDimension" should be called before "trackPageView" */
  _paq.push(['trackPageView']);
  _paq.push(['enableLinkTracking']);
  (function() {
    var u="//localhost:8000/";
    _paq.push(['setTrackerUrl', u+'matomo.php']);
    _paq.push(['setSiteId', '1']);
    var d=document, g=d.createElement('script'), s=d.getElementsByTagName('script')[0];
    g.async=true; g.src=u+'matomo.js'; s.parentNode.insertBefore(g,s);
  })();
</script>
<!-- End Matomo Code -->
```


