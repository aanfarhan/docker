FROM php:7.4-fpm-buster
# install necessary packages

RUN set -x \
        && apt-get update \
        && apt-get install libaio-dev mc unzip zlib1g-dev --no-install-recommends --no-install-suggests -y \
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*

ENV ORA_CLIENT=instantclient_19_1
ENV ORA_VERSION=19.1
#change to use another verison, e.g: oci8-2.2.0
ENV OCI_VERSION=oci8-2.2.0

# install oracle instant client
# copy instantclient from local dir
COPY ${ORA_CLIENT} /opt/${ORA_CLIENT}
#RUN ln -s /opt/${ORA_CLIENT}/libclntsh.so.${ORA_VERSION} /opt/${ORA_CLIENT}/libclntsh.so
#RUN ln -s /opt/${ORA_CLIENT}/libocci.so.${ORA_VERSION} /opt/${ORA_CLIENT}/libocci.so
RUN echo /opt/${ORA_CLIENT} > /etc/ld.so.conf.d/oracle-instantclient.conf
RUN ldconfig

# install &amp; enable oci8

RUN pecl install --onlyreqdeps --nobuild ${OCI_VERSION} \
        && cd "$(pecl config-get temp_dir)/oci8" \
        && phpize \
        && ./configure --with-oci8=instantclient,/opt/${ORA_CLIENT} \
        && make && make install \
        && docker-php-ext-enable oci8

# install &amp; enable pdo-oci

RUN docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/${ORA_CLIENT} \
        && docker-php-ext-install pdo_oci

# copy dev php.ini

RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

# install composer

RUN curl -sS https://getcomposer.org/installer | php -- \
        --install-dir=/usr/local/bin \
        --filename=composer

WORKDIR /home/www

EXPOSE 9000

CMD ["php-fpm"]