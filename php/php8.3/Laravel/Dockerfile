# 1. Этап сборки (build stage)
FROM php:8.3-fpm-alpine AS builder

# Установка необходимых пакетов для сборки расширений и самих расширений
RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS

# Устанавливаем библтотеки для сборки расширений
RUN apk add --no-cache --virtual \
        unzip \
        git \
        curl \
        nodejs \
        npm \
        libzip-dev \
        libpng-dev \
        libjpeg-turbo-dev \
        freetype-dev \
    ;

# Собираем расширения
RUN set -eux; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-install gd zip pdo pdo_mysql; \
    docker-php-ext-enable gd;

# Удаление пакетов, которые использовались для сборки, но не нужны в runtime
RUN apk del .build-deps; \
    # Удаление временных файлов
    rm -rf /var/cache/apk/* /tmp/* /usr/src/php*

# Установка Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Настройка рабочего каталога
WORKDIR /var/www/html

# Копирование файлов приложения
COPY . .

RUN set -eux; composer install --no-dev --optimize-autoloader; \
    php artisan migrate && \
    php artisan config:cache && \
    php artisan route:cache && \
    php artisan view:cache; \
    npm install; \
    npm run build; \
    rm -rf ./node_modules /root/.composer/cache; \
    chown -R www-data:www-data /var/www/html

# 2. Финальный этап (production stage)
FROM php:8.3-fpm-alpine

# Установка необходимых пакетов
RUN apk --no-cache add \
    freetype \
    libjpeg-turbo \
    libpng

WORKDIR /var/www/html

# Копируем PHP и расширения из промежуточного образа
COPY --from=builder /usr/local/lib/php/extensions/no-debug-non-zts-20230831 /usr/local/lib/php/extensions/no-debug-non-zts-20230831
COPY --from=builder /var/www/html /var/www/html

EXPOSE 9000

CMD ["php-fpm"]

