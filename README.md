# Laravel 10 Production Deployment on AWS EC2 with Blue-Green Deployment & GitHub Actions

This repository provides a production-ready guide for deploying a Laravel 10 application on AWS EC2 using Blue-Green Deployment, Zero Downtime, and GitHub Actions CI/CD.

It covers server setup, Nginx, PHP-FPM, MySQL, SSL, deployment automation, health checks, and rollback strategies following real-world deployment practices.

---

# Setup Ubuntu in EC2 Intance

## Step 1: Open the EC2 Dashboard

1. Search for **EC2** in the AWS search bar.
2. Select **EC2** from the services list.
3. The EC2 Dashboard will open.

---

## Step 2: Launch an Instance

1. Click **Launch Instance**.
2. Enter a name for your instance.
3. Choose an Instance Type
4. Create or Select a Key Pair
5. Configure Network Settings
6. Configure Storage
7. Click **Launch Instance**.
8. Connect to the Instance

# Setup Nginx in Ubuntu

## Step 1: Nginx Installation and Configuration Guide

1. sudo apt update
2. sudo apt upgrade -y
3. sudo apt install nginx -y
4. nginx -v
5. sudo systemctl start nginx
6. sudo systemctl enable nginx
7. sudo systemctl status nginx

Exit the status screen by pressing `q`.

---

## Step 2: Verify Nginx

Open your browser and visit:

```text
http://<PUBLIC_IP>
```

## You should see the **Welcome to Nginx!** page.

# Install PHP 8.2 with Required Extensions

## Step 1: Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

---

## Step 2: Install Required Package

```bash
sudo apt install -y software-properties-common
```

---

## Step 3: Add the PHP Repository

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
```

---

## Step 4: Install PHP 8.2 and Required Extensions

```bash
sudo apt install -y \
php8.2 \
php8.2-cli \
php8.2-common \
php8.2-fpm \
php8.2-mysql \
php8.2-mbstring \
php8.2-xml \
php8.2-bcmath \
php8.2-curl \
php8.2-zip \
php8.2-gd \
php8.2-intl \
php8.2-soap \
php8.2-readline \
php8.2-opcache \
php8.2-redis \
php8.2-imagick
```

---

## Step 5: Verify the Installation

Check the installed PHP version.

```bash
php -v
```

Example output:

```text
PHP 8.2.x (cli)
```

# Install Composer

## Step 1: Download the Composer Installer

```bash
curl -sS https://getcomposer.org/installer -o composer-setup.php
```

---

## Step 2: Verify the Installer (Optional)

```bash
HASH=$(curl -sS https://composer.github.io/installer.sig)

php -r "if (hash_file('sha384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
```

---

## Step 3: Install Composer Globally

```bash
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

---

## Step 4: Remove the Installer

```bash
rm composer-setup.php
```

---

## Step 5: Verify the Installation

Check the installed Composer version.

```bash
composer --version
```

Example output:

```text
Composer version 2.x.x
```

---

# Install MySQL

## Step 1: Update Package Index

```bash
sudo apt update
sudo apt upgrade -y
```

---

## Step 2: Install MySQL

```bash
sudo apt install mysql-server -y
```

---

## Step 3: Verify Installation

```bash
mysql --version
```

Example

```
mysql Ver 8.0.xx
```

---

## Step 4: Check Service

```bash
sudo systemctl status mysql
```

---

## Step 5: Enable Auto Start

```bash
sudo systemctl enable mysql
```

---

## Step 6: Start MySQL

```bash
sudo systemctl start mysql
```

---

## Step 7: Restart MySQL

```bash
sudo systemctl restart mysql
```

---

## Step 8: Check Listening Port

```bash
sudo ss -tulpn | grep 3306
```

Expected

```
127.0.0.1:3306
```

Never expose port **3306** publicly unless absolutely required.

## Step 9: Secure MySQL

Run

```bash
sudo mysql_secure_installation
```

## Step 10: Login

```bash
sudo mysql
```

or

```bash
mysql -u root -p
```

Exit

```sql
exit;
```

## Step 11: Create Database

```sql
CREATE DATABASE database_name
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

Verify

```sql
SHOW DATABASES;
```

## Step 12: Create Database User

```sql
CREATE USER 'database_user'@'localhost'
IDENTIFIED BY 'password';
```

Grant Permissions

```sql
GRANT ALL PRIVILEGES
ON database_name.*
TO 'database_user'@'localhost';
```

Reload

```sql
FLUSH PRIVILEGES;
```

Verify

```sql
SHOW GRANTS FOR 'database_user'@'localhost';
```

Never use **root** for Laravel applications.

```bash
mysql -u database_user -p
```

Select Database

```sql
USE database_name;
```

Show Tables

```sql
SHOW TABLES;
```

Never expose MySQL publicly.

```text
3306 → Private Network Only
```

# Create Deployment Directory

```text
/var/www/laravel-project
│
├── blue/
├── green/
├── shared/
│   ├── storage/
│   ├── bootstrap/cache/
│   └── .env
│
└── current -> blue
```

```bash
1. sudo mkdir -p /var/www/laravel-project
2. sudo mkdir -p /var/www/laravel-project{blue, green, shared}
3. sudo mkdir -p /var/www/laravel-project/shared/storage
4. sudo mkdir -p /var/www/laravel-project/shared/bootstrap/cache
```

# Directory Permission

```bash
1. sudo chown -R ubuntu:www-data /var/www/laravel-project
2. Directory Permission: sudo find /var/www/laravel-project -type d -exec chmod 775 {} \;
3. File Permission: sudo find /var/www/laravel-project -type f -exec chmod 664 {} \;
4. cd /var/www/laravel-project
5. sudo chmod -R 775 storage
6. sudo chmod -R 775 bootstrap/cache
7. sudo chmod -R g+w storage
8. sudo chmod -R g+w bootstrap/cache
9. Add www-data Group in Ubuntu User: sudo usermod -aG www-data ubuntu
10. exit and again login
11. Check: groups
12. You will get output: ubuntu -> www-data
13. Add Setgid Bit:  sudo find /var/www/laravel-project -type d -exec chmod g+s {} \;
14. sudo chown -R ubuntu:www-data /var/www/laravel-project/shared
15. sudo chmod -R 775 /var/www/laravel-project/shared
16. sudo chown -R ubuntu:www-data /var/www/laravel-project/blue
17. sudo chown -R ubuntu:www-data /var/www/laravel-project/green
18. Check: ls -la /var/www/laravel-project
```

# First Deployment (Blue)

```bash
1. cd ~
2. ln -sfn /var/www/laravel-project/blue /var/www/laravel-project/current
3. cd /var/www/laravel-project
4. git clone https://github.com/username/project.git blue
5. cd blue
6. composer install --no-dev --prefer-dist --optimize-autoloader
7. php artisan key:generate
8. php artisan migrate --force
9. php artisan optimize
10. cp .env.example /var/www/laravel-project/shared/.env
11. ln -s /var/www/laravel-project/shared/.env .env
12. cp -R storage/. /var/www/laravel-project/shared/storage/
13. rm -rf storage
14. ln -s /var/www/laravel-project/shared/storage storage
15. cp -R bootstrap/cache/. /var/www/laravel-project/shared/bootstrap/cache/
16. rm -rf bootstrap/cache
17. ln -s /var/www/laravel-project/shared/bootstrap/cache bootstrap/cache
18. Check Symlink: ls -la
19. Check current live project: readlink current

```

# Deploy New Version (Green)

```bash
1. cd ~
2. ln -sfn /var/www/laravel-project/green /var/www/laravel-project/current
3. cd /var/www/laravel-project
4. git clone https://github.com/username/project.git green
5. cd green
6. composer install --no-dev --prefer-dist --optimize-autoloader
7. php artisan key:generate
8. php artisan migrate --force
9. php artisan optimize
10. cp .env.example /var/www/laravel-project/shared/.env
11. ln -s /var/www/laravel-project/shared/.env .env
12. cp -R storage/. /var/www/laravel-project/shared/storage/
13. rm -rf storage
14. ln -s /var/www/laravel-project/shared/storage storage
15. cp -R bootstrap/cache/. /var/www/laravel-project/shared/bootstrap/cache/
16. rm -rf bootstrap/cache
17. ln -s /var/www/laravel-project/shared/bootstrap/cache bootstrap/cache
18. Check Symlink: ls -la
19. Check current live project: readlink current

```

# Configure Database
nano .env
```text
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=db_name
DB_USERNAME=db_user
DB_PASSWORD=db_password

```

# Setup conf file

## Step 1: Create & Open custom laravel.conf file

```bash
sudo nano /etc/nginx/sites-available/laravel
```

## Step 2: Enter this code in laravel.conf file

```bash
server {

    listen 80;

    server_name YOUR_DOMAIN_OR_IP;

    root /var/www/app/current/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## Step 3: Enable & Test the Configuration

```bash
sudo ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
```

Remove default conf file

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Before restarting Nginx, verify the configuration.

```bash
sudo nginx -t
```

Expected output:

```text
syntax is ok
test is successful
```

## Step 4: Reload Nginx

```bash
sudo systemctl restart nginx
```

# GitHub Actions CI/CD (Blue-Green Deployment)

## Step 1: Structure

```text
.github/
    └── workflows/
        └── deploy.yml
```

## Step 2: GitHub Secrets

1. EC2_HOST=43.xxx.xxx.xxx
2. EC2_USERNAME=ubuntu
3. EC2_SSH_KEY=<Private SSH Key>

## Step 3: GitHub Actions Workflow

```bash
**Create**
.github/workflows/deploy.yml
```

```bash
name: Laravel Blue-Green Deployment

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: 43.xxx.xxx.xxx
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}

          script: |
            set -e

            APP_DIR="/var/www/laravel-project"

            CURRENT=$(readlink -f "$APP_DIR/current")

            if [ "$CURRENT" = "$APP_DIR/blue" ]; then
              TARGET="green"
            else
              TARGET="blue"
            fi

            echo "Current : $CURRENT"
            echo "Deploy Target : $TARGET"

            rm -rf "$APP_DIR/$TARGET"

            git clone https://github.com/Ismailcse98/blue-green-zero-downtime-deployment.git "$APP_DIR/$TARGET"

            cd "$APP_DIR/$TARGET"

            composer install --no-dev --prefer-dist --optimize-autoloader

            ln -sfn "$APP_DIR/shared/.env" .env

            rm -rf storage
            ln -sfn "$APP_DIR/shared/storage" storage

            rm -rf bootstrap/cache
            ln -sfn "$APP_DIR/shared/bootstrap/cache" bootstrap/cache

            php artisan migrate --force

            php artisan config:cache
            php artisan route:cache
            php artisan view:cache
            php artisan optimize

            ln -sfn "$APP_DIR/$TARGET" "$APP_DIR/current"

            sudo systemctl reload php8.2-fpm
            sudo systemctl reload nginx

            echo "Deployment Successful"
```

## Step 4: Deployment Commands

```bash
git add .
git commit -m "Deploy"
git push origin main
```

# Production Ready Script

```bash
git fetch
git checkout
git reset --hard origin/main
```

# Conclusion

This repository demonstrates a complete production deployment workflow for Laravel applications using AWS EC2, Blue-Green Deployment, and GitHub Actions CI/CD. It is intended as a practical reference for building reliable, automated, and production-ready Laravel deployments.
