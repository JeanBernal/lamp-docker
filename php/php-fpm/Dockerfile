#FROM php:7.2-fpm
FROM php:5.5-fpm
RUN apt-get update && apt-get install -y \
        git \
        libzip-dev \
        zip \
        unzip \
        libpq-dev \
        firebird-dev \
        --yes --no-install-recommends \
	libssl-dev
RUN docker-php-ext-configure zip --with-libzip
#RUN docker-php-ext-install zip pdo_firebird pdo_pgsql pdo_mysql
#RUN pecl install mongodb \
 #  && docker-php-ext-enable mongodb
RUN docker-php-ext-install mysqli
