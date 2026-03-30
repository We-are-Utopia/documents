# syntax=docker/dockerfile:1.7
FROM composer:2.8 AS vendor
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-dev --prefer-dist --no-interaction --no-progress --optimize-autoloader
COPY . .
RUN composer dump-autoload --optimize --classmap-authoritative

FROM php:8.3-fpm-alpine AS runtime
WORKDIR /var/www/html
RUN apk add --no-cache icu-dev libzip-dev oniguruma-dev \
    && docker-php-ext-install pdo_mysql intl zip opcache
COPY --from=vendor /app ./
RUN addgroup -S app && adduser -S app -G app \
    && chown -R app:app storage bootstrap/cache
USER app
EXPOSE 9000
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 CMD php -v || exit 1
CMD ["php-fpm", "-F"]
