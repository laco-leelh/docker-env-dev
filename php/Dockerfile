# FROM php:7.2.6-fpm
FROM php:7.2

MAINTAINER laco

# Env
ENV SWOOLE_VERSION=4.5.1

# Setting env
RUN mkdir -p /usr/local/ext/php/conf.d

# DEBIAN_FRONTEND=noninteractive 避免交互式选项操作
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
	&& apt-get install -y \
	procps \
	wget \
	libfreetype6-dev \
	libjpeg62-turbo-dev \
	libmcrypt-dev \
	libxml2-dev \
	libpng-dev \
	openssl \
	libssl-dev \
	pkg-config \
	vim \
	# ---- imap begin ----
	libc-client2007e-dev \
	krb5-user \
	libkrb5-dev \
	# ---- imap end ----
	&& rm -r /var/lib/apt/lists/*

# Install PHP extensions
# redis 4.0.2
COPY ./ext/redis.tgz /ext/redis.tgz
# mongodb 1.4.3
COPY ./ext/mongodb.tgz /ext/mongodb.tgz
# mcrypt 1.0.1
COPY ./ext/mcrypt.tgz /ext/mcrypt.tgz

# 性能分析
COPY ./ext/swoole_loader.so /usr/local/lib/php/extensions/no-debug-non-zts-20170718/swoole_loader.so

RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
	&& docker-php-ext-install -j$(nproc) gd \
		zip \
		soap \
		mbstring \
		bcmath \
		pdo_mysql \
		mysqli \
		sockets \
		opcache

	# ---- imap begin ----
RUN pecl install xlswriter
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl && docker-php-ext-install imap
	# ---- imap end ----

RUN pecl install /ext/redis.tgz && echo "extension=redis.so" > /usr/local/ext/php/conf.d/redis.ini
RUN pecl install /ext/mongodb.tgz && echo "extension=mongodb.so" > /usr/local/ext/php/conf.d/mongodb.ini
RUN pecl install /ext/mcrypt.tgz && echo "extension=mcrypt.so" > /usr/local/ext/php/conf.d/mcrypt.ini
RUN echo "extension=swoole_loader.so" > /usr/local/ext/php/conf.d/swoole_loader.ini

# PHP config
ADD ./php.ini		/usr/local/etc/php/php.ini
ADD ./php-fpm.conf 	/usr/local/etc/php-fpm.conf

# Install Composer
# RUN curl -sS https://getcomposer.org/installer | php \
COPY ./ext/composer.phar /ext/composer.phar
RUN mv /ext/composer.phar /usr/local/bin/composer \
	&& composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
	&& composer self-update --clean-backups

# Install Swoole
RUN wget https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz -O swoole.tar.gz \
	&& mkdir -p swoole \
	&& tar -xf swoole.tar.gz -C swoole --strip-components=1 \
	&& rm swoole.tar.gz \
	&& ( \
		cd swoole \
		&& phpize \
		&& ./configure --enable-mysqlnd --enable-sockets --enable-openssl --enable-http2 \
		&& make -j$(nproc) \
		&& make install \ 
	) \
	&& rm -r swoole \ 
	&& docker-php-ext-enable swoole

# docker-php-ext-configure 相当于 ./configure
# docker-php-ext-install 相当于 pecl

WORKDIR /opt

RUN usermod -u 1000 www-data

EXPOSE 9000
VOLUME ["/opt"]

