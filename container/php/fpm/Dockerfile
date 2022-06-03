FROM php:7.2-fpm as php_base

LABEL maintainer "Oleksii Temchenko <otemchenko@notesco.com>"

#fix docker-php-ext-install bug
RUN sed -i 's/docker-php-\(ext-$ext.ini\)/\1/' /usr/local/bin/docker-php-ext-install

RUN echo "Acquire::Check-Valid-Until \"false\";\nAcquire::Check-Date \"false\";" | cat > /etc/apt/apt.conf.d/10no--check-valid-until

RUN apt-get update && apt-get install -y zlib1g-dev


# Install other needed extensions
RUN buildDeps=" \
		libfreetype6-dev \
		libjpeg-dev \
		libldap2-dev \
		libmcrypt-dev \
		zlib1g-dev \
        libicu-dev \
        libsqlite3-dev \
        libpq-dev \
        postgresql-server-dev-all \
        libpq5 \
        librdkafka-dev \
        libmemcached-dev \
	"; \
	set -x \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu \
	&& docker-php-ext-install ldap \
	&& docker-php-ext-install mbstring \
	&& docker-php-ext-install zip \
    && docker-php-ext-install pdo \
    && docker-php-ext-install pgsql pdo_pgsql \
    && docker-php-ext-install mysqli pdo_mysql  \
    && docker-php-ext-enable pdo_mysql
RUN pecl install rdkafka \
    && docker-php-ext-enable rdkafka
RUN apt-get update && apt-get install -y wget \
    && wget "https://pecl.php.net/get/memcache-4.0.5.2.tgz" \
    && tar zxvf memcache-4.0.5.2.tgz \
    && cd memcache-4.0.5.2 && phpize && ./configure && make && make install \
    && echo "extension=memcache.so" > /usr/local/etc/php/conf.d/memcache.ini

RUN pecl install protobuf \
    && docker-php-ext-enable protobuf

RUN apt-get purge -y --auto-remove $buildDeps

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

# Setup timezone to Etc/UTC
RUN cat /usr/local/etc/php/php.ini-production | sed 's/^;\(date.timezone.*\)/\1 \"Etc\/UTC\"/' > /usr/local/etc/php/php.ini

# Disable cgi.fix_pathinfo in php.ini
RUN sed -i 's/;\(cgi\.fix_pathinfo=\)1/\10/' /usr/local/etc/php/php.ini

#ant

#RUN cd \
#    && curl -L -b "oraclelicense=a" https://builds.openlogic.com/downloadJDK/openlogic-openjdk-jre/8u262-b10/openlogic-openjdk-jre-8u262-b10-linux-x32.tar.gz -O \
#    && tar -xzf openlogic-openjdk-jre-8u262-b10-linux-x32.tar.gz \
#    && mv openlogic-openjdk-jre-8u262-b10-linux-32 /opt/openjdk \
#    && rm openlogic-openjdk-jre-8u262-b10-linux-x32.tar.gz
#
#ENV JAVA_HOME /opt/openjdk
#
#ENV ANT_VERSION 1.10.12
#RUN cd
#RUN wget -q https://downloads.apache.org/ant/binaries/apache-ant-${ANT_VERSION}-bin.tar.gz
#RUN tar -xzf apache-ant-${ANT_VERSION}-bin.tar.gz
#RUN mv apache-ant-${ANT_VERSION} /opt/ant
#RUN rm apache-ant-${ANT_VERSION}-bin.tar.gz
#
#ENV ANT_HOME /opt/ant
#ENV PATH ${PATH}:/opt/ant/bin

##############################################

RUN mkdir /.composer \
    && chmod 0777 /.composer

USER 1000
RUN composer global require phpunit/phpunit "^5.7.12"
USER root
RUN ln -s /.composer/vendor/phpunit/phpunit/phpunit /usr/local/bin/phpunit

USER 1000
WORKDIR "/var/www"
EXPOSE 9000
ENTRYPOINT php-fpm --nodaemonize

#----------------------------------------------------------------------------------------------------------------------

FROM php_base AS php_xdebug
USER root

# Install and enable xDebug
RUN pecl install xdebug

# Prevent error in nginx error.log
RUN touch /var/log/xdebug_remote.log
RUN chmod 777 /var/log/xdebug_remote.log

USER 1000
