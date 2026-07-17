## Zero Downtime Deployment with CI CD

zero downtime deployment project

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

```
3306 → Private Network Only
```

## Step 13: Check Version

```bash
mysql --version
```






## Step 15: Create & Open custom laravel.conf file

```bash
sudo nano /etc/nginx/sites-available/laravel
```

## Step 7: Enter this code in laravel.conf file

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

## Step 16: Test the Configuration

Before restarting Nginx, verify the configuration.

```bash
sudo nginx -t
```

Expected output:

```text
syntax is ok
test is successful
```

## Step 8: Reload Nginx
