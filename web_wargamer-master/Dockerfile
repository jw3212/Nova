FROM ubuntu:20.04
WORKDIR /var/www/html

# MariaDB와 Apache 패키지 설치
RUN apt-get update && apt-get install -y \
    apache2 \
    php \
    php-cli \
    php-fpm \
    php-mysql \
    php-json \
    php-opcache \
    php-xml \
    mariadb-server \
    mariadb-client \
    procps \
    && apt-get clean

# Apache 모듈 설정
RUN a2enmod proxy
RUN a2enmod proxy_fcgi

# MariaDB 권한 설정
RUN chown -R mysql:mysql /var/lib/mysql
RUN chmod -R 755 /var/lib/mysql

# Apache 설정
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf && \
    echo "ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/var/www/html/$1" >> /etc/apache2/apache2.conf

# PHP 설정
RUN sed -i 's/;date.timezone =/date.timezone = Asia\/Seoul/g' /etc/php/7.4/cli/php.ini && \
    mkdir -p /run/php && \
    sed -i 's/listen = \/run\/php\/php7.4-fpm.sock/listen = 127.0.0.1:9000/g' /etc/php/7.4/fpm/pool.d/www.conf
