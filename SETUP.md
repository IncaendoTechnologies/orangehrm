# OrangeHRM — Setup & Deployment Guide

## Requirements

- PHP 8.0–8.4
- MySQL 5.7+ or MariaDB 10.3+
- Composer 2.x
- Node.js 18+ and Yarn 4.x
- Apache or Nginx (for production) / PHP built-in server (for local dev)

---

## Local Development Setup

### 1. Clone the repository

```bash
git clone <repository-url>
cd orangehrm
```

### 2. Configure the database

Copy the sample config and fill in your credentials:

```bash
cp lib/confs/Conf.php.sample lib/confs/Conf.php
```

Edit `lib/confs/Conf.php` and set your DB host, name, user, and password.

> **Note:** `lib/confs/Conf.php` is gitignored and must never be committed.

### 3. Create the database

```bash
mysql -u root -p -e "CREATE DATABASE your_db_name CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

Import an existing dump if available:

```bash
mysql -u root -p your_db_name < backup.sql
```

### 4. Install PHP dependencies

```bash
cd src && composer install
```

### 5. Install and build the frontend

```bash
cd src/client && yarn install && yarn build
```

### 6. Set permissions

```bash
chmod -R 775 src/cache src/log src/config
```

### 7. Start local server

```bash
php -S localhost:8080 -t web web/router.php
```

Open `http://localhost:8080` in your browser.

---

## Production Deployment

### Apache / Nginx

Point the document root to the `web/` directory.

**Apache** — enable `mod_rewrite` and add to your vhost:

```apache
<VirtualHost *:80>
    ServerName yourdomain.com
    DocumentRoot /var/www/orangehrm/web

    <Directory /var/www/orangehrm/web>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

**Nginx:**

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/orangehrm/web;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### Deploy steps (on server)

```bash
# 1. Pull latest code
git pull origin main

# 2. Install PHP dependencies (no dev packages)
cd src && composer install --no-dev --optimize-autoloader

# 3. Build frontend
cd src/client && yarn install && yarn build

# 4. Clear cache
rm -rf src/cache/*

# 5. Set permissions
chmod -R 775 src/cache src/log src/config
chown -R www-data:www-data src/cache src/log src/config
```

> **Important:** Manually copy `lib/confs/Conf.php` to the server with production DB credentials. Do not commit this file.

---

## Reset Admin Password

If you need to reset a user password:

```bash
# Generate a bcrypt hash
php -r "echo password_hash('YourNewPassword', PASSWORD_BCRYPT, ['cost' => 12]);"

# Update in DB
mysql -h 127.0.0.1 -u root -p your_db_name -e \
  "UPDATE ohrm_user SET user_password = '<hash>' WHERE user_name = 'your@email.com';"
```

---

## Running Tests

```bash
# PHP unit tests
cd src && composer test

# Frontend unit tests
cd src/client && yarn test:unit

# E2E tests (Cypress)
cd src/test/functional && yarn test
```
