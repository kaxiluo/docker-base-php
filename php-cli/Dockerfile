ARG X_PHP_VERSION
ARG X_DEBIAN_RELEASE=buster
FROM php:${X_PHP_VERSION}-cli-${X_DEBIAN_RELEASE}

LABEL maintainer="Linyiyong <linyiyongcn@163.com>"

ARG X_PHP_VERSION
ARG X_DEBIAN_RELEASE=buster

ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ] && ["${X_DEBIAN_RELEASE}" = "buster"]; then \
        cd /tmp \
        && cp /etc/apt/sources.list /etc/apt/sources.list.bak \
        && { \
            echo "deb http://mirrors.aliyun.com/debian/ buster main non-free contrib"; \
            echo "deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib"; \
            echo "deb http://mirrors.aliyun.com/debian-security buster/updates main"; \
            echo "deb-src http://mirrors.aliyun.com/debian-security buster/updates main"; \
            echo "deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib"; \
            echo "deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib"; \
            echo "deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib"; \
            echo "deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib"; \
        } > /etc/apt/sources.list \
    ;elif [ ${CHANGE_SOURCE} = true ] && ["${X_DEBIAN_RELEASE}" = "bullseye"]; then \
        cd /tmp \
        && cp /etc/apt/sources.list /etc/apt/sources.list.bak \
        && { \
            echo "deb http://mirrors.aliyun.com/debian/ bullseye main non-free contrib"; \
            echo "deb-src http://mirrors.aliyun.com/debian/ bullseye main non-free contrib"; \
            echo "deb http://mirrors.aliyun.com/debian-security/ bullseye-security main"; \
            echo "deb-src http://mirrors.aliyun.com/debian-security/ bullseye-security main"; \
            echo "deb http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib"; \
            echo "deb-src http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib"; \
            echo "deb http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib"; \
            echo "deb-src http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib"; \
        } > /etc/apt/sources.list \
    ;elif [ ${CHANGE_SOURCE} = true ] ; then \
        sed -i -E 's/(deb|security).debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
    ;fi

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=Asia/Shanghai

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN set -xe \
    && apt-get update -y \
    && pecl channel-update pecl.php.net \
    && apt-get install -y \
      apt-utils telnet net-tools procps iputils-ping \
      vim git wget curl zsh

RUN set -eux \
    && apt-get install -y --no-install-recommends \
            autoconf libzip-dev zip unzip \
            libmemcached-dev libz-dev libpq-dev \
            libjpeg-dev libpng-dev  libfreetype6-dev libwebp-dev \
            libssl-dev libmcrypt-dev  libonig-dev \
            libc-client-dev libkrb5-dev \
            libxslt-dev librabbitmq-dev \
            libcurl4-openssl-dev \
            libnghttp2-dev

COPY src/* /tmp/

# php ext
RUN set -ex \
    && cd /tmp \
    # pdo_mysql mysqli ......
    && docker-php-ext-install pdo_mysql mysqli exif zip sockets pcntl bcmath xsl \
    # imap
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    # source extract
    && docker-php-source extract \
    # redis
    && tar -zxvf redis-5.3.2.tgz \ 
    && mv redis-5.3.2 /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
    # igbinary
    && tar -zxvf igbinary-3.1.6.tgz \ 
    && mv igbinary-3.1.6 /usr/src/php/ext/igbinary \
    && docker-php-ext-install igbinary \
    # amqp  
    && tar -zxvf amqp-1.10.2.tgz \
    && mv amqp-1.10.2 /usr/src/php/ext/amqp \
    && docker-php-ext-install amqp \
    # mailparse  
    && tar -zxvf mailparse-3.1.1.tgz \
    && mv mailparse-3.1.1 /usr/src/php/ext/mailparse \
    && docker-php-ext-install mailparse

# GD
ARG INSTALL_GD=false
RUN if [ ${INSTALL_GD} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "4" ]; then \
      docker-php-ext-configure gd --with-freetype --with-jpeg --with-png; \
    else \
      docker-php-ext-configure gd --with-freetype-dir=/usr/lib/ --with-jpeg-dir=/usr/lib/ --with-png-dir=/usr/lib/; \
    fi; \
    docker-php-ext-install gd \
;fi

## swoole
ARG INSTALL_SWOOLE=false
ARG SWOOLE_VERSION=4.6.3
RUN if [ ${INSTALL_SWOOLE} = true ]; then \
    cd /tmp && \
    tar -zxvf swoole-${SWOOLE_VERSION}.tgz && \
    mv swoole-${SWOOLE_VERSION} /usr/src/php/ext/swoole \
    && docker-php-ext-configure swoole --enable-openssl --enable-http2 --enable-mysqlnd --enable-swoole-curl --enable-swoole-json \
    && docker-php-ext-install swoole \
;fi

## xdebug
ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \ 
    cd /tmp \
    && tar -zxvf xdebug-2.9.8.tgz \
    && mv xdebug-2.9.8 /usr/src/php/ext/xdebug \
    && docker-php-ext-install xdebug \
;fi

# Composer
RUN set -ex \
    && cd /tmp \
    && wget -O composer-latest.phar https://mirrors.aliyun.com/composer/composer.phar \
    && mv composer-latest.phar /usr/local/bin/composer \
    && chmod u+x /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer \
    && echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> ~/.bashrc

# Check
RUN set -ex \
    && cd /tmp \
    && php -v \
    && php -m \
    && echo -e "PHP${PHP_VERSION}-CLI Build Completed!"

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog && \
    docker-php-source delete

WORKDIR /var/www
