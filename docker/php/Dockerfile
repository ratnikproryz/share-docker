FROM surnet/alpine-wkhtmltopdf:3.20.2-0.12.6-full as wkhtmltopdf
FROM php:8.2-fpm-alpine

SHELL ["/bin/ash", "-oeux", "pipefail", "-c"]

# tinker(psysh)
ARG PSYSH_DIR=/usr/local/share/psysh
ARG PSYSH_PHP_MANUAL=$PSYSH_DIR/php_manual.sqlite
ARG PHP_MANUAL_URL=http://psysh.org/manual/ja/php_manual.sqlite

# timezone
ARG TZ

# composer
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer

RUN apk add --update --no-cache --virtual=.build-dependencies \
    autoconf \
    gcc \
    g++ \
    make \
    tzdata \
    git
RUN apk add libpng libpng-dev libjpeg-turbo-dev libwebp-dev zlib-dev libxpm-dev gd supervisor
RUN apk add --update --no-cache \
    icu-dev \
    libzip-dev \
    oniguruma-dev \
    imagemagick-dev \
    linux-headers
RUN cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
  echo ${TZ} > /etc/timezone && \
  pecl install xdebug-3.3.1 && \
  pecl install imagick && \
  git clone https://github.com/phpredis/phpredis.git /usr/src/php/ext/redis && \
  apk del .build-dependencies && \
  docker-php-ext-configure gd --with-jpeg && \
  docker-php-ext-install -j$(nproc) gd zip && \
  docker-php-ext-install intl pdo_mysql mbstring bcmath redis exif && \
  docker-php-ext-enable xdebug imagick && \
  mkdir $PSYSH_DIR && curl $PHP_MANUAL_URL -o $PSYSH_PHP_MANUAL
COPY --from=composer:lts /usr/bin/composer /usr/bin/composer

RUN apk add --no-cache \
    libstdc++ \
    libx11 \
    libxrender \
    libxext \
    libssl3 \
    font-noto-cjk \
  && apk add --no-cache --virtual .build-deps \
  && rm -rf /tmp/* \
  && apk del .build-deps

COPY --from=wkhtmltopdf /bin/wkhtmltopdf  /usr/local/bin/wkhtmltopdf
COPY --from=wkhtmltopdf /bin/wkhtmltoimage /usr/local/bin/wkhtmltoimage
COPY --from=wkhtmltopdf /lib/libwkhtmltox* /lib/