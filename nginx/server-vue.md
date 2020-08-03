
# Nginx + Vue
Instalación y configuración de servidor ngnix de prueba y producción CentOS.

Lista de paquetes instalados:
- nginx
- node.js
- npm
- vue
- nano
- htop
- wget
- epel-release
- certbot
- git
  
## Content Table

- [Install](#install) 
- [Config](#config) 
- [Notes](#notes)   

## Install
> SSH into the server running your HTTP website as a user with sudo privileges.
> All packages must be installed from the "root" user

1. Actualización del servidor
	```
	yum update -y; yum install nano -y; yum install wget -y; yum install epel-release -y; yum install htop -y; yum install git -y; yum install nodejs -y; yum install unzip -y
	``` 
2. Install Nginx - Start, Add & Restart
	```
	yum install nginx -y; systemctl start nginx.service; systemctl enable nginx; systemctl restart nginx.service
	```
3. Disable SELINUX
	1.  Open the  `/etc/selinux/config`:
	    ``` nano /etc/selinux/config ```
	    
	    ```ini
	    # This file controls the state of SELinux on the system.
	    # SELINUX= can take one of these three values:
	    #       enforcing - SELinux security policy is enforced.
	    #       permissive - SELinux prints warnings instead of enforcing.
	    #       disabled - No SELinux policy is loaded.
	    SELINUX=disabled
	    # SELINUXTYPE= can take one of these two values:
	    #       targeted - Targeted processes are protected,
	    #       mls - Multi Level Security protection.
	    SELINUXTYPE=targeted
	    ```
	    
	2.  Save the file and reboot your CentOS system with:
	    
	    ```
	    sudo shutdown -r now
	    ```
	    
	3.  Once the system boots up, verify the change with the  `sestatus`  command:
	    
	    ```
	    sestatus
	    ```
	    
	    The output should look like this:
	    
	    ```
        SELinux status: disabled
        ```
4. Install Vue

    ```
    npm install -g @vue/cli
    ```

8. Install SSL Letsencrypt [Source Guide](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx)
    1. Install Certbot
    Run this command on the command line on the machine to install Certbot.

        ``` 
        sudo yum install certbot python2-certbot-nginx 
        ```
    2. Install your certificates...
    Run this command to get a certificate and have Certbot edit your Nginx configuration automatically to serve it, turning on HTTPS access in a single step.

        ```
        sudo certbot --nginx
        ```

    3. Set up automatic renewal

        ```
        echo "0 0,12 * * * root python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
        ```
## Config
3. Config nginx
    - [Global Config](../resources/nginx.conf)
    - [Example Nginx Config](../resources/ngnix-example-vue-conf.md)
## Notes
- Esta configuración esta pensada de manera general pero es mejor optimizar según los casos
- Se recomienda crear un archivo de configuración nuevo en la ruta: `/etc/nginx/conf.d/example.com.conf`
- Default Nginx Directory
    ```
    /usr/share/nginx/html
    ```
## Contributing
  
- Abel Rodríguez [@gniuslab](https://github.com/gniuslab)
- Juan de León [@juanrdlo](https://github.com/juanrdlo)
