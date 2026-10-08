FROM composer:2 AS vendor

WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-interaction --no-scripts --no-autoloader --prefer-dist --ignore-platform-reqs
COPY . .
# Laravel 5.8 PackageManifest only understands the Composer 1 installed.json layout
RUN composer dump-autoload --optimize --no-scripts \
 && php -r '$f = "vendor/composer/installed.json"; $j = json_decode(file_get_contents($f), true); if (isset($j["packages"])) file_put_contents($f, json_encode($j["packages"]));'

FROM php:7.3-apache

RUN docker-php-ext-install pdo_mysql \
 && a2enmod rewrite \
 && sed -ri 's!/var/www/html!/var/www/html/public!g' /etc/apache2/sites-available/*.conf

WORKDIR /var/www/html
COPY --from=vendor /app ./
RUN cp .env.example .env \
 && php artisan key:generate --force \
 && php artisan package:discover \
 && chown -R www-data:www-data storage bootstrap/cache
