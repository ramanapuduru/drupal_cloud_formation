FROM php:5.6-apache
MAINTAINER Maintained by IT Trailblazers
ENV MYSQL_ROOT_PASSWORD some_password
ENV DRUPAL_DB_NAME drupal
ENV DRUPAL_DB_USER drupal
ENV DRUPAL_DB_PASS drupal
ENV DRUPAL_DB_HOST 172.17.0.2
ENV MYSQL_MAJOR 5.72
ENV MYSQL_VERSION 5.7.17-1debian8
ENV DRUSH_VERSION 7.4.0 
RUN a2enmod rewrite
# install the PHP extensions we need
RUN set -ex \
&& buildDeps=' \
libjpeg62-turbo-dev \
libpng12-dev \
libpq-dev \
' \
&& apt-get update && apt-get install -y --no-install-recommends $buildDeps \
&& rm -rf /var/lib/apt/lists/* \
&& docker-php-ext-configure gd \
--with-jpeg-dir=/usr \
--with-png-dir=/usr \
&& docker-php-ext-install -j "$(nproc)" gd mbstring pdo pdo_mysql pdo_pgsql zip bcmath calendar ctype dba gd iconv json mbstring mysql mysqli opcache pdo pdo_mysql pdo_pgsql pgsql phar \
&& apt-mark manual \
libjpeg62-turbo \
libpq5 \
&& apt-get update && apt-get install -y git vim  \
&& ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime  \
&& apt-get purge -y --auto-remove $buildDeps && apt-get -y install nfs-common nfs-client \
&& rm -rf /var/lib/apt/lists/* 
WORKDIR /install_dir
# https://www.drupal.org/node/3060/release
ENV DRUPAL_VERSION 7.54
ENV DRUPAL_MD5 3068cbe488075ae166e23ea6cd29cf0f

RUN curl -fSL "https://ftp.drupal.org/files/projects/drupal-${DRUPAL_VERSION}.tar.gz" -o drupal.tar.gz \
&& echo "${DRUPAL_MD5} *drupal.tar.gz" | md5sum -c - \
&& tar -xz --strip-components=1 -f drupal.tar.gz \
&& rm drupal.tar.gz 

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php && php -r "unlink('composer-setup.php');"
# Use this if you want the preferred DRUSH version..
#RUN cp composer.phar /usr/bin/composer  && composer global require drush/drush
#use this if you want specific versions --- Check default matrix compatibility 
#RUN cp composer.phar /usr/bin/composer  && composer global require drush/drush:"${DRUSH_VERSION}" 
#RUN cp composer.phar /usr/bin/composer  && composer global require drush/drush:"$DRUSH_VERSION" 
# The following has been added to install drush globally....
# So you shold be able to just run Drush directly instead of /root/.composer/vendor/bin/drush
RUN cp  composer.phar /usr/local/bin/composer && \
    ln -s /usr/local/bin/composer /usr/bin/composer && \
    git clone https://github.com/drush-ops/drush.git /usr/local/src/drush && \ 
    cd /usr/local/src/drush && \
    git checkout 7.4.0  && \
    ln -s /usr/local/src/drush/drush /usr/bin/drush && \
    composer install && drush --version
RUN usermod -u 495 www-data && \
    groupmod -g 495 www-data && \
    usermod -g www-data www-data 
RUN set -ex; \
# gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
export GNUPGHOME="$(mktemp -d)"; \
gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
gpg --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
rm -r "$GNUPGHOME"; \
apt-key list > /dev/null && apt-get clean
RUN mkdir -p /install_dir/sites/default
#    && ln -s /var/www/sites-enabled/ /etc/apache2/sites-enabled
ADD settings.php /install_dir/sites/default
ADD php.ini /usr/local/etc/php

EXPOSE 80 22
#Cleanup agt-get to reduce disk
ADD copyfile.sh /bin/copyfile.sh
ADD change_db.sh /bin/change_db.sh
#RUN /bin/change_db.sh $DRUPAL_DB_NAME $DRUPAL_DB_USER $DRUPAL_DB_PASS $DRUPAL_DB_HOST
RUN mkdir -p /etc/apache2/sites-enabled
#RUN  cp -r /install_dir/* /var/www/html && chown -R www-data:www-data /var/www
RUN  chown -R www-data:www-data /var/www
WORKDIR /var/www
CMD ["docker-php-entrypoint", "/usr/sbin/apache2ctl","-D FOREGROUND"]
