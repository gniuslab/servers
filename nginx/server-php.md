
# Nginx + PHP
  
Instalación y configuración de servidor ngnix de prueba y producción CentOS.
  
## Content Table

- [Install](#install)
- [Config](#config)
- [Notes](#notes)

## Install
> SSH into the server running your HTTP website as a user with sudo privileges.
> All packages must be installed from the "root" user
1. Configuring a Static IP
	1. Open the configuration file for your network interface.

		```
		vi /etc/sysconfig/network-scripts/ifcfg-eth0
		```

	2. Add the following settings to the file:
		```
		ONBOOT=yes
		DHCP=static
		```
	3. Restarting the network interface
		```
		ifup eth0
		```
2. Disable Firewall
	1. First, stop the FirewallD service with

	```
	sudo systemctl stop firewalld
	```

	2. Disable the FirewallD service to start automatically on system boot:

	```
	sudo systemctl disable firewalld
	```
	
	3. Mask the FirewallD service which will prevent the firewall from being started by other services:

	````
	sudo systemctl mask --now firewalld
	```

1. Actualización del servidor
	```
	yum update -y; yum install nano -y; yum install wget -y; yum install epel-release -y; yum install htop -y; yum install git -y; yum install nodejs -y; yum install unzip -y
	```
2. Open port 80 to nginx

	```
	firewall-cmd --permanent --zone=public --add-port=80/tcp
	```

	```
	firewall-cmd --reload
	```

4. Install Nginx - Start, Add & Restart
	```
	yum install nginx -y; systemctl start nginx.service; systemctl enable nginx; systemctl restart nginx.service
	```
5. Disable SELINUX
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
5. Install FPM Complements
	1. Download Pacakages
		```
		wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm
		yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm -y
		```
	2. Install remi-release package
		```
		rpm -Uvh remi-release-7.rpm
		```
	3. Install yum-utils package
		```
		yum install yum-utils -y
		```
6. Install PHP-FPM 7.4
	```
	yum search yum-config-manager
	```
	```
	yum-config-manager --enable remi-php74
	```
	```
	yum --enablerepo=remi,remi-php74 install php-fpm php-common -y
	```
	Install PHP packages
	```
	yum --enablerepo=remi,remi-php74 install php-opcache php-pecl-apcu php-cli php-pear php-pdo php-mysqlnd php-pgsql php-pecl-mongodb php-pecl-redis php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml php-intl php-json php-gettext php-curl php-date php-sqlite3 php-mysqli php-openssl php-zip -y
	```
	Replace default apache config `/etc/php-fpm.d/www.conf`:
	
   ``` 
   nano /etc/php-fpm.d/www.conf 
   ```
		
   ```
	user = nginx
	group = nginx
	listen = /var/run/php-fpm/php-fpm.sock
	listen.owner = nginx
	listen.group = nginx
   ```
7. Restart PHP
	```
	systemctl start php-fpm.service; systemctl enable php-fpm.service
	```
8. Install Composer
    1. Once PHP CLI is installed, download the Composer installer script with:
        ```
        php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
        ```
    2. To verify the data integrity
        ```
        HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
        ```
    3. To verify that the installation script is not corrupted run the following command:
        ```
        php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
        ```
        If the hashes match, the following message will be shown:

        ```
        Installer verified
        ```
    4. Run the following command to install Composer in the /usr/local/bin directory:
        ```
        sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
        ```
        The composer is installed as a system-wide command and it will be available for all users.
        ```
        All settings correct for using Composer
        Downloading...

        Composer (version 1.8.5) successfully installed to: /usr/local/bin/composer
        Use it: php /usr/local/bin/composer
        ```
    5. The last step is to verify the installation:
        ```
        composer
        ```

9. Install SSL Letsencrypt [Source Guide](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx) (Optional)
    1. Install Certbot
    Run this command on the command line on the machine to install Certbot.

        ``` 
        sudo yum install certbot python2-certbot-nginx -y
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
10. Add new user deploy (Optional)
	- Add new user
		```
		adduser deploy
		```
	- Set Password
		```
		passwd deploy
		```
	- Configurar el servicio de ssh para que el acceso a root no este habilitado
		```
		nano /etc/ssh/sshd_config
		```
		Descomentar la linea:
		```
		PermitRootLogin no
		```
		Restart SSH Service
		```
		service sshd restart
		```

## Config
1. Change permissions session folder to enable session writing in PHP
	```
	chown -R nginx:nginx /var/lib/php/session
	```
2. Replace [php.ini](../resources/php-prod.ini)

3. Config nginx
	Config Route
    - [Global Config](../resources/nginx.conf)
		```
		nano /etc/nginx/nginx.conf
		```
    - [Example Nginx Config](../resources/ngnix-example-conf.md)
		```
		nano /etc/nginx/conf.d/example.com.conf
		```
## Notes
- Esta configuración esta pensada de manera general pero es mejor optimizar según los casos
- Default Nginx Directory
    ```
    /usr/share/nginx/html
    ```
- Create site directory
	```
    mkdir /usr/share/nginx/html/site
    ```
- Add permissions
	```
    chmod -R 775 /usr/share/nginx/html/site
    ```
## Contributing
  
- Abel Rodríguez [@gniuslab](https://github.com/gniuslab)
- Juan de León [@juanrdlo](https://github.com/juanrdlo)
