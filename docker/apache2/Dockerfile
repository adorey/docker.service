FROM debian:buster-slim

ARG FCGI_FILE
ARG APACHE_USER
ARG APACHE_GROUP
ARG SERVICE_PATH
ARG SERVICE_HOST
ARG SERVICE_NAME
ARG APP_MAINTAINER_EMAIL

# Install apache and supplimentary programs.gettext-base allow us to use envsubst to
# replace environment in files, curl is for debugging the container.
RUN apt-get update \
    && apt-get -y upgrade \
    && DEBIAN_FRONTEND=noninteractive apt-get -y install \
    apache2 apache2-utils curl gettext-base

# enable / disable mod configuration for php-fpm
RUN a2enmod proxy && a2enmod proxy_fcgi && a2dismod mpm_event && a2enmod mpm_worker

# Enable apache mods.
RUN a2enmod rewrite

# Manually set up the apache environment variables
ENV APACHE_RUN_USER ${APACHE_USER}
ENV APACHE_RUN_GROUP ${APACHE_GROUP}

ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

ENV FCGI_FILE ${FCGI_FILE}
ENV SERVICE_PATH ${SERVICE_PATH}
ENV SERVICE_HOST ${SERVICE_HOST}
ENV SERVICE_NAME ${SERVICE_NAME}
ENV APP_MAINTAINER_EMAIL ${APP_MAINTAINER_EMAIL}

# Expose apache.
EXPOSE 80

# Update the default apache site with the config we created.
ADD sites/default.template /etc/apache2/sites-available/default.template

# create conf from template + environment and enable it
RUN envsubst < /etc/apache2/sites-available/default.template > /etc/apache2/sites-available/${SERVICE_NAME}.conf \
    && a2ensite ${SERVICE_NAME}.conf

# restarting apache to apply our configuration
RUN apache2ctl restart

# By default start up apache in the foreground, override with /bin/bash for interative.
CMD /usr/sbin/apache2ctl -D FOREGROUND