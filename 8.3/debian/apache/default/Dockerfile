FROM php:8.3.30-apache-trixie

LABEL maintainer="Nico Herbig <nico@nicoherbig.io>"


# Add additional Apache user and add them to the Apache group (www-data) to ensure that the ID (UID)
# is consistently assigned regardless of which additional dependencies are added.

ENV WWW_USER_ID=10033 WWW_USER=www-user
ENV WWW_GROUP_ID=33 WWW_GROUP=www-data

RUN useradd --gid ${WWW_GROUP} --uid ${WWW_USER_ID} --create-home --shell /bin/bash ${WWW_USER}


# Set time zone to "Europe/Berlin".

ENV TZ=Europe/Berlin

RUN set -ex \
    && apt-get update \
    && apt-get install -y --no-install-recommends tzdata \
    && rm -rf /var/lib/apt/lists/* \
    && ln -fns /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && { \
        echo 'date.timezone = ${TZ}'; \
       } | tee /usr/local/etc/php/conf.d/docker-php-timezone.ini


# Set locale to "de_DE.UTF-8".

RUN set -ex \
    && if [ -f /etc/dpkg/dpkg.cfg.d/docker ]; then \
            # If this file exists, we are likely in "debian:xxx-slim" and locales are thus being excluded
            # so we need to remove that exclusion (since we need locales).
            grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d/docker; \
            sed -ri '/\/usr\/share\/locale/d' /etc/dpkg/dpkg.cfg.d/docker; \
            ! grep -q '/usr/share/locale' /etc/dpkg/dpkg.cfg.d/docker; \
       fi \
    && apt-get update \
    && apt-get install -y --no-install-recommends locales \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -i de_DE -c -f UTF-8 -A /usr/share/locale/locale.alias de_DE.UTF-8 \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en


# Configure Apache and PHP.
#   - activate commonly used Apache modules
#   - switch Apache server signature off
#   - make Apache document root configureable by environment variable
#   - use PHP production settings as default

ENV APACHE_DOCUMENT_ROOT=/app

RUN set -ex \
    && a2enmod expires deflate headers rewrite \
    && sed -i -e '/ServerTokens/s/OS/Prod/g' -e '/ServerSignature/s/On/Off/g' /etc/apache2/conf-available/security.conf \
    && sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf \
    && sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf \
    && cp ${PHP_INI_DIR}/php.ini-production ${PHP_INI_DIR}/php.ini

WORKDIR ${APACHE_DOCUMENT_ROOT}
