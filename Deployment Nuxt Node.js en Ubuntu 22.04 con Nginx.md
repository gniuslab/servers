# Guía Deployment Nuxt/Node.js en Ubuntu 22.04 con Nginx

## 1. Crear estructura de directorios

mkdir -p /var/www/dominio.com/html
chown -R www-data:www-data /var/www/dominio.com

## 2. Clonar proyecto

cd /var/www/dominio.com/html
sudo -u www-data git clone https://github.com/usuario/repo.git .

## 3. Configurar permisos npm

mkdir -p /var/www/.npm
chown -R www-data:www-data /var/www/.npm

## 4. Instalar dependencias

sudo -u www-data npm install --prefix /var/www/dominio.com/html

## 5. Configurar .env

sudo -u www-data cp .env.example .env
sudo -u www-data nano .env

## 6. Build producción

sudo -u www-data npm run build

## 7. Instalar PM2 globalmente

npm install -g pm2

## 8. Crear ecosystem.config.js

cat > /var/www/dominio.com/html/ecosystem.config.js << 'EOF'
module.exports = {
  apps: [{
    name: 'nombre-app',
    script: '.output/server/index.mjs',
    user: 'www-data',
    cwd: '/var/www/dominio.com/html',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
}
EOF

## 9. Iniciar con PM2

cd /var/www/dominio.com/html
pm2 start ecosystem.config.js
pm2 startup
pm2 save

## 10. Configurar Nginx

cat > /etc/nginx/sites-available/dominio.com << 'EOF'
server {
    listen 80;
    server_name dominio.com www.dominio.com;

    location /_nuxt/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
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
    }
}
EOF

## 11. Activar sitio

ln -s /etc/nginx/sites-available/dominio.com /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx

## 12. SSL con Certbot (Opcional)

apt install certbot python3-certbot-nginx -y
certbot --nginx -d dominio.com -d www.dominio.com

## Comandos útiles PM2

pm2 list
pm2 logs nombre-app
pm2 restart nombre-app
pm2 stop nombre-app
pm2 delete nombre-app

## Deploy actualizaciones

cd /var/www/dominio.com/html
sudo -u www-data git pull
sudo -u www-data npm install
sudo -u www-data npm run build
pm2 restart nombre-app

## Verificar estado

netstat -tlnp | grep 3000
pm2 status
systemctl status nginx
