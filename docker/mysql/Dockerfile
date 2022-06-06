ARG MYSQL_VERSION

FROM mysql:${MYSQL_VERSION}

MAINTAINER DOREY Aurélien <aurelien.dorey@gmail.com>

ARG MYSQL_USER
ARG MYSQL_PASSWORD
ARG MYSQL_DATABASE
ARG MYSQL_ROOT_PASSWORD

#setting configuration file in the container
RUN rm /etc/mysql/conf.d/*
ADD conf.d/* /etc/mysql/conf.d/
RUN chmod 0444 /etc/mysql/conf.d/*

# setup mysql environment
ENV MYSQL_USER ${MYSQL_USER}
ENV MYSQL_PASSWORD ${MYSQL_PASSWORD}
ENV MYSQL_DATABASE ${MYSQL_DATABASE}
ENV MYSQL_ROOT_PASSWORD ${MYSQL_ROOT_PASSWORD}