FROM debian:bullseye
MAINTAINER Jérémy "Jejem" Desvages <jejem@phyrexia.org>

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        apache2 \
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
        remoteip \
        rewrite \
    ;
