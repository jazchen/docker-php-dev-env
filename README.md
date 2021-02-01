# 透過 Docker Compose 建立 PHP 網站開發環境

- 環境由 Nginx + PHP-FPM + MariaDB + phpMyAdmin 組成
- 使用 Docker 官方映像檔，或由官方映像檔為基礎即時生成自訂映像檔
- 映像檔選擇都以 alpine 為優先，盡可能縮小容器的大小
- 使用 .env 檔案設定相關參數，PHP 和 MariaDB 可自行選擇環境所需版本
- 提供 PHP(php.ini) 及 MariaDB(my.cnf) 設定檔，可自行調整設定
- 容器僅提供開發環境，所有自行開發的內容都儲存在 host 上
- 在 host 上提供使用 php 和 composer 指令


## README.md
en [English](README.en.md)


## 需求

- git
- docker & docker-compose


## 快速開始

```bash
    $ git clone https://github.com/jazchen/php-web-dev-docker.git myweb
    $ cd myweb
    $ docker-compose up -d
```
等映像檔下載和建立完成後就會跟著啟動容器\
若 host 的 80、443、3306 這三個 port 沒有被其它程序佔用，容器會順利啟動\
接著打開瀏覽器，輸入網址 http://127.0.0.1 就可以看到 phpinfo() 的畫面


## 環境預設值

- PHP 使用 **7.2** 版，MariaDB  使用 **10.3** 版
- 使用 host 的以下 port
	- **HTTP 80**
	- **HTTPS 443**
	- **MariaDB 3306**
- 預設的網站根目錄 **webroot**
- 預設的 MariaDB 資料庫檔案目錄 **mysql-data**


## 設定 .env 檔案

- **nginx 需要使用 1.19 或以上的版本** (1.19 才支援 template 套用環境變數)

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

## 其他設定
相關設定檔案放在 `.conf` 資料夾
- **nginx/ssl/** - 對應 nginx 容器內的 `/etc/nginx/ssl` 路徑
- **nginx/default.conf.template** - nginx 設定檔樣板
- **my.cnf** - MariaDB 的初始設定檔
- **php.ini** - PHP 的初始設定檔 (php.ini-development)
- **php-xdebug.ini** - php xdebug 設定檔，預設使用 3.x 版本


## PHP

在 `.env` 檔案中以 **DOCKER_PHP_VERSION** 指定 PHP 版本，預設使用 7.2\
以 `php:7.2-fpm-alpine` 為基礎建立一個新的映像檔，安裝 composer 及以下 php extension
- mysqli
- pdo_mysql
- bcmath
- gettext
- yaml
- xdebug

新的映像檔 tag 為 `php-fpm:7.2-alpine-composer` 檔案不大只有80MB

**若需要安裝其他 php extension，修改 `.docker/php/Dockerfile` 後執行 `docker-compose build` 重新建立映像檔**

### 在 host 中執行 `php` 和 `composer`
提供 `.docker/bin/php` 和 `.docker/bin/composer` 2 個 shell 檔案\
當php容器啟動時可在 host 中分別執行容器內的 php 和 composer 指令

**注意：\
在 host 中呼叫這兩個 shell 時，可對 host 中的檔案或資料夾進行操作\
但僅限 `docker-compose.yml` 檔案所在路徑內，以及更下層的路徑\
無法對上層路徑的對象進行操作，會顯示無法開啟檔案的錯誤訊息**

建議可以在 `~/.bashrc` 檔案最後加上這兩行

    set +h;
    export PATH=../.docker/bin:.docker/bin:./vendor/bin:$PATH

這樣方便直接以 `php` 和 `composer` 這2個指令直接執行對應的 shell 檔案\
也可以執行由 composer 安裝套件後放在 vendor/bin 中的相關命令，例如 phpunit

### 也適用於 Laravel 開發
如果有依照上面的建議修改 `~/.bashrc` 檔案，下面的操作可以快速建立新的 laravel 專案/網站
```bash
$ git clone https://github.com/jazchen/php-web-dev-docker.git myweb
$ cd myweb
# 假設 laravel 安裝的資料夾名稱是 myLaravel
# 開起 .env 檔案，找到 DOCKER_NGINX_WEBROOT_DIR=webroot
# 修改為 DOCKER_NGINX_WEBROOT_DIR=myLaravel/public
$ docker-compose up -d
$ composer create-project laravel/laravel myLaravel --prefer-dist
$ cd myLaravel
$ phpunit
```
沒有意外應該會看到 phpunit 的順利通過測試的結果\
開啟瀏覽器輸入網址 http://127.0.0.1 可以看到 Laravel 初始畫面\
別忘了開啟 `.conf/nginx.conf.template` 檔案加入 Laravel 需要的設定，再並執行 `docker-compose restart nginx`

在 myLaravel 資料夾中執行 `git init` 就可以開始對開發專案進行版本控制了


## MariaDB

資料庫檔案預設放在專案下的 `mysql_data` 資料夾內，可修改 `.env` 檔案自行調整檔案位置

在 PHP 中連線到 MariaDB 時，連接的伺服器名稱為 **`mariadb`**\
下面是以 PHP 官方網站的 PDO 的範例做調整
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

### phpMyAdmin 管理工具
預設的 phpMyAdmin 的網址是 http://127.0.0.1/phpMyAdmin \
透過 `.env` 檔案中 **`DOCKER_PHPMYADMIN_URI=/phpMyAdmin`** 設定 phpMyAdmin 的 URI\
若修改為 `DOCKER_PHPMYADMIN_URI=/pma` 則 phpMyAdmin 入口會是 http://127.0.0.1/pma

### 其他相容的 Client
開發環境將 mariadb 容器的 3306 port 對應到 host 的 3306 port
所以可以使用其他 mysql 相容的 client 端進行管理，例如

```bash
D:\mysql-8.0.23-winx64\bin> mysql -u root -p
Enter password:
```

## 其他說明
在 `docker-compose.yml` 中對各個容器都使用 `container_name` 指定固定名稱，方便在這個開發環境內有統一模式可以遵循。例如 php 連線到 MariaDB 時 server 統一使用 mariadb\
但帶來的限制是，同一時間只能有一個 **開發環境** 被啟動為容器，無論容器是 start 還是 stop

如果有多個開發環境的需求，以下為注意事項
- 要切換到另一個 開發環境 時，須先執行 `docker-compose down` 指令刪除該環境的所有容器，否則另一個開發環境的容器會因為 container name 相同而無法啟動\
**刪除容器只是刪除開發環境，自行開發的資料都會在 host 上，不會有資料遺失的問題**
- 若有多個環境使用相同的PHP版本，但對 php extension 有不同需求\
可以透過 `.env` 檔案中的 `DOCKER_PHP_IMAGE_POSTFIX` 指定附加在映像檔 tag 最後的字串\
避免切換不同開發環境時都要去重新 build 映像檔
