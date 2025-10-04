# Guía de Deployment Laravel/PHP en Ubuntu 22.04 con Nginx

## Instalación de MySQL

### Instalar MySQL 8.0
```bash
apt update
apt install mysql-server -y
systemctl start mysql
systemctl enable mysql
```

### Configuración Inicial Segura
```bash
mysql_secure_installation
```

### Crear Usuario y Base de Datos
```bash
mysql -u root -p
```

```sql
CREATE DATABASE nombre_bd CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'usuario'@'localhost' IDENTIFIED BY 'password_seguro';
GRANT ALL PRIVILEGES ON nombre_bd.* TO 'usuario'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### Configuración de Acceso Remoto (Opcional)
```bash
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Cambiar:
```
bind-address = 0.0.0.0
```

Crear usuario remoto:
```sql
CREATE USER 'usuario'@'%' IDENTIFIED BY 'password_seguro';
GRANT ALL PRIVILEGES ON nombre_bd.* TO 'usuario'@'%';
FLUSH PRIVILEGES;
```

Reiniciar:
```bash
systemctl restart mysql
```

### Comandos Útiles MySQL

```bash
systemctl status mysql
systemctl restart mysql
systemctl stop mysql
systemctl start mysql
```

```sql
SHOW DATABASES;
USE nombre_bd;
SHOW TABLES;
DESCRIBE nombre_tabla;
SHOW PROCESSLIST;
SHOW VARIABLES LIKE 'max_connections';
SELECT USER, HOST FROM mysql.user;
DROP DATABASE nombre_bd;
DROP USER 'usuario'@'localhost';
```

### Backup y Restore

Backup:
```bash
mysqldump -u usuario -p nombre_bd > backup.sql
mysqldump -u usuario -p --all-databases > backup_all.sql
```

Restore:
```bash
mysql -u usuario -p nombre_bd < backup.sql
```

### Optimización MySQL para Laravel

```bash
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

```ini
[mysqld]
max_connections = 200
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
query_cache_size = 0
query_cache_type = 0
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
```

```bash
systemctl restart mysql
```

## Instalación de PHP

### PHP 8.4
```bash
add-apt-repository ppa:ondrej/php -y
apt update
apt install php8.4-fpm php8.4-mysql php8.4-mbstring php8.4-xml php8.4-bcmath php8.4-curl php8.4-zip php8.4-gd php8.4-intl php8.4-redis -y
```

### PHP 8.2
```bash
apt install php8.2-fpm php8.2-mysql php8.2-mbstring php8.2-xml php8.2-bcmath php8.2-curl php8.2-zip php8.2-gd php8.2-intl php8.2-redis -y
```

### PHP 7.4
```bash
apt install php7.4-fpm php7.4-mysql php7.4-mbstring php7.4-xml php7.4-bcmath php7.4-curl php7.4-zip php7.4-gd php7.4-intl php7.4-redis -y
```

## Instalación de Composer

```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer
```

## Estructura de Directorios

```bash
mkdir -p /var/www/dominio.com/html
chown -R www-data:www-data /var/www/dominio.com
```

## Clonar Proyecto

```bash
cd /var/www/dominio.com/html
sudo -u www-data git clone https://github.com/usuario/repo.git .
```

## Instalar Dependencias

```bash
sudo -u www-data composer install --no-dev --optimize-autoloader
```

## Configuración de Permisos Laravel

```bash
chown -R www-data:www-data /var/www/dominio.com/html
chmod -R 755 /var/www/dominio.com/html
chmod -R 775 /var/www/dominio.com/html/storage
chmod -R 775 /var/www/dominio.com/html/bootstrap/cache
```

## Configuración de Entorno

```bash
sudo -u www-data cp .env.example .env
sudo -u www-data php artisan key:generate
nano .env
```

## Ejecutar Migraciones

```bash
sudo -u www-data php artisan migrate --force
sudo -u www-data php artisan config:cache
sudo -u www-data php artisan route:cache
sudo -u www-data php artisan view:cache
```

## Configuración de Nginx

### Para PHP 8.4
```bash
cat > /etc/nginx/sites-available/dominio.com << 'EOF'
server {
    listen 80;
    server_name dominio.com www.dominio.com;
    root /var/www/dominio.com/html/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location = /robots.txt {
        access_log off;
        log_not_found off;
    }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF
```

### Para PHP 8.2
```bash
cat > /etc/nginx/sites-available/dominio.com << 'EOF'
server {
    listen 80;
    server_name dominio.com www.dominio.com;
    root /var/www/dominio.com/html/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location = /robots.txt {
        access_log off;
        log_not_found off;
    }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF
```

### Para PHP 7.4
```bash
cat > /etc/nginx/sites-available/dominio.com << 'EOF'
server {
    listen 80;
    server_name dominio.com www.dominio.com;
    root /var/www/dominio.com/html/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location = /robots.txt {
        access_log off;
        log_not_found off;
    }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF
```

## Activar Sitio

```bash
ln -s /etc/nginx/sites-available/dominio.com /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

## Verificar PHP-FPM

```bash
systemctl status php8.4-fpm
systemctl status php8.2-fpm
systemctl status php7.4-fpm
```

## Certificado SSL con Certbot

```bash
apt install certbot python3-certbot-nginx -y
certbot --nginx -d dominio.com -d www.dominio.com
```

## Configuración de Queue Worker (Opcional)

```bash
cat > /etc/supervisor/conf.d/laravel-worker.conf << 'EOF'
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/dominio.com/html/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/dominio.com/html/storage/logs/worker.log
stopwaitsecs=3600
EOF

supervisorctl reread
supervisorctl update
supervisorctl start laravel-worker:*
```

## Configuración de Cron

```bash
crontab -e -u www-data
```

Agregar:
```
* * * * * cd /var/www/dominio.com/html && php artisan schedule:run >> /dev/null 2>&1
```

## Deploy de Actualizaciones

```bash
cd /var/www/dominio.com/html
sudo -u www-data git pull
sudo -u www-data composer install --no-dev --optimize-autoloader
sudo -u www-data php artisan migrate --force
sudo -u www-data php artisan config:cache
sudo -u www-data php artisan route:cache
sudo -u www-data php artisan view:cache
supervisorctl restart laravel-worker:*
```

## Cambiar Versión de PHP

```bash
update-alternatives --config php
systemctl restart php8.4-fpm
systemctl restart nginx
```

## Optimización para Producción

```bash
php artisan optimize
php artisan event:cache
php artisan icons:cache
```

## Comandos Útiles

```bash
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
composer dump-autoload
supervisorctl status
tail -f storage/logs/laravel.log
```
