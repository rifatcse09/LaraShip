#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
FROM php:8.2-fpm-alpine

LABEL authors="Rifat"

ARG SERVER_ENVIRONMENT

RUN set -ex \
  && apk --no-cache add \
    postgresql-dev

# Add Repositories
RUN rm -f /etc/apk/repositories &&\
    echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/main" >> /etc/apk/repositories && \
    echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/community" >> /etc/apk/repositories

# fix work iconv library with alpine
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ --allow-untrusted gnu-libiconv

ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# Add Build Dependencies
RUN apk add --no-cache \
    libzip-dev \
    libwebp-dev \
    icu-dev \
    libjpeg-turbo-dev \
    libjpeg-turbo \
    libpng-dev \
    libxpm-dev \
    freetype-dev \
    zlib-dev \
    curl-dev \
    icu-dev \
    g++ \
    npm \
    zip


RUN apk --no-cache add php82-pear \
php82-dev gcc \
musl-dev \
php82-pecl-imagick \
make \
imagemagick \
imagemagick-dev

# Configure & Install Extension
RUN docker-php-ext-install -j "$(nproc)" \
    curl \
    pdo \
    pdo_pgsql \
    pgsql \
    bcmath \
    zip

# Install Redis
RUN pecl install redis && docker-php-ext-enable redis

RUN docker-php-ext-configure intl && \
  docker-php-ext-install intl &&\
  docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql

# Install Imagick extension
RUN pecl install imagick && docker-php-ext-enable imagemagick
  
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp && \
    docker-php-ext-install -j$(nproc) gd

# Configure and enable OPCache extension
RUN docker-php-ext-configure opcache --enable-opcache
RUN docker-php-ext-install opcache

# Custom php.ini config
COPY .docker/app/opcache.ini $PHP_INI_DIR/conf.d/
COPY .docker/app/php.ini $PHP_INI_DIR/conf.d/

# Add Composer lastest version
RUN curl -sS https://getcomposer.org/installer | \
php -- --install-dir=/usr/bin/ --filename=composer

COPY .docker/app/init.sh /user/bin
RUN chmod +x /user/bin/init.sh

# Setup Working Dir
WORKDIR /var/www/html

# RUN mkdir -p storage/logs
# RUN touch storage/logs/worker.log

ENTRYPOINT ["/user/bin/startx.sh"]