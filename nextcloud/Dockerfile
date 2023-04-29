FROM ghcr.io/qwertzlbert/base-build:jammy as BUILD
ARG git_version=main
RUN \
    curl -fsSL -o nextcloud.tar.bz2 \
        "https://download.nextcloud.com/server/releases/nextcloud-${git_version}.tar.bz2"; \
    curl -fsSL -o nextcloud.tar.bz2.asc \
        "https://download.nextcloud.com/server/releases/nextcloud-${git_version}.tar.bz2.asc"; \
    export GNUPGHOME="$(mktemp -d)"; \
    # gpg key from https://nextcloud.com/nextcloud.asc
    gpg --batch --keyserver keyserver.ubuntu.com  --recv-keys 28806A878AE423A28372792ED75899B9A724937A; \
    gpg --batch --verify nextcloud.tar.bz2.asc nextcloud.tar.bz2; \
    tar -xjf nextcloud.tar.bz2 -C /usr/src/; \
    gpgconf --kill all; \
    rm -rf /usr/src/nextcloud/updater; \
    mkdir -p /usr/src/nextcloud/data; \
    mkdir -p /usr/src/nextcloud/custom_apps; \
    chmod +x /usr/src/nextcloud/occ

FROM php:8.1.18-fpm-alpine3.17
COPY --from=BUILD /usr/src/nextcloud /usr/src/nextcloud

# Basic dependencies
RUN set -ex; \
    \
    apk add --no-cache \
        imagemagick \
        rsync \
    ; \
    \
    rm /var/spool/cron/crontabs/root; \
    echo '*/5 * * * * php -f /var/www/html/cron.php' > /var/spool/cron/crontabs/www-data

# PHP extensions

RUN mkdir /var/www/data; \
    chown -R www-data:root /var/www; \
    chmod -R g=u /var/www

VOLUME /var/www/html

COPY nextcloud/*.sh nextcloud/upgrade.exclude /

VOLUME \
    [ \
        "/usr/src/nextcloud/config/", \
        "/php/config", \
        "/php-fpm/config", \
        "/var/www/html" \
    ]

RUN chmod +x /entrypoint.sh; \
    chmod +x /cron.sh;

RUN useradd -M www-data

ENTRYPOINT ["/entrypoint.sh"]
CMD ["php-fpm"]