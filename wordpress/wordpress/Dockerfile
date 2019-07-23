# =============================================================================
# wordpress:5-php7.3-apache
#
# WordPress 5.*, Apache 2.4, PHP 7.3
# Modification: Rsyslog/Fail2Ban setup
#
# =============================================================================
FROM wordpress:5-php7.3-apache
LABEL maintainer="Jacob Thompson <jacob.thompson@wiwasolvet.ca>"
ARG PROXY_IP
ARG BASE_URL

RUN apt-get update &&  \
    apt-get install -y \
    nano \
    vim \
    rsyslog \
    apt-utils \
    zlib1g-dev \
    brotli \
    libtidy-dev \
    apache2-dev \
    git
#    && rm -r /var/lib/apt/lists/*

# Adds necessary rsyslog format for Fail2Ban regex
RUN sed -i -e 's|\$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat|#\$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat|g' /etc/rsyslog.conf && \
    echo '$template CustomFormat,"%timestamp:::date-year%-%timestamp:::date-month%-%timestamp:::date-day% %timestamp:::date-hour%:%timestamp:::date-minute%:%timestamp:::date-second% %HOSTNAME% %syslogtag%%msg%\\n' >> /etc/rsyslog.conf && \
    echo '$ActionFileDefaultTemplate CustomFormat' >> /etc/rsyslog.conf


# Adds setting for Apache to ensure Fail2Ban correctly uses proxy IP on production server setup. i.e. ban the client IP not the proxy IP.
RUN a2enmod remoteip && \
    touch /etc/apache2/conf-enabled/remoteip.conf && \
    printf "%s\n" "RemoteIPHeader X-Forwarded-For" \
           "RemoteIPTrustedProxy $PROXY_IP" > /etc/apache2/conf-enabled/remoteip.conf

# service apache2 restart

# Sets timezone which normally uses dpkg-reconfigure tzdata (2 for America, 65 for Halifax)
ENV TZ=America/Halifax
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN docker-php-ext-install tidy

# Enable memcache and memcached
#RUN mkdir -p /usr/src/php/ext

#RUN apt-get install -y libpq-dev libmemcached-dev && \
#    curl -o memcached.tgz -SL http://pecl.php.net/get/memcached-3.1.3.tgz && \
#        tar -xf memcached.tgz -C /usr/src/php/ext/ && \
#        echo extension=memcached.so >> /usr/local/etc/php/conf.d/memcached.ini && \
#        rm memcached.tgz && \
#        mv /usr/src/php/ext/memcached-3.1.3 /usr/src/php/ext/memcached
#
#RUN apt-get install --no-install-recommends -y unzip libssl-dev libpcre3 libpcre3-dev && \
#    cd /usr/src/php/ext/ && \
#    curl -sSL -o php7.zip https://github.com/websupport-sk/pecl-memcache/archive/NON_BLOCKING_IO_php7.zip && \
#    unzip php7.zip && \
#    mv pecl-memcache-NON_BLOCKING_IO_php7 memcache && \
#    docker-php-ext-configure memcache --with-php-config=/usr/local/bin/php-config && \
#    docker-php-ext-install memcache && \
#    echo "extension=memcache.so" > /usr/local/etc/php/conf.d/ext-memcache.ini && \
#    rm -rf /tmp/pecl-memcache-php7 php7.zip
#
#RUN docker-php-ext-install memcached

RUN a2enmod ext_filter

RUN a2enmod headers

RUN cd /home/ && \
    git clone --recursive --depth=1 https://github.com/kjdev/php-ext-brotli.git && \
    cd php-ext-brotli && \
    phpize && \
    ./configure && \
    make && \
    make install

RUN touch /usr/local/etc/php/conf.d/brotli.ini && \
    printf "%s\n" \
    "extension=brotli.so" >> /usr/local/etc/php/conf.d/brotli.ini

RUN touch /usr/local/etc/php/conf.d/uploads.ini && \
    printf "%s\n" "file_uploads = On" \
    "memory_limit = 512M" \
    "upload_max_filesize = 256M" \
    "post_max_size = 256M" \
    "max_execution_time = 300" \
    "max_input_time = 300" \
    "upload_tmp_dir = /var/www/html/wp-content/tmp/" > /usr/local/etc/php/conf.d/uploads.ini

#RUN chown www-data:root /var/www/html/wp-content && \
#    chown -R www-data:root /var/www/html/wp-content/uploads

RUN printf "%s\n" "ServerTokens Prod" \
    "ServerSignature Off" \
    "Header add Content-Security-Policy: \"default-src $BASE_URL https://fonts.gstatic.com; style-src $BASE_URL https://fonts.googleapis.com https://netdna.bootstrapcdn.com 'unsafe-inline'; script-src $BASE_URL https://dl.dropboxusercontent.com/s/pxxqg90g7zxtt8n/ 'self' 'unsafe-eval' 'unsafe-inline'; font-src $BASE_URL 'self' data: fonts.gstatic.com https://netdna.bootstrapcdn.com; img-src $BASE_URL https://secure.gravatar.com 'self' data:;\""
    "Header set X-Content-Type-Options: \"nosniff\"" \
    "Header always set X-Xss-Protection \"1; mode=block\"" >> /etc/apache2/conf-enabled/security.conf

RUN printf "%s\n" "<IfModule mod_headers.c>" \
    "Header unset Server" \
    "Header always unset X-Powered-By:" \
    "Header unset X-Powered-By" \
    "Header unset X-CF-Powered-By" \
    "Header unset X-Mod-Pagespeed" \
    "Header unset X-Pingback" \
    "Header always set X-Frame-Options \"SAMEORIGIN\"" \
    "Header always set Strict-Transport-Security \"max-age=31536000; includeSubDomains\"" \
    "</IfModule>" >> /etc/apache2/apache2.conf

# -----------------------------------------------------------------------------
# Set ports and env variable HOME
# -----------------------------------------------------------------------------
EXPOSE 80
# -----------------------------------------------------------------------------
# Start
# -----------------------------------------------------------------------------
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["apache2-foreground"]
