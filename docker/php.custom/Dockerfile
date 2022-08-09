ARG PHP_IMG_VERSION

FROM php:${PHP_IMG_VERSION}

ARG TIMEZONE
ARG NEED_REDIS
ARG NEED_XDEBUG
ARG SERVICE_NAME
ARG NEED_MEMCACHE

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
    git openssl \
    unzip p7zip p7zip-full zip libzip-dev \
    build-essential libmemcached-dev zlib1g-dev libmagickwand-dev xz-utils sharutils \
    gcc make autoconf libc-dev pkg-config libssh2-1-dev \
    libc-client-dev libkrb5-dev

# Activating php etension
# You can see available extension in container by typing "docker-php-ext-install" in the container shell
RUN docker-php-ext-install pdo pdo_mysql
RUN docker-php-ext-install pcntl
RUN docker-php-ext-install opcache

# install extre php extension (exemple with xdebug and his configuration)
RUN if [ ${NEED_XDEBUG} = true ] ; then pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "display_startup_errors = on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "display_errors = on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.max_nesting_level=256" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini ; fi

#install php-memcache
RUN if [ ${NEED_MEMCACHE} = true ] ; then git clone https://github.com/websupport-sk/pecl-memcache.git ; fi
RUN if [ ${NEED_MEMCACHE} = true ] ; then cd pecl-memcache \
    && phpize \
    && ./configure \
    && make \
    && make install ; fi
RUN if [ ${NEED_MEMCACHE} = true ] ; then docker-php-ext-enable memcache ; fi

RUN if [ ${NEED_MEMCACHE} = true ] ; then pecl install memcached \
    && docker-php-ext-enable memcached ; fi

# install apcu extension
#RUN pecl install apcu_bc \
#    && docker-php-ext-enable apcu \
#    && echo "extension=apc.so" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini

RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && docker-php-ext-install gd \
    && docker-php-ext-install sockets

RUN docker-php-ext-install zip xml
RUN docker-php-ext-configure intl && docker-php-ext-install intl

# copying our www.conf to the right place
#COPY ./www.conf /usr/local/etc/php-fpm.d/www.conf

# Use the default development configuration
RUN mv "${PHP_INI_DIR}/php.ini-development" "${PHP_INI_DIR}/php.ini"

# Override php ini with custom settings
COPY general.ini ${PHP_INI_DIR}/conf.d/

# Redis
RUN if [ ${NEED_REDIS} = true ] ; then rm -rf phpredis ; fi
RUN if [ ${NEED_REDIS} = true ] ; then git clone https://github.com/phpredis/phpredis ; fi
RUN if [ ${NEED_REDIS} = true ] ; then cd phpredis \
    && phpize \
    && ./configure \
    && make \
    && make install ; fi
RUN if [ ${NEED_REDIS} = true ] ; then docker-php-ext-enable redis ; fi

# application directory path (inside container)
WORKDIR /var/www/${SERVICE_NAME}