# Guía Deployment Laravel/PHP en Ubuntu 22.04 con Nginx

## 1. Instalar PHP y extensiones

### PHP 8.4
add-apt-repository ppa:ondrej/php -y
apt update
apt install php8.4-fpm php8.4-mysql php8.4-mbstring php8.4-xml php8.4-bcmath php8.4-curl php8.4-zip php8.4-gd php8.4-intl php8.4-redis -y

### PHP 8.2
apt install php8.2-fpm php8.2-mysql php8.2-mbstring php8.2-xml php8.2-bcmath php8.2-curl php8.2-zip php8.2-gd php8.2-intl php8.2-redis -y

### PHP 7.4
apt install php7.4-fpm php7.4-mysql php7.4-mbstring php7.4-xml php7.4-bcmath php7.4-curl php7.4-zip php7.4-gd php7.4-intl php7.4-redis -y

## 2. Instalar Composer

curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer

## 3. Crear estructura de directorios

mkdir -p /var/www/dominio.com/html
chown -R www-data:www-data /var/www/dominio.com

## 4. Clonar proyecto

cd /var/www/dominio.com/html
sudo -u www-data git clone https://github.com/usuario/repo.git .

## 5. Instalar dependencias

sudo -u www-data composer install --no-dev --optimize-autoloader

## 6. Configurar permisos Laravel

chown -R www-data:www-data /var/www/dominio.com/html
chmod -R 755 /var/www/dominio.com/html
chmod -R 775 /var/www/dominio.com/html/storage
chmod -R 775 /var/www/dominio.com/html/bootstrap/cache

## 7. Configurar .env

sudo -u www-data cp .env.example .env
sudo -u www-data php artisan key:generate
nano .env

## 8. Ejecutar migraciones

sudo -u www-data php artisan migrate --force
sudo -u www-data php artisan config:cache
sudo -u www-data php artisan route:cache
sudo -u www-data php artisan view:cache

## 9. Configurar Nginx - PHP 8.4

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

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

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

## 10. Configurar Nginx - PHP 8.2

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

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

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

## 11. Configurar Nginx - PHP 7.4

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

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

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

## 12. Activar sitio

ln -s /etc/nginx/sites-available/dominio.com /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx

## 13. Verificar PHP-FPM

systemctl status php8.4-fpm
systemctl status php8.2-fpm
systemctl status php7.4-fpm

## 14. SSL con Certbot

apt install certbot python3-certbot-nginx -y
certbot --nginx -d dominio.com -d www.dominio.com

## 15. Configurar Queue Worker (Opcional)

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

## 16. Configurar Cron

crontab -e -u www-data

* * * * * cd /var/www/dominio.com/html && php artisan schedule:run >> /dev/null 2>&1

## Deploy actualizaciones

cd /var/www/dominio.com/html
sudo -u www-data git pull
sudo -u www-data composer install --no-dev --optimize-autoloader
sudo -u www-data php artisan migrate --force
sudo -u www-data php artisan config:cache
sudo -u www-data php artisan route:cache
sudo -u www-data php artisan view:cache
supervisorctl restart laravel-worker:*

## Cambiar versión PHP en servidor existente

update-alternatives --config php
systemctl restart php8.4-fpm
systemctl restart nginx

## Optimización producción

php artisan optimize
php artisan event:cache
php artisan icons:cache

## Comandos útiles

php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
composer dump-autoload
supervisorctl status
tail -f storage/logs/laravel.log
