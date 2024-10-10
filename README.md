# Getting Started with PHP Laravel in a Docker Container

## Introduction
Laravel is a powerful PHP framework designed for web artisans. It provides an elegant syntax and a robust set of tools to build modern web applications. Docker, on the other hand, is a platform that allows you to package applications into containers, ensuring consistency across different environments. Combining Laravel with Docker can streamline your development workflow and simplify deployment.

## Prerequisites
Before we begin, make sure you have the following installed on your machine:
- **Docker**: [Download and install Docker](https://www.docker.com/get-started) for your operating system.
- **Composer**: [Download and install Composer](https://getcomposer.org/download/) to manage PHP dependencies.

## Step 1: Create a New Laravel Project
First, we need to create a new Laravel project. Open your terminal and run the following command:

```bash
composer create-project --prefer-dist laravel/laravel my-laravel-app
```

This will create a new Laravel project in the `my-laravel-app` directory.

## Step 2: Set Up Docker
Next, we need to set up Docker for our Laravel project. Create a `Dockerfile` in the root directory of your project with the following content:

```Dockerfile
# Use the official PHP image as the base image
FROM php:8.0-fpm

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www-data:www-data . /var/www

# Change current user to www
USER www-data

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
```

## Step 3: Create a Docker Compose File
Create a `docker-compose.yml` file in the root directory with the following content:

```yaml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: laravel-app
    container_name: laravel-app
    restart: unless-stopped
    working_dir: /var/www
    volumes:
      - .:/var/www
      - ./docker/php/local.ini:/usr/local/etc/php/conf.d/local.ini
    networks:
      - laravel

  webserver:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - .:/var/www
      - ./docker/nginx/conf.d:/etc/nginx/conf.d
    networks:
      - laravel

  db:
    image: mysql:5.7
    container_name: mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
      MYSQL_USER: laravel
      MYSQL_PASSWORD: laravel
    ports:
      - "3306:3306"
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - laravel

networks:
  laravel:

volumes:
  dbdata:
```

## Step 4: Configure Nginx
Create a directory `docker/nginx/conf.d` and add a configuration file `default.conf` with the following content:

```nginx
server {
    listen 80;
    index index.php index.html;
    server_name localhost;
    root /var/www/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## Step 5: Start Docker Containers
With everything set up, you can now start your Docker containers. Run the following command in your terminal:

```bash
docker-compose up -d
```

This will build and start the containers in the background.

## Step 6: Access Your Laravel Application
Open your browser and navigate to `http://localhost:8000`. You should see the Laravel welcome page, indicating that your Laravel application is up and running inside a Docker container.
