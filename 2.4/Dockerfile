FROM debian:bullseye-slim
MAINTAINER Jérémy "Jejem" Desvages <jejem@phyrexia.org>

ENV APACHE_CONFDIR /etc/apache2
ENV APACHE_ENVVARS $APACHE_CONFDIR/envvars

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        apache2 \
        elinks \
        locales \
    ; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    echo "en_GB ISO-8859-1" > /etc/locale.gen; \
    echo "en_GB.ISO-8859-15 ISO-8859-15" >> /etc/locale.gen; \
    echo "en_GB.UTF-8 UTF-8" >> /etc/locale.gen; \
    echo "fr_FR ISO-8859-1" >> /etc/locale.gen; \
    echo "fr_FR@euro ISO-8859-15" >> /etc/locale.gen; \
    echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen; \
    echo "LANG=\"fr_FR.UTF-8\"" > /etc/default/locale; \
    DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales

RUN set -eux; \
    rm -f /etc/localtime; \
    cp -f /usr/share/zoneinfo/Europe/Paris /etc/localtime; \
    echo "Europe/Paris" > /etc/timezone; \
    DEBIAN_FRONTEND=noninteractive dpkg-reconfigure tzdata

RUN set -eux; \
    sed -ri 's/^export ([^=]+)=(.*)$/: ${\1:=\2}\nexport \1/' "$APACHE_ENVVARS"; \
    \
    . "$APACHE_ENVVARS"; \
    for dir in \
        "$APACHE_LOCK_DIR" \
        "$APACHE_RUN_DIR" \
        "$APACHE_LOG_DIR" \
    ; do \
        rm -rvf "$dir"; \
        mkdir -p "$dir"; \
        chown "$APACHE_RUN_USER:$APACHE_RUN_GROUP" "$dir"; \
        chmod 777 "$dir"; \
    done; \
    \
    rm -rvf /var/www/html/*; \
    \
    ln -sfT /dev/stderr "$APACHE_LOG_DIR/error.log"; \
    ln -sfT /dev/stdout "$APACHE_LOG_DIR/access.log"; \
    ln -sfT /dev/stdout "$APACHE_LOG_DIR/other_vhosts_access.log"; \
    chown -R --no-dereference "$APACHE_RUN_USER:$APACHE_RUN_GROUP" "$APACHE_LOG_DIR"

RUN { \
        echo 'DirectoryIndex disabled'; \
        echo 'DirectoryIndex index.php index.html'; \
        echo; \
        echo '<Directory /var/www/>'; \
        echo '\tOptions -Indexes'; \
        echo '\tAllowOverride All'; \
        echo '</Directory>'; \
    } | tee "$APACHE_CONFDIR/conf-available/docker-apache.conf" \
    && a2enconf docker-apache

RUN set -eux; \
    echo "RemoteIPHeader X-Forwarded-For" > /etc/apache2/conf-available/remoteip.conf; \
    echo "RemoteIPTrustedProxy 10.0.0.0/8" >> /etc/apache2/conf-available/remoteip.conf; \
    echo "RemoteIPTrustedProxy 172.16.0.0/12" >> /etc/apache2/conf-available/remoteip.conf; \
    echo "RemoteIPTrustedProxy 192.168.0.0/16" >> /etc/apache2/conf-available/remoteip.conf; \
    echo "LogFormat \"%v:%p %a %l %u %t \\\"%r\\\" %>s %O \\\"%{Referer}i\\\" \\\"%{User-Agent}i\\\"\" vhost_combined" >> /etc/apache2/conf-available/remoteip.conf; \
    echo "LogFormat \"%a %l %u %t \\\"%r\\\" %>s %O \\\"%{Referer}i\\\" \\\"%{User-Agent}i\\\"\" combined" >> /etc/apache2/conf-available/remoteip.conf; \
    echo "LogFormat \"%a %l %u %t \\\"%r\\\" %>s %O\" common" >> /etc/apache2/conf-available/remoteip.conf

RUN set -eux; \
    a2enconf \
        remoteip \
    ; \
    a2enmod \
        proxy_fcgi \
        remoteip \
        rewrite \
    ;

COPY docker-apache-entrypoint /usr/local/bin/

ENTRYPOINT ["docker-apache-entrypoint"]
STOPSIGNAL SIGWINCH

COPY apache2-foreground /usr/local/bin/
WORKDIR /var/www/html

EXPOSE 80
CMD ["apache2-foreground"]
