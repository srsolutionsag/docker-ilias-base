ARG PHP_VERSION

FROM php:${PHP_VERSION}

RUN apt-get update && apt-get install -y --no-install-recommends \
        cron \
        curl \
        ghostscript \
        imagemagick \
        libmagickwand-dev \
        ffmpeg \
        libldap-common \
        libldap-dev \
        libpng-dev \
        libjpeg-dev \
        libfreetype6-dev \
        libxslt-dev \
        libzip-dev \
        mariadb-client \
        pwgen \
        unzip \
        zip \
        xfonts-75dpi \
        xfonts-base \
    && rm -rf /var/lib/apt/* /var/cache/apt/*

# Compile imagick from source to patch unterminated preprocessor conditions error
# See https://github.com/Imagick/imagick/issues/640
COPY 10-alpha/imagick_fix_unterminated_preprocessor_conditions.patch /tmp/
RUN curl -SL 'https://github.com/Imagick/imagick/archive/refs/tags/3.7.0.tar.gz' -o /tmp/imagick.tar.gz \
    && mkdir -p /tmp/imagick \
    && tar -xf /tmp/imagick.tar.gz -C /tmp/imagick --strip-components=1 \
    && patch -d /tmp/imagick -p1 -i /tmp/imagick_fix_unterminated_preprocessor_conditions.patch \
    && docker-php-ext-configure /tmp/imagick \
    && docker-php-ext-install /tmp/imagick \
    && rm -r /tmp/imagick_fix_unterminated_preprocessor_conditions.patch /tmp/imagick.tar.gz /tmp/imagick

RUN sed -i '/disable ghostscript format types/,+6d' /etc/ImageMagick-6/policy.xml

RUN docker-php-ext-configure ldap --with-libdir=lib/$(uname -m)-linux-gnu/ \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install \
        gd \
        ldap \
        mysqli \
        pdo \
        pdo_mysql \
        soap \
        xsl \
        zip \
    && pecl install pecl install "channel://pecl.php.net/xmlrpc-1.0.0RC3" \
    && docker-php-ext-enable xmlrpc

RUN a2enmod \
    expires \
    headers \
    rewrite

ENV ILIAS_WWW_PATH=/var/www/html
ENV ILIAS_DATA_PATH=/var/www/html/public/data
ENV ILIAS_ILIASDATA_PATH=/var/iliasdata/ilias

RUN mkdir -p ${ILIAS_ILIASDATA_PATH} \
    && chown www-data:root ${ILIAS_ILIASDATA_PATH} \
    && chmod 775 ${ILIAS_ILIASDATA_PATH}
VOLUME ${ILIAS_ILIASDATA_PATH}

RUN mkdir -p ${ILIAS_DATA_PATH} \
    && chown www-data:root ${ILIAS_DATA_PATH} \
    && chmod 775 ${ILIAS_DATA_PATH}
VOLUME ${ILIAS_DATA_PATH}

COPY docker-ilias-entrypoint-new /usr/local/bin/docker-ilias-entrypoint

RUN sed -i 's|DocumentRoot /var/www/html|DocumentRoot /var/www/html/public|g' /etc/apache2/sites-enabled/000-default.conf

ENTRYPOINT ["docker-ilias-entrypoint"]
CMD ["apache2-foreground"]
EXPOSE 80
