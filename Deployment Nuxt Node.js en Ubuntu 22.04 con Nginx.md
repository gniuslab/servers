# Guía de Deployment Nuxt/Node.js en Ubuntu 22.04 con Nginx

## Instalación de Node.js

### Node.js 20 LTS (Recomendado)
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
apt install nodejs -y
node -v
npm -v
```

### Node.js 18 LTS
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
apt install nodejs -y
```

### Node.js 16 LTS
```bash
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
apt install nodejs -y
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

## Configuración de Permisos NPM

```bash
mkdir -p /var/www/.npm
chown -R www-data:www-data /var/www/.npm
```

## Instalar Dependencias

```bash
sudo -u www-data npm install --prefix /var/www/dominio.com/html
```

## Configuración de Entorno

```bash
sudo -u www-data cp .env.example .env
sudo -u www-data nano .env
```

## Build de Producción

```bash
sudo -u www-data npm run build
```

## Instalación de PM2

```bash
npm install -g pm2
```

## Iniciar Aplicación con PM2

```bash
cd /var/www/dominio.com/html
mkdir -p logs
pm2 start /usr/local/lsws/dominio.com/html/.output/server/index.mjs --name dominio.com
pm2 startup
pm2 save
```


## Configuración de Nginx

```bash
cat > /etc/nginx/sites-available/dominio.com

```
server {
    listen 80;
    server_name dominio.com www.dominio.com;

    client_max_body_size 100M;

    location /_nuxt/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
        
        add_header Cache-Control "public, max-age=31536000, immutable";
    }

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

## Activar Sitio

```bash
ln -s /etc/nginx/sites-available/dominio.com /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

## Certificado SSL con Certbot

```bash
apt install certbot python3-certbot-nginx -y
certbot --nginx -d dominio.com -d www.dominio.com
```

## Comandos Útiles PM2

```bash
pm2 list
pm2 logs nombre-app
pm2 logs nombre-app --lines 100
pm2 restart nombre-app
pm2 reload nombre-app
pm2 stop nombre-app
pm2 delete nombre-app
pm2 monit
pm2 flush
pm2 save
pm2 resurrect
pm2 unstartup
```

## Deploy de Actualizaciones

```bash
cd /var/www/dominio.com/html
sudo -u www-data git pull
sudo -u www-data npm install
sudo -u www-data npm run build
pm2 restart nombre-app
```

## Verificar Estado

```bash
netstat -tlnp | grep 3000
pm2 status
pm2 info nombre-app
systemctl status nginx
curl -I http://localhost:3000
```

## Configuración Avanzada de Nginx (Opcional)

### Con Cache
```bash
cat > /etc/nginx/sites-available/dominio.com << 'EOF'
proxy_cache_path /var/cache/nginx/nuxt levels=1:2 keys_zone=nuxt_cache:10m max_size=1g inactive=60m use_temp_path=off;

server {
    listen 80;
    server_name dominio.com www.dominio.com;

    client_max_body_size 100M;

    location /_nuxt/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        proxy_cache nuxt_cache;
        proxy_cache_valid 200 60m;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_background_update on;
        proxy_cache_lock on;
        
        add_header X-Cache-Status $upstream_cache_status;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
EOF

mkdir -p /var/cache/nginx/nuxt
chown -R www-data:www-data /var/cache/nginx/nuxt
```

## Troubleshooting

### Ver logs en tiempo real
```bash
pm2 logs nombre-app --lines 50
tail -f /var/www/dominio.com/html/logs/pm2-error.log
tail -f /var/log/nginx/error.log
```

### Limpiar cache de PM2
```bash
pm2 flush
```

### Reiniciar todo
```bash
pm2 restart all
systemctl restart nginx
```

### Verificar puerto en uso
```bash
lsof -i :3000
netstat -tulpn | grep :3000
```

## Optimización para Producción

### Variables de entorno recomendadas (.env)
```env
NODE_ENV=production
NITRO_PRESET=node-server
NITRO_PORT=3000
NITRO_HOST=0.0.0.0
```

### Build optimizado
```bash
sudo -u www-data npm run build -- --preset=node-server
```
