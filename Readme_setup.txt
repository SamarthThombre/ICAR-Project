start 

#1 created new folder ICAR-Project

#2 create Dockerfile && docker-compose.yml

$$ Dockerfile content

FROM php:8.4-apache

# Install system dependencies and PHP extensions required for Drupal 11
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libpq-dev \
    libxml2-dev \
    libzip-dev \
    zip \
    unzip \
    git \
    && rm -rf /var/lib/apt/lists/*

# Configure and install PHP extensions
RUN docker-php-ext-configure gd --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    gd \
    opcache \
    pdo \
    pdo_mysql \
    zip

# Configure recommended PHP.ini settings for Drupal
RUN { \
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=60'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Enable Apache rewrite module for Drupal clean URLs
RUN a2enmod rewrite

# Install Composer globally
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set the working directory
WORKDIR /var/www/html

# Adjust Apache configuration to point to the 'web' subdirectory
RUN sed -i 's|/var/www/html|/var/www/html/web|g' /etc/apache2/sites-available/000-default.conf



$$ docker-compose.yml content


services:
  drupal:
    build: .
    container_name: drupal_app
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
    environment:
      - DRUPAL_DATABASE_HOST=db
      - DRUPAL_DATABASE_NAME=drupal_db
      - DRUPAL_DATABASE_USER=drupal_user
      - DRUPAL_DATABASE_PASSWORD=drupal_password
    depends_on:
      - db

  db:
    image: mariadb:10.11
    container_name: drupal_db
    restart: always
    environment:
      MYSQL_DATABASE: drupal_db
      MYSQL_USER: drupal_user
      MYSQL_PASSWORD: drupal_password
      MYSQL_ROOT_PASSWORD: root_password
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3307:3306"

volumes:
  db_data:



#3 creating docker image && seting up drupal

# Download Drupal using a temporary container:

sudo docker compose run --rm drupal composer create-project drupal/recommended-project temporary-dir --no-interaction

# Move the files into your main folder:

sudo docker compose run --rm drupal bash -c "mv temporary-dir/* temporary-dir/.* . 2>/dev/null || true && rmdir temporary-dir"

# Now that the web folder exists, start the container normally:

sudo docker compose up -d


#4 set drupal in browser

Open your web browser and go to: http://localhost:8080

You should see the Drupal installation wizard again. Just like last time, choose your language, select the "Standard" installation profile, and when you reach the Database Configuration screen, remember our crucial Docker fix:

    Click Advanced Options.

    Change the Host from localhost to db.

    Database name: drupal_db

    Database username: drupal_user

    Database password: drupal_password

