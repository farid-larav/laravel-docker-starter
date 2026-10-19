
FROM php:8.4-cli

# Install PHP extensions and system dependencies
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libzip-dev \
    libicu-dev \
    libonig-dev \
    && docker-php-ext-install \
        pdo_mysql \
        mbstring \
        bcmath \
        pcntl \
        zip \
        intl \
    && rm -rf /var/lib/apt/lists/*

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Project directory
WORKDIR /var/www/html

EXPOSE 8000

# Laravel development server
CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]

