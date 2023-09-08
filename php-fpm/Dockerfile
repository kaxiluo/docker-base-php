ARG X_PHP_VERSION
ARG X_DEBIAN_RELEASE=buster
FROM php:${X_PHP_VERSION}-fpm-${X_DEBIAN_RELEASE}

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

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN set -xe \
    && apt-get update -y \
    && pecl channel-update pecl.php.net \
    #&& apt-get install -y aptitude \
    && apt-get install -y --no-install-recommends \
      apt-utils  \
      telnet net-tools procps iputils-ping \
      vim git wget curl

RUN set -eux \
    &&  apt-get update -y \
    && apt-get install -y --no-install-recommends \
            autoconf \
            # zip
            libzip-dev zip unzip \
            libmemcached-dev libz-dev libpq-dev \
            libjpeg-dev libpng-dev  libfreetype6-dev libwebp-dev \
            libssl-dev libmcrypt-dev  libonig-dev \
            # xsl
            libxslt-dev

# only use
# composer-2.2.x-for-php5.3-7.1.phar
COPY src/* /tmp/

#######################
# PHP EXT
#######################

# pdo_mysql mysqli exif zip bcmath xsl intl soap opcache sockets pcntl
RUN set -ex \
    && cd /tmp \
    && docker-php-ext-install pdo_mysql mysqli exif zip bcmath xsl intl soap opcache sockets pcntl

RUN docker-php-source extract

# GD
RUN if [ ${X_PHP_VERSION} = "7.4" ] || [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
        docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-webp \
            --with-freetype; \
    else \
        docker-php-ext-configure gd \
            --with-jpeg-dir=/usr/lib \
            --with-webp-dir=/usr/lib \
            --with-png-dir=/usr/lib \
            --with-freetype-dir=/usr/include/freetype2; \
    fi; \
    docker-php-ext-install gd

## redis 
RUN if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install -o -f redis-4.3.0; \
    else \
      pecl install -o -f redis; \
    fi \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis
# igbinary
RUN pecl install igbinary \
   && docker-php-ext-enable igbinary

# AMQP
RUN set -eux; \ 
    apt-get -yqq install librabbitmq-dev; \
    if [ ${X_PHP_VERSION} = "7.3" ] \
    || [ ${X_PHP_VERSION} = "7.2" ] \
    || [ ${X_PHP_VERSION} = "7.1" ] \
    || [ ${X_PHP_VERSION} = "7.0" ] \
    || [ ${X_PHP_VERSION} = "5.6" ]; then \
    pecl install amqp-1.11.0; \
    else \
    pecl install amqp; \
    fi; \
    docker-php-ext-enable amqp; \
    php -m | grep -oiE '^amqp$';  

# mongodb
RUN if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install mongo; \
      docker-php-ext-enable mongo; \
      php -m | grep -oiE '^mongo$'; \
    else \
      if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && { [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ] || [ $(php -r "echo PHP_MINOR_VERSION;") = "1" ] ;}; then \
        pecl install mongodb-1.9.2; \
      else \
        pecl install mongodb; \
      fi; \
      docker-php-ext-enable mongodb; \
      php -m | grep -oiE '^mongodb$'; \
    fi;

# imap    
RUN apt-get install -yqq libc-client-dev libkrb5-dev && \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-install imap
# mailparse
RUN if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      echo 'todo'; \
    else \
      rm -rf /tmp/pear; \
      if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && { [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ] || [ $(php -r "echo PHP_MINOR_VERSION;") = "1" ] || [ $(php -r "echo PHP_MINOR_VERSION;") = "2" ];}; then \
        pecl install mailparse-3.1.3; \
      else \
        pecl install -o -f mailparse; \
      fi; \
      docker-php-ext-enable mailparse; \
    fi;

# Composer
RUN set -ex && \
    cd /tmp && \
    #&& wget https://mirrors.aliyun.com/composer/composer.phar \
    #&& wget https://getcomposer.org/download/1.9.0/composer.phar \
    if [ ${X_PHP_VERSION} = "7.1" ]; then \
        mv composer-2.2.x-for-php5.3-7.1.phar /usr/local/bin/composer; \
    else \
        wget -O composer-latest.phar https://mirrors.aliyun.com/composer/composer.phar && \
        mv composer-latest.phar /usr/local/bin/composer; \
    fi && \
    chmod u+x /usr/local/bin/composer && \
    composer config -g repo.packagist composer https://mirrors.aliyun.com/composer && \
    echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> ~/.bashrc

# Check
RUN set -ex \
    && cd /tmp \
    && php -v \
    && php -m \
    && echo -e "PHP${PHP_VERSION}-FPM Build Completed!"

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog && \
    docker-php-source delete

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000