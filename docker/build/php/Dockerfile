FROM php:8.3-fpm as base

# set main params
ARG BUILD_ARGUMENT_ENV=dev
ENV ENV=$BUILD_ARGUMENT_ENV
ENV APP_HOME /var/www/html
ARG HOST_UID=1000
ARG HOST_GID=1000
ENV USERNAME=www-data
ARG INSIDE_DOCKER_CONTAINER=1
ENV INSIDE_DOCKER_CONTAINER=$INSIDE_DOCKER_CONTAINER
ARG XDEBUG_CONFIG=main
ENV XDEBUG_CONFIG=$XDEBUG_CONFIG

# check environment
RUN if [ "$BUILD_ARGUMENT_ENV" = "default" ]; then echo "Set BUILD_ARGUMENT_ENV in docker build-args like --build-arg BUILD_ARGUMENT_ENV=dev" && exit 2; \
    elif [ "$BUILD_ARGUMENT_ENV" = "dev" ]; then echo "Building development environment."; \
    elif [ "$BUILD_ARGUMENT_ENV" = "test" ]; then echo "Building test environment."; \
    elif [ "$BUILD_ARGUMENT_ENV" = "staging" ]; then echo "Building staging environment."; \
    elif [ "$BUILD_ARGUMENT_ENV" = "prod" ]; then echo "Building production environment."; \
    else echo "Set correct BUILD_ARGUMENT_ENV in docker build-args like --build-arg BUILD_ARGUMENT_ENV=dev. Available choices are dev,test,staging,prod." && exit 2; \
    fi

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

RUN apt-get update && apt-get upgrade -y && apt-get install -y \
      procps \
      nano \
      git \
      unzip \
      sudo \
      zip \
      locales \
   && sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen \
   && chmod +x /usr/local/bin/install-php-extensions && install-php-extensions \
      gd \
      pdo_mysql \
      sockets \
      intl \
      bcmath \
      opcache \
      zip \
      http \
    && rm -rf /tmp/* \
    && rm -rf /var/list/apt/* \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:UTF-8
ENV LC_ALL=en_US.UTF-8

# put php config
COPY ./docker/config/$BUILD_ARGUMENT_ENV/www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./docker/config/$BUILD_ARGUMENT_ENV/php.ini /usr/local/etc/php/php.ini

# install Xdebug in case dev/test environment
COPY --chmod=755 ./docker/build/php/do_we_need_xdebug.sh /tmp/
COPY ./docker/config/dev/xdebug-${XDEBUG_CONFIG}.ini /tmp/xdebug.ini
RUN /tmp/do_we_need_xdebug.sh

# install composer
COPY --from=composer:latest --chmod=755 /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1

FROM base

# copy source files and config file
ADD --chown=${USERNAME}:${USERNAME} application $APP_HOME
COPY --chown=${USERNAME}:${USERNAME} .env.$ENV $APP_HOME/.env

# set working directory
WORKDIR $APP_HOME

RUN for DIR in 'api backend console frontend'; do \
      if [ -d $APP_HOME/$DIR/runtime ]; then chown -R ${USERNAME}:${USERNAME} $APP_HOME/$DIR/runtime; fi \
    done

USER ${USERNAME}

# install all PHP dependencies
RUN if [ "$BUILD_ARGUMENT_ENV" = "dev" ] || [ "$BUILD_ARGUMENT_ENV" = "test" ]; \
    then COMPOSER_MEMORY_LIMIT=-1 COMPOSER_CACHE_READ_ONLY=1 composer install --optimize-autoloader --no-interaction --no-progress; \
    else COMPOSER_MEMORY_LIMIT=-1 COMPOSER_CACHE_READ_ONLY=1 composer install --optimize-autoloader --no-interaction --no-progress --no-dev; \
    fi

USER root
