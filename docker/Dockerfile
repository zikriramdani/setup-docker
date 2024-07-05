FROM amazonlinux:2023

LABEL maintanice="ZikriRamdani"

WORKDIR /var/www/html

ENV TZ=UTC
ENV SUPERVISOR_PHP_COMMAND="/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan serve --host=0.0.0.0 --port=80"

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# import gpg key
RUN rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7 && \
    rpm --import https://dev-certifications.s3-ap-southeast-1.amazonaws.com/pub/IUS-COMMUNITY-GPG-KEY

RUN dnf update -y \
    # install server dependencies
    && dnf install -y gnupg ca-certificates zip unzip git \
        libzip libzip-devel wget vim cronie procps libXrender* fontconfig-devel \
        libXext google-noto-cjk-fonts gcc-c++ make ImageMagick ImageMagick-devel \
        zlib zlib-devel pcre-devel python-pip \
    && dnf update -y \
    # install php and extensions
    && dnf install -y php8.2 php8.2-mysqlnd php8.2-gd \
        php8.2-common php8.2-devel php8.2-soap php8.2-bcmath \
        php8.2-mbstring php8.2-intl php8.2-ldap php8.2-zip \
        php8.2-sodium \
        php-pear php-json php-cgi \
    # install composer
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    # install nodejs
    && dnf install -y nodejs20 nodejs20-npm \
    && ln -s /usr/bin/node-20 /usr/bin/node \
    && ln -s /usr/bin/npm-20 /usr/bin/npm \
    && ln -s /usr/bin/npx-20 /usr/bin/npx \
    && yum update -y && dnf update -y \
    # install mariadb/mysql
    && yum install -y mariadb105-server \
    && yum autoremove && dnf autoremove \
    && yum clean all && dnf clean all \
    # install supervisor via "pip"
    && pip install supervisor

# install extension php via "pecl"
RUN pecl channel-update pecl.php.net \
    && printf "\n" | pecl install imagick redis \
        msgpack igbinary swoole xdebug pcov

RUN setcap "cap_net_bind_service=+ep" /usr/bin/php

RUN groupadd --force -g 1000 zikriramdani
RUN useradd -ms /bin/bash --no-user-group -g 1000 -u 1337 zikriramdani

RUN mkdir /var/log/supervisor

COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php.ini
ADD php.d/ /etc/php.d/
RUN chmod +x /usr/local/bin/start-container

EXPOSE 8000

ENTRYPOINT ["start-container"]