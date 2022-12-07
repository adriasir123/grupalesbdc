# phpMyAdmin

## Apache

Instalo:

```shell
sudo apt install apache2
```

Compruebo que funciona:

```shell
vagrant@servidormariadb:~$ sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-10-24 12:17:52 UTC; 16s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 1584 (apache2)
      Tasks: 55 (limit: 1128)
     Memory: 8.8M
        CPU: 23ms
     CGroup: /system.slice/apache2.service
             ├─1584 /usr/sbin/apache2 -k start
             ├─1586 /usr/sbin/apache2 -k start
             └─1587 /usr/sbin/apache2 -k start

Oct 24 12:17:52 servidormariadb systemd[1]: Starting The Apache HTTP Server...
Oct 24 12:17:52 servidormariadb apachectl[1583]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerNam>
Oct 24 12:17:52 servidormariadb systemd[1]: Started The Apache HTTP Server.
```

## PHP

Instalo los paquetes que necesitaremos:

```shell
sudo apt install wget php php-cgi php-mysqli php-pear php-mbstring libapache2-mod-php php-common php-phpseclib php-mysql
```

## En MariaDB

Login como root:

```shell
sudo mariadb
```

Creo el usuario:

```shell
CREATE USER 'pma-admin'@localhost IDENTIFIED BY '1234';
```

Le doy privilegios:

```shell
GRANT ALL PRIVILEGES ON *.* TO 'pma-admin'@localhost IDENTIFIED BY '1234';
FLUSH PRIVILEGES;
```

## Descarga

```shell
wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
tar xvf phpMyAdmin-latest-all-languages.tar.gz
```

## Instalación

Paso la aplicación al DocumentRoot por defecto:

```shell
sudo mv phpMyAdmin-5.2.0-all-languages /var/www/html/phpmyadmin
```

Creo el fichero de configuración:

```shell
cd /var/www/html
cp phpmyadmin/config.sample.inc.php phpmyadmin/config.inc.php
```

Creo el directorio tmp:

```shell
mkdir /var/www/html/phpmyadmin/tmp
```

Genero la clave que usaremos para phpMyadmin:

```shell
vagrant@servidormariadb:/var/www/html$ openssl rand -base64 32
knLvNBNeqZDIoJxYS4v+RnLqj8mzNxmZWdJSr1aClPk=
```

Modifico la siguiente línea en `/var/www/html/phpmyadmin/config.inc.php` con la clave generada, recortada por el final para que sea exactamente de 32 caracteres:

```shell
$cfg['blowfish_secret'] = 'knLvNBNeqZDIoJxYS4v+RnLqj8mzNxmZ'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```

Añado también la siguiente línea en `/var/www/html/phpmyadmin/config.inc.php`:

```shell
$cfg['TempDir'] = '/var/www/html/phpmyadmin/tmp';
```

Cambio los propietarios:

```shell
sudo chown -R www-data:www-data /var/www/html/phpmyadmin
```

Creo el siguiente fichero conf:

```shell
sudo touch /etc/apache2/conf-available/phpmyadmin.conf
```

Le añado el siguiente contenido:

```shell
Alias /phpmyadmin /var/www/html/phpmyadmin

<Directory /var/www/html/phpmyadmin/>
   AddDefaultCharset UTF-8
   <IfModule mod_authz_core.c>
          <RequireAny>
      Require all granted
     </RequireAny>
   </IfModule>
</Directory>

<Directory /var/www/html/phpmyadmin/setup/>
   <IfModule mod_authz_core.c>
     <RequireAny>
       Require all granted
     </RequireAny>
   </IfModule>
</Directory>
```

Lo habilito:

```shell
sudo a2enconf phpmyadmin.conf
```

Reinicio Apache:

```shell
sudo systemctl restart apache2
```

## Acceso

Entro y accedo con el usuario que creamos anteriormente:

![pmaacceso](https://i.imgur.com/DDaOoDk.png)

Al principio tendremos este warning, clicamos en "Find out why":

![findoutwhy](https://i.imgur.com/uheEOjU.png)

Tenemos que clicar "Create" para crear la base de datos que necesita phpMyAdmin:

![create](https://i.imgur.com/ADXP0Od.png)

Vemos que todas las salidas son "OK" y que la base de datos se ha creado:

![oksalidas](https://i.imgur.com/1mOXul6.png)

## Uso

A partir de ahora ya podríamos administrar nuestras bases de datos con esta herramienta.

Por ejemplo, podemos ver las tablas que tiene `bibliofilos`:

![tablasbibliofilos](https://i.imgur.com/1WdyoEm.png)

Podemos ver también todos los registros que tiene cada tabla:

![bibliotecas](https://i.imgur.com/YvkPHiA.png)

![libros](https://i.imgur.com/uWWOi8A.png)

![trabajadores](https://i.imgur.com/gqGZj94.png)

Y como es lógico, a partir de ahora podríamos hacer todas las operaciones de administración sobre MariaDB que quisiéramos.

## Prueba desde `clientemariadb`

![pruebaclientemariadb](https://i.imgur.com/DcvYMkN.png)

## Bibliografía

<https://www.how2shout.com/linux/how-to-install-phpmyadmin-on-debian-11-bullseye-apache/>
