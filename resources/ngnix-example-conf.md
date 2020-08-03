# Ngnix Config example.conf

1. Edit example.conf file `nano /etc/nginx/conf.d/example.conf`:

    ```
    server {
        listen       80;
        server_name example.com;

        root /usr/share/nginx/html;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.php index.html index.htm;

        charset utf-8;      

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }
        location = /sitemap.xml  { access_log off; log_not_found off; }

        error_page   404 500 502 503 504  /index.php;

        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }
    ```
2. Restart Nginx
	```
	systemctl restart nginx.service
	```