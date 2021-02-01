# PHP Web Development environment By Docker

- Composed of Nginx + PHP-FPM + MariaDB + phpMyAdmin
- Use the official Docker images, or generate a custom image based on the official image
- Use Alpine based images, smaller in size
- Declare environment variables in .env file, you can choose PHP and MariaDB version
- Provide PHP (php.ini) and MariaDB (my.cnf) configuration files, you can adjust the settings yourself
- The container only provides a development environment, and all self-developed content is stored on the host
- Use php and composer command on the host

## README.md
zh_TW [繁体中文](README.md)


## Requirements

- git
- docker & docker-compose


## Usage

```bash
    $ git clone https://github.com/jazchen/php-web-dev-docker.git myweb
    $ cd myweb
    $ docker-compose up -d
```
After the images is downloaded and custom image created, the container will be starting.
If the three ports of host 80, 443, and 3306 are not occupied by other programs, the container will start smoothly.\
Then open the browser and enter the URL http://127.0.0.1 to see the phpinfo() screen


## Environment default

- PHP version: **7.2**, MariaDB version: **10.3**
- Use three ports on host
	- **HTTP 80**
	- **HTTPS 443**
	- **MariaDB 3306**
- Website root directory **webroot**
- MariaDB data directory **mysql-data**


## Environment variables .env file

- **Nginx must be 1.19 or newer** (will extract environment variables before nginx starts)

```
##### EzBox PHP Web Development Environment by docker-compose start #####
# common
DOCKER_RESTART_MODE=unless-stopped

# for php-fpm
DOCKER_PHP_VERSION=7.2
DOCKER_PHP_IMAGE_POSTFIX=

# for nginx
DOCKER_NGINX_TAG=1.19-alpine
DOCKER_NGINX_PORT_HTTP=80
DOCKER_NGINX_PORT_HTTPS=443
DOCKER_NGINX_WEBROOT_DIR=webroot

# for MariaDB
DOCKER_MARIADB_TAG=10.3
DOCKER_MARIADB_PORT=3306
DOCKER_MARIADB_UID_GID="1000:1000"
DOCKER_MARIADB_DATA_DIR=./mysql-data
DOCKER_MYSQL_ROOT_PASSWORD=DB_PassWord

# for phpMyAdmin
DOCKER_PHPMYADMIN_URI=/phpMyAdmin
##### EzBox PHP Web Development Environment by docker-compose end #####
```


## Other settings
The configuration files are placed in the `.conf` folder
- **nginx/ssl/** - Mapping to `/etc/nginx/ssl` in the nginx container
- **nginx/default.conf.template** - template of nginx conf file
- **my.cnf** - Initial my.cnf of MariaDB
- **php.ini** - Initial php.ini-development file of PHP
- **php-xdebug.ini** - php xdebug configuration file, use 3.x version by default


## PHP

Specify the PHP version with **DOCKER_PHP_VERSION** in the `.env` file. The default is 7.2.\
Auto create a new custom image from `php:7.2-fpm-alpine`, install composer and the following php extension
- mysqli
- pdo_mysql
- bcmath
- gettext
- yaml
- xdebug

The custom image tag is `php-fpm:7.2-alpine-composer`, The image size is 80MB

**If you need to install other php extensions, modify `.docker/php/Dockerfile` and execute `docker-compose build` to rebuild the image**

### Execute `php` and `composer` in host
Provide two shell files of `.docker/bin/php` and `.docker/bin/composer`\
When the php container started, the `php` and `composer` commands in the container can be executed separately in the host

**NOTE：\
When calling these two shells on the host, you can operate on files or folders in the host\
But only in the path where the `docker-compose.yml` file is located, and the path below it\
Unable to operate the object in the upper path, an error message that the file cannot be opened will be displayed**

It is recommended to add these two lines at the end of the `~/.bashrc` file

    set +h;
    export PATH=../.docker/bin:.docker/bin:./vendor/bin:$PATH

Then you can just execute `php` and `composer` in host to call these two shell\
You can also execute related commands placed in vendor/bin after installing the package by composer, such as phpunit

### Also for Laravel
If you modify the `~/.bashrc` file according to the above suggestions, the following operations can quickly create a new Laravel Project/Website
```bash
$ git clone https://github.com/jazchen/php-web-dev-docker.git myweb
$ cd myweb
# Here, we install Laravel into myLaravel folder
# Open the .env file and find DOCKER_NGINX_WEBROOT_DIR=webroot
# Modify to DOCKER_NGINX_WEBROOT_DIR=myLaravel/public
$ docker-compose up -d
$ composer create-project laravel/laravel myLaravel --prefer-dist
$ cd myLaravel
$ phpunit
```
You should see the result of phpunit passing the test successfully\
Open the browser and enter the URL http://127.0.0.1 to see the initial screen of Laravel\
Don't forget to open the `.conf/nginx.conf.template` file to add the settings required by Laravel, and then run `docker-compose restart nginx`

Execute `git init` in the myLaravel folder to start version control of the development project


## MariaDB

The database file is placed in the `mysql_data` folder under the project by default, you can modify the `.env` file to adjust the file location by yourself.

When connecting to MariaDB in PHP, the name of the connected server is **`mariadb`**\
The following is an example of PDO on the official PHP website to make adjustments
```php
<?php
/* Connect to a MySQL database using driver invocation */
$dsn = 'mysql:dbname=testdb;host=mariadb';
$user = 'root';
$password = 'DB_PassWord';

try {
    $dbh = new PDO($dsn, $user, $password);
} catch (PDOException $e) {
    echo 'Connection failed: ' . $e->getMessage();
}

```

### phpMyAdmin Management tools
The default phpMyAdmin URL http://127.0.0.1/phpMyAdmin \
Set the URI of phpMyAdmin by **`DOCKER_PHPMYADMIN_URI=/phpMyAdmin`** in the `.env` file\
If it is modified to `DOCKER_PHPMYADMIN_URI=/pma`, the phpMyAdmin will be open by http://127.0.0.1/pma

### You can use other mysql management tools
The development environment maps the 3306 port of the mariadb container to the 3306 port of the host. So you can alos use other mysql clients, for example

```bash
D:\mysql-8.0.23-winx64\bin> mysql -u root -p
Enter password:
```

## Others
In `docker-compose.yml`, use `container_name` to specify a fixed name for each container, so that a unified pattern can be followed in this development environment. For example, when php connects to MariaDB, the server name uses mariadb\
But the limitation is that only one **development environment** can be started as a container at the same time, regardless of whether the container is start or stop

If there are multiple development environment requirements, the following are the precautions
- To switch to another development environment, you must first execute the `docker-compose down` command to delete all containers in that environment, otherwise the container in another development environment will fail to start because of the same container name\
**The data developed by yourself is on the host, there will be no data loss problem**
- If multiple environments use the same PHP version, but have different requirements for php extensions. You can specify the string appended to the end of the image tag by the `DOCKER_PHP_IMAGE_POSTFIX` in the `.env` file
