FROM php:7.2-apache
MAINTAINER Alexander Hofbauer <hofbauer.alexander@gmail.com>

ENV TYPO3CMS_PATH '/var/www/html/typo3/vendor/bin/typo3cms'
ENV COMPOSER_ALLOW_SUPERUSER 1

# Install TYPO3 & Composer
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
#### TYPO3
    libxml2-dev libfreetype6-dev \
    libjpeg62-turbo-dev libjpeg-dev \
    libmcrypt-dev \
    zlib1g-dev \
    zip zlib1g-dev libzip-dev \
    libpng-dev libxpm-dev \
    libxml2-dev \
    graphicsmagick \
    tar \
#### Composer
    wget curl git-core subversion mercurial
#### add PHP packages
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install -j$(nproc) mysqli soap gd zip opcache && \
    echo 'always_populate_raw_post_data = -1\nmax_execution_time = 240\nmax_input_vars = 1500\nupload_max_filesize = 32M\npost_max_size = 32M' > /usr/local/etc/php/conf.d/typo3.ini && \
#### configure Apache
    a2enmod rewrite && \
#### Clenaup
    apt-get clean && \
    apt-get -y purge \
        libxml2-dev libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        zlib1g-dev && \
    rm -rf /var/lib/apt/lists/* /usr/src/*

#### Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

###### Install TYPO3 using Composer
RUN composer create-project typo3/cms-base-distribution typo3

###### Create Volumes
VOLUME /var/www/html/typo3/public/fileadmin
VOLUME /var/www/html/typo3/public/typo3conf
VOLUME /var/www/html/typo3/public/typo3temp
VOLUME /var/www/html/typo3/public/uploads

###### Set Workdir
WORKDIR /var/www/html/typo3
