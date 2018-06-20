FROM alpine:3.7

LABEL maintainer="a.v.galushko86@gmail.com" \
    license="MIT" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.vendor="graze" \
    org.label-schema.name="php-alpine" \
    org.label-schema.description="small php image based on alpine"

RUN set -xe \
    && apk add --update --no-cache \
    ca-certificates \
    curl \
    openssh-client \
    libmemcached-libs \
    libevent \
    libssl1.0 \
    musl \
    yaml \
    php7 \
    php7-fpm \
    php7-apcu \
    php7-bcmath \
    php7-ctype \
    php7-curl \
    php7-dom \
    php7-fileinfo \
    php7-iconv \
    php7-intl \
    php7-json \
    php7-openssl \
    php7-opcache \
    php7-mbstring \
    php7-memcached \
    php7-mcrypt \
    php7-mysqlnd \
    php7-mysqli \
    php7-pcntl \
    php7-pgsql \
    php7-pdo_mysql \
    php7-pdo_pgsql \
    php7-pdo_sqlite \
    php7-phar \
    php7-posix \
    php7-session \
    php7-simplexml \
    php7-soap \
    php7-sockets \
    php7-tokenizer \
    php7-xml \
    php7-xmlreader \
    php7-xmlwriter \
    php7-zip \
    php7-zlib \
    php7-gd \
    && apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing/ --allow-untrusted \
    gnu-libiconv

# install and remove building packages
ENV PHPIZE_DEPS autoconf file g++ gcc libc-dev make pkgconf re2c php7-dev php7-pear \
    yaml-dev libevent-dev openssl-dev

ENV PHP_INI_DIR /etc/php7

RUN set -xe \
    && apk add --no-cache --repository "http://dl-cdn.alpinelinux.org/alpine/edge/testing" \
    --virtual .phpize-deps \
    $PHPIZE_DEPS \
    && sed -i 's/^exec $PHP -C -n/exec $PHP -C/g' $(which pecl) \
    && pecl channel-update pecl.php.net \
    && pecl install yaml event \
    && echo "extension=yaml.so" > $PHP_INI_DIR/conf.d/01_yaml.ini \
    && echo "extension=event.so" > $PHP_INI_DIR/conf.d/01_event.ini \
    && rm -rf /usr/share/php7 \
    && rm -rf /tmp/* \
    && apk del .phpize-deps
RUN  php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && php -r "unlink('composer-setup.php');" \
    # Tweak configs
    && sed -i \
        -e "s,expose_php = On,expose_php = Off,g" \
        -e "s,;cgi.fix_pathinfo=1,cgi.fix_pathinfo=0,g" \
        -e "s,post_max_size = 8M,post_max_size = 100M,g" \
        -e "s,upload_max_filesize = 2M,upload_max_filesize = 100M,g" \
        /etc/php7/php.ini \
    && sed -i \
        -e "s,;daemonize = yes,daemonize = no,g" \
        -e "s,user = nobody,user = www,g" \
        -e "s,;chdir = /var/www,chdir = /var/www/mediawiki,g" \
        -e "s,;listen.owner = nobody,listen.owner = www,g" \
        -e "s,;listen.group = nobody,listen.group = www,g" \
        -e "s,listen = 127.0.0.1:9000,listen = 0.0.0.0:9000,g" \
        -e "s,;clear_env = no,clear_env = no,g" \
        /etc/php7/php-fpm.d/www.conf \
    # forward logs to docker log collector
    && ln -sf /dev/stderr /var/log/php7/error.log \
    && mkdir -p /var/www \
    && cd /tmp \
    && adduser -S -D -H www \
    && addgroup www

COPY php7/ $PHP_INI_DIR/
WORKDIR /var/www/
ARG BUILD_DATE
ARG VCS_REF
LABEL org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.build-date=$BUILD_DATE

# Fix for iconv: https://github.com/docker-library/php/issues/240
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

WORKDIR /
ENTRYPOINT  /usr/sbin/php-fpm7 -F