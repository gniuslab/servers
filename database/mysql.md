
# MYSQL 8
  
Instalación y configuración de base de datos MYSQL CentOS.
  
## Content Table

- [Install](#install) 
- [Notes](#notes)   

## Install
> SSH into the server running your HTTP website as a user with sudo privileges.
> All packages must be installed from the "root" user

> Este docuemento esta enfocado en instalar solo la base de datos, si necesita instalar un servidor web puede ver la documentación en el siguiente enlace: [Install Nginx + PHP](../nginx/server-php.md)
1. Setup Yum repository

    Execute the following command to enable MySQL yum repository on CentOS:
	```
	rpm -Uvh https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm
	``` 
2. Install MySQL 8 Community Server
    
    since the MySQL yum repository has multiple repositories configuration for multiple MySQL versions, you need to disable all repositories in mysql repo file:

	```
	sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/mysql-community.repo
	```
    And execute the following command to install MySQL 8:
    ```
    yum --enablerepo=mysql80-community install mysql-community-server
    ```

3. Start MySQL Service
	Use this command to start mysql service:
	```
    service mysqld start
    ```
4. Show the default password for root user

	Copie la contraseña que genera mysql en un lugar seguro, para poder acceder y cambiarla
    ```
    grep "A temporary password" /var/log/mysqld.log
    ```
5. MySQL Secure Installation
    Ejecute este comando para ingresar la contraseña del usuario root
	```
	mysql_secure_installation
	```

    You will need to enter the new password for the root‘s account twice. It will prompt for some questions, it is recommended to type yes (y):
    - Remove anonymous users? (Press y|Y for Yes, any other key for No) : y

    - Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y

    - Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y

    - Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
	
6. Restart and enable the MySQL service
	```
	service mysqld restart
	```
    and autostart mysql service on system’s startup:
    ```
	chkconfig mysqld on
	```
7. Connect to MySQL
    1. Use this command to connect to MySQL server:
        ```
        mysql -u root -p
        ```
    2. Show all databases
        ```
        mysql> show databases;
        ```
        Here is the output:
        ```
        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | mysql              |
        | performance_schema |
        | sys                |
        +--------------------+
        4 rows in set (0.05 sec)
        ```
    3. Create New Database:
        ```
        mysql> create database example_db;
        ```
    4. Exit Mysql Console:
        ```
        mysql> exit;
        ```
## Notes
> Si necesitas conectar la base de datos de manera remota tiene que crear un usuario con los privilegios para poder acceder a ella, ya que el usuario root solo está permitido para acceso local

> Para hacer eso puedes seguir estos comandos desde la console de mysql:
```
mysql> CREATE USER 'deploy'@'localhost' IDENTIFIED BY 'some_pass';
```

```
mysql> GRANT ALL PRIVILEGES ON *.* TO'deploy'@'localhost' WITH GRANT OPTION;
```

```
mysql> CREATE USER 'deploy'@'%' IDENTIFIED BY 'some_pass';
```

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'deploy'@'%' WITH GRANT OPTION;
```
> Cambiar el 'some_pass' por la contraseña de preferencia
## Contributing
  
- Abel Rodríguez [@gniuslab](https://github.com/gniuslab)
- Juan de León [@juanrdlo](https://github.com/juanrdlo)
