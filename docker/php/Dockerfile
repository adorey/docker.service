ARG TIMEZONE
ARG PHP_IMG_VERSION

FROM php:${PHP_IMG_VERSION}

MAINTAINER DOREY Aurélien <aurelien.dorey@gmail.com>

# Install Composer
RUN curl -sS https://getcomposer.org/installer \
    | php -- --install-dir=/usr/local/bin --filename=composer \
    && composer --version

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo ${TIMEZONE} > /etc/timezone \
    && printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini \
    && "date"

# Adding container tools / core php extension
RUN apt-get update \
    && apt-get install -y \
    openssl \
    unzip p7zip p7zip-full zip libzip-dev \
    build-essential libmemcached-dev zlib1g-dev libmagickwand-dev xz-utils sharutils \
    gcc make autoconf libc-dev pkg-config libssh2-1-dev \
    libc-client-dev libkrb5-dev \
    sudo git


# Activating php etension
# You can see available extension in container by typing "docker-php-ext-install" in the container shell
RUN docker-php-ext-install pdo pdo_mysql
RUN docker-php-ext-install pcntl
RUN docker-php-ext-install opcache

# install extre php extension (exemple with xdebug and his configuration)
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.max_nesting_level=256" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

#install php-memcache
RUN git clone https://github.com/websupport-sk/pecl-memcache.git
RUN cd pecl-memcache \
    && phpize \
    && ./configure \
    && make \
    && make install
RUN docker-php-ext-enable memcache

RUN pecl install memcached \
    && docker-php-ext-enable memcached

# install imagick extension
RUN pecl install imagick \
    && docker-php-ext-enable imagick

# install apcu extension
RUN pecl install apcu_bc \
    && docker-php-ext-enable apcu \
    && echo "extension=apc.so" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini

RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && docker-php-ext-install gd \
    && docker-php-ext-install sockets

#RUN pecl install amqp-1.9.4 && docker-php-ext-enable amqp
RUN docker-php-ext-install zip xml
RUN docker-php-ext-configure intl && docker-php-ext-install intl

# copying our www.conf to the right place
#COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf

# Use the default development configuration
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

# Override php ini with custom settings
COPY general.ini $PHP_INI_DIR/conf.d/

# Redis
RUN rm -rf phpredis 
RUN git clone https://github.com/phpredis/phpredis
RUN cd phpredis \
    && phpize \
    && ./configure \
    && make \
    && make install
RUN docker-php-ext-enable redis

# application directory pas (inside container)
WORKDIR /var/www/consoneo