FROM php:7.1.7-fpm

RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        curl \
        libcurl4-gnutls-dev \
        libmemcached-dev \
        zlib1g-dev \
        wget \
    && rm -rf /var/lib/apt/lists/*

RUN docker-php-source extract \
    && docker-php-ext-install bcmath pdo_mysql zip \
    && docker-php-source delete

RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

RUN pecl install redis-4.0.2 \
    && pecl install yaf-3.0.6 \
    && pecl install mongodb-1.5.3 \
    && pecl install yar-2.0.4 \
    && pecl install memcached-3.0.4 \
    && docker-php-ext-enable redis yaf mongodb yar memcached

RUN curl -L -o /tmp/tideways.tar.gz "https://github.com/tideways/php-xhprof-extension/archive/v4.1.7.tar.gz" \
    && mkdir tideways \
    && tar -xf /tmp/tideways.tar.gz -C tideways --strip-components=1 \
    && rm /tmp/tideways.tar.gz \
    && ( cd tideways && phpize && ./configure && make -j$(nproc) && make install ) \
    && docker-php-ext-enable tideways \
    && rm -r tideways

RUN curl -sS https://getcomposer.org/installer | \
    php -- --install-dir=/usr/bin/ --filename=composer \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/