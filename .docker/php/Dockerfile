ARG  SOURCE_TAG
FROM php:$SOURCE_TAG

# 安裝 PHP Extension 依賴的函式庫
RUN apk add --no-cache \
# for gettext
	libintl \
# for yaml
	yaml \
# 安裝 編譯時依賴的函式庫/工具 (暫時性)
 && apk add --no-cache --virtual .build-deps \
	$PHPIZE_DEPS \
	autoconf \
	gettext-dev \
	yaml-dev \
# 用 docker 官方的工具安裝 extension
 && docker-php-ext-install -j$(nproc) \
	mysqli \
	pdo_mysql \
	bcmath \
	gettext \
# 更新PECL頻道
 && pecl channel-update pecl.php.net \
# PECL套件 安裝&&啟用
 && pecl install yaml && docker-php-ext-enable yaml \
 && pecl install xdebug && docker-php-ext-enable xdebug \
# 清除 編譯時依賴的函式庫/工具
 && apk del --purge .build-deps \
# 清除 php原始碼
 && docker-php-source delete \
# 安裝 composer
 && curl -sSLf https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# 允許 composer 以 root 身分執行
ENV COMPOSER_ALLOW_SUPERUSER=1
