FROM php:7.4-fpm

#RUN docker-php-ext-install \
#        zip \
#        soap \
#        exif \
#        hash \
#        fileinfo \
#        mysqli \
#        opcache \
#        openssl \
#        pcre \
#        gd \
#        filter \
#        iconv \
#        zlib \
#        xmlreader \
#        simplexml \
#        intl \
#        mbstring \
#        xmlrpc \
#    && docker-php-ext-enable \
#        zip \
#        soap \
#        exif \
#        hash \
#        fileinfo \
#        mysqli \
#        opcache \
#        openssl \
#        pcre \
#        gd \
#        filter \
#        iconv \
#        zlib \
#        xmlreader \
#        simplexml \
#        intl \
#        mbstring \
#        xmlrpc
 
#RUN docker-php-ext-configure gd --with-freetype --with-jpeg

RUN set -x \
    && addgroup --system --gid 101 nginx \                                                                                                                                                                       
    && adduser --system --disabled-login --ingroup nginx --no-create-home --gecos "nginx user" --shell /bin/false --uid 101 nginx

RUN set -x \
    && mkdir -p /var/log/php-fpm \
    && touch /var/log/php-fpm/plemp.slow.log

COPY php.ini $PHP_INI_DIR/php.ini
COPY plemp-php-pool.conf /usr/local/etc/php-fpm.d/plemp-php-pool.conf
