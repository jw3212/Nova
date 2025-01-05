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

# MariaDB 초기 설정
RUN mysql_install_db --user=mysql --datadir=/var/lib/mysql

# MariaDB 설정
RUN echo "[mysqld]" >> /etc/mysql/mariadb.conf.d/50-server.cnf && \
    echo "port = 3306" >> /etc/mysql/mariadb.conf.d/50-server.cnf && \
    echo "bind-address = 0.0.0.0" >> /etc/mysql/mariadb.conf.d/50-server.cnf && \
    echo "default-authentication-plugin = mysql_native_password" >> /etc/mysql/mariadb.conf.d/50-server.cnf

# 초기화 스크립트 디렉토리 생성
RUN mkdir -p /docker-entrypoint-initdb.d

# DB 초기화 스크립트 생성
RUN printf '#!/bin/bash\n\
/usr/sbin/mariadbd --user=mysql &\n\
sleep 5\n\
mysql -e "CREATE DATABASE IF NOT EXISTS LED_WG;" && \
mysql -e "CREATE USER IF NOT EXISTS '\''root'\''@'\''localhost'\'' IDENTIFIED BY '\''1234'\''" && \
mysql -e "GRANT ALL PRIVILEGES ON LED_WG.* TO '\''root'\''@'\''localhost'\''" && \
mysql -e "FLUSH PRIVILEGES;" && \
mysql -e "CREATE TABLE users ( \
    u_id INT NOT NULL AUTO_INCREMENT, \
    nickname VARCHAR(50) NOT NULL, \
    username VARCHAR(50) NOT NULL, \
    password VARCHAR(50) NOT NULL, \
    email VARCHAR(50) NOT NULL, \
    user_role VARCHAR(50) NOT NULL, \
    PRIMARY KEY (u_id) \
);" && \
mysql -e "CREATE TABLE challenges_data ( \
    c_id INT NOT NULL AUTO_INCREMENT, \
    c_title VARCHAR(50) NOT NULL, \
    c_ssh VARCHAR(50), \
    c_web TINYINT(1) NOT NULL, \
    c_link VARCHAR(50), \
    c_hint TEXT, \
    c_point INT, \
    c_difficulty VARCHAR(50), \
    c_key VARCHAR(50), \
    c_text TEXT, \
    c_solves INT, \
    PRIMARY KEY (c_id) \
);" && \
mysql -e "CREATE TABLE user_data ( \
    d_id INT NOT NULL AUTO_INCREMENT, \
    d_uid INT, \
    d_cid INT, \
    d_time DATETIME, \
    PRIMARY KEY (d_id), \
    FOREIGN KEY (d_uid) REFERENCES users(u_id) ON DELETE CASCADE ON UPDATE CASCADE, \
    FOREIGN KEY (d_cid) REFERENCES challenges_data(c_id) ON DELETE CASCADE ON UPDATE CASCADE \
);" \n\
pkill mariadbd\n\
sleep 5\n' > /docker-entrypoint-initdb.d/init-db.sh && \
    chmod +x /docker-entrypoint-initdb.d/init-db.sh

# Node.js 및 npm 설치
RUN apt-get update && apt-get install -y nodejs npm && apt-get clean

# 웹 소스 파일 복사
COPY web/ /var/www/html/

# Node.js 의존성 설치
WORKDIR /var/www/html
RUN npm install

# 권한 설정
RUN chown -R www-data:www-data /var/www/html && \
    chmod -R 755 /var/www/html

# 시작 스크립트 생성
RUN printf '#!/bin/bash\n\
/docker-entrypoint-initdb.d/init-db.sh\n\
\n\
/usr/sbin/mariadbd --user=mysql &\n\
\n\
/usr/sbin/php-fpm7.4 &\n\
\n\
sleep 5\n\
\n\
/usr/sbin/apache2ctl -D FOREGROUND' > /start.sh && chmod +x /start.sh

# 실행 명령
CMD ["/start.sh"]
