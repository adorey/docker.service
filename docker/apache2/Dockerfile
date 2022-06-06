FROM debian:buster-slim

MAINTAINER Aurélien DOREY <aurelien.dorey@gmail.com>

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
ENV APACHE_RUN_USER ubuntu
ENV APACHE_RUN_GROUP ubuntu
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

# Expose apache.
EXPOSE 80

# Update the default apache site with the config we created.
ADD sites-available/default.template /etc/apache2/sites-available/default.template

# create conf from template + environment and enable it
RUN /bin/bash -c 'envsubst < /etc/apache2/sites-available/default.template > /etc/apache2/sites-available/default.conf' \
    && a2ensite default.conf

# By default start up apache in the foreground, override with /bin/bash for interative.
CMD /usr/sbin/apache2ctl -D FOREGROUND