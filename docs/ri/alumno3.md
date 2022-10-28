# Alumno 3. Arantxa Fernández Morató

Tras la instalación de cada servidor, debe crearse una base de datos con al menos tres tablas o colecciones y poblarse de datos adecuadamente. Debe crearse un usuario y dotarlo de los privilegios necesarios para acceder remotamente a los datos.
Los clientes deben estar siempre en máquinas diferentes de los respectivos servidores a los que acceden.
Se documentará todo el proceso de configuración de los servidores.
Se aportarán pruebas del funcionamiento remoto de cada uno de los clientes.
Se aportará el código de las aplicaciones realizadas y prueba de funcionamiento de las mismas.

```s
    • Instalación de un servidor Oracle 19c sobre Debian, otro Postgres, otro MySQL y otro de MongoDB y configuración para permitir el acceso remoto desde la red local.
    • Prueba desde un cliente remoto de SQL*Plus.
    • Realización de una aplicación web en cualquier lenguaje que conecte con el servidor Postgres tras autenticarse y muestre alguna información almacenada en el mismo.
```

## 1. MySQL

### 1.1 Instalación

En el servidor actualizamos los repositorios e instalamos **gnupg** y **wget**, si no estuvieran ya instalados.

`sudo apt update`

`sudo apt install -y gnupg wget`

Utilizando wget descargamos el paquete de mysql del repositorio oficial.

`wget https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb`

Procedemmos a instalar el paquete descargado.

`sudo dpkg -i mysql-apt-config_0.8.22-1_all.deb`

Nos aparecerá una ventana que nos muestra la versión de MySQL que se instalará, nos informa de que las herramientas y conectores están activados y la opción de software de prueba está desactivada. Presionamos *Aceptar* para continuar.

![instalacion-mysql](/img/capturas-arantxa/1.png)

Volvemos a actualizar los repositorios.

`sudo apt update`

Ya podremos instalar MySQL server en Debian 11.

`sudo apt install -y mysql-server`

Durante la instalación nos preguntará la contraseña del usuario administrador root.

![instalacion-mysql2](/img/capturas-arantxa/2.png)

Nos aparece un menje de disponibilidad de un nuevo plugin de autenticación. Presionamos *Acepta*.

![instalacion-mysql3](/img/capturas-arantxa/3.png)

Dejamos seleccionado el plugin de autenticación recomendado.

![instalacion-mysql4](/img/capturas-arantxa/4.png)

Terminada la instalación pasaremos a entrar con el usuario root.

`mysql -u root -p`

![entrando-mysql](/img/capturas-arantxa/5.png)


### 1.2 Creación usuario

Creamos un usuario y contraseña para el administrador de las bases de datos que se creen.

`create user 'admin'@'%' identified by 'admin';`

Le damos todos los privilegios.

`grant all privileges on *.* to admin with grant option;`

`flush privileges;`


### 1.3 Creación base de datos

`create database maravilla;`

Me conecto a la base de datos con el usuario creado anteriormente.

`use maravilla`

![creacion-user-bd](/img/capturas-arantxa/22.png)


### 1.4 Creación tablas e inserción de datos

He creado las tablas 'pelicula', 'actor' y 'pelicula_actor'.

```
create table actor
(
codigo varchar (5),
nombre varchar (15),
apellido varchar (15),
fechanac date,
constraint pk_actor primary key (codigo)
);

create table pelicula
(
codigo varchar (5),
titulo varchar (30),
fecha_estreno date,
puntuacion decimal (3,1),
lengua_original varchar (3),
constraint pk_pelicula primary key (codigo),
constraint puntuacion_ck check (puntuacion between 0 and 10)
);

create table pelicula_actor
(
codigo_actor varchar (5),
codigo_pelicula varchar (5),
tipo_personaje enum ('Principal','Secundario'),
constraint pk_pelicula_actor primary key (codigo_actor,codigo_pelicula),
constraint fk_codigo_actor foreign key (codigo_actor) references actor (codigo),
constraint fk_codigo_pelicula foreign key (codigo_pelicula) references pelicula (codigo)
);
```

Y he añadido los siguientes inserts:

```
insert into pelicula values ('MA001', 'Iron Man', '2008-05-02', 7.9, 'EN');
insert into pelicula values ('MA002', 'Thor', '2011-05-06', 7.0, 'EN');
insert into pelicula values ('MA003', 'Captain America: the first avenger', '2011-07-22', 6.9, 'EN');
insert into pelicula values ('MA004', 'The Avengers', '2012-05-04', 8, 'EN');
insert into pelicula values ('MA005', 'Guardians of the Galaxy', '2014-08-01', 8, 'EN');
insert into pelicula values ('MA006', 'Avengers: Age of Ultron', '2015-05-01', 7.3, 'EN');
insert into pelicula values ('MA007', 'Doctor Strange', '2016-11-04', 7.5, 'EN');
insert into pelicula values ('MA008', 'Spider-Man: Homecoming', '2017-07-07', 7.4, 'EN');
insert into pelicula values ('MA009', 'Black Panther', '2018-02-16', 7.3, 'EN');
insert into pelicula values ('MA010', 'Avengers: Infinity War', '2018-04-27', 8.4, 'EN');
insert into pelicula values ('MA011', 'Avengers: Endgame', '2019-04-26', 8.4, 'EN');

insert into actor values ('001','Robert', 'Downey', '1965-04-04');
insert into actor values ('002','Jon', 'Favreau', '1966-10-19');
insert into actor values ('003','Chris', 'Hemsworth', '1983-08-11');
insert into actor values ('004','Tom', 'Hiddleston', '1981-02-09');
insert into actor values ('005','Natalie', 'Portman', '1981-06-09');
insert into actor values ('006','Chris', 'Evans', '1981-06-13');
insert into actor values ('007','Sebastian', 'Stan', '1982-08-13');
insert into actor values ('008','Jeremy', 'Renner', '1971-01-07');
insert into actor values ('009','Scarlett', 'Johansson', '1984-11-22');
insert into actor values ('010','Mark', 'Ruffalo', '1967-11-22');
insert into actor values ('011','Clark', 'Gregg', '1962-04-02');
insert into actor values ('012','Samuel L.', 'Jackson', '1948-12-21');
insert into actor values ('013','Chris', 'Pratt', '1979-06-21');
insert into actor values ('014','Zoe', 'Saldaña', '1978-06-19');
insert into actor values ('015','Dave', 'Bautista', '1969-01-18');
insert into actor values ('016','Vin', 'Diesel', '1967-07-18');
insert into actor values ('017','Bradley', 'Cooper', '1975-01-05');
insert into actor values ('018','Michael', 'Rooker', '1955-04-06');
insert into actor values ('019','Elisabeth', 'Olsen', '1989-02-16');
insert into actor values ('020','Paul', 'Bettany', '1971-05-27');
insert into actor values ('021','Anthony', 'Mackie', '1978-09-23');
insert into actor values ('022','Bennedict', 'Cumberbatch', '1976-07-19');
insert into actor values ('023','Rachel', 'McAdams', '1978-11-17');
insert into actor values ('024','Tom', 'Holland', '1996-06-01');
insert into actor values ('025','Zendaya', 'Coleman', '1996-09-01');
insert into actor values ('026','Chadwick', 'Boseman', '1976-11-29');
insert into actor values ('027','Josh', 'Brolin', '1968-02-12');
insert into actor values ('028','Brie', 'Larson', '1989-10-01');

insert into pelicula_actor values ('001', 'MA001', 'Principal');
insert into pelicula_actor values ('002', 'MA001', 'Secundario');
insert into pelicula_actor values ('003', 'MA002', 'Principal');
insert into pelicula_actor values ('004', 'MA002', 'Principal');
insert into pelicula_actor values ('005', 'MA002', 'Principal');
insert into pelicula_actor values ('006', 'MA003', 'Principal');
insert into pelicula_actor values ('007', 'MA003', 'Secundario');
insert into pelicula_actor values ('001', 'MA004', 'Principal');
insert into pelicula_actor values ('002', 'MA004', 'Secundario');
insert into pelicula_actor values ('003', 'MA004', 'Principal');
insert into pelicula_actor values ('004', 'MA004', 'Principal');
insert into pelicula_actor values ('006', 'MA004', 'Principal');
insert into pelicula_actor values ('008', 'MA004', 'Principal');
insert into pelicula_actor values ('009', 'MA004', 'Principal');
insert into pelicula_actor values ('010', 'MA004', 'Principal');
insert into pelicula_actor values ('011', 'MA004', 'Secundario');
insert into pelicula_actor values ('012', 'MA004', 'Principal');
insert into pelicula_actor values ('013', 'MA005', 'Principal');
insert into pelicula_actor values ('014', 'MA005', 'Principal');
insert into pelicula_actor values ('015', 'MA005', 'Principal');
insert into pelicula_actor values ('016', 'MA005', 'Principal');
insert into pelicula_actor values ('017', 'MA005', 'Principal');
insert into pelicula_actor values ('018', 'MA005', 'Secundario');
insert into pelicula_actor values ('001', 'MA006', 'Principal');
insert into pelicula_actor values ('003', 'MA006', 'Principal');
insert into pelicula_actor values ('006', 'MA006', 'Principal');
insert into pelicula_actor values ('008', 'MA006', 'Principal');
insert into pelicula_actor values ('009', 'MA006', 'Principal');
insert into pelicula_actor values ('010', 'MA006', 'Principal');
insert into pelicula_actor values ('019', 'MA006', 'Principal');
insert into pelicula_actor values ('020', 'MA006', 'Principal');
insert into pelicula_actor values ('022', 'MA007', 'Principal');
insert into pelicula_actor values ('023', 'MA007', 'Secundario');
insert into pelicula_actor values ('024', 'MA008', 'Principal');
insert into pelicula_actor values ('025', 'MA008', 'Principal');
insert into pelicula_actor values ('001', 'MA008', 'Secundario');
insert into pelicula_actor values ('002', 'MA008', 'Secundario');
insert into pelicula_actor values ('026', 'MA009', 'Principal');
insert into pelicula_actor values ('027', 'MA010', 'Principal');
insert into pelicula_actor values ('001', 'MA010', 'Principal');
insert into pelicula_actor values ('003', 'MA010', 'Principal');
insert into pelicula_actor values ('004', 'MA010', 'Secundario');
insert into pelicula_actor values ('006', 'MA010', 'Principal');
insert into pelicula_actor values ('007', 'MA010', 'Secundario');
insert into pelicula_actor values ('009', 'MA010', 'Principal');
insert into pelicula_actor values ('010', 'MA010', 'Principal');
insert into pelicula_actor values ('012', 'MA010', 'Secundario');
insert into pelicula_actor values ('013', 'MA010', 'Principal');
insert into pelicula_actor values ('014', 'MA010', 'Principal');
insert into pelicula_actor values ('015', 'MA010', 'Principal');
insert into pelicula_actor values ('016', 'MA010', 'Principal');
insert into pelicula_actor values ('017', 'MA010', 'Principal');
insert into pelicula_actor values ('019', 'MA010', 'Principal');
insert into pelicula_actor values ('020', 'MA010', 'Principal');
insert into pelicula_actor values ('021', 'MA010', 'Secundario');
insert into pelicula_actor values ('022', 'MA010', 'Principal');
insert into pelicula_actor values ('024', 'MA010', 'Principal');
insert into pelicula_actor values ('026', 'MA010', 'Principal');
insert into pelicula_actor values ('001', 'MA011', 'Principal');
insert into pelicula_actor values ('003', 'MA011', 'Principal');
insert into pelicula_actor values ('006', 'MA011', 'Principal');
insert into pelicula_actor values ('008', 'MA011', 'Principal');
insert into pelicula_actor values ('009', 'MA011', 'Principal');
insert into pelicula_actor values ('010', 'MA011', 'Principal');
insert into pelicula_actor values ('027', 'MA011', 'Principal');
insert into pelicula_actor values ('007', 'MA011', 'Secundario');
insert into pelicula_actor values ('013', 'MA011', 'Secundario');
insert into pelicula_actor values ('014', 'MA011', 'Secundario');
insert into pelicula_actor values ('015', 'MA011', 'Secundario');
insert into pelicula_actor values ('016', 'MA011', 'Secundario');
insert into pelicula_actor values ('017', 'MA011', 'Secundario');
insert into pelicula_actor values ('019', 'MA011', 'Secundario');
insert into pelicula_actor values ('021', 'MA011', 'Secundario');
insert into pelicula_actor values ('022', 'MA011', 'Secundario');
insert into pelicula_actor values ('024', 'MA011', 'Secundario');
insert into pelicula_actor values ('026', 'MA011', 'Secundario');
insert into pelicula_actor values ('028', 'MA011', 'Secundario');
```

Se puede comprobar las tablas creadas con **show tables;**.

![tablas-mysql](/img/capturas-arantxa/23.png)

Y los datos añadidos a cada tabla con **select**.

![datos-tabla1](/img/capturas-arantxa/24.png)

![datos-tabla2](/img/capturas-arantxa/25.png)

![datos-tabla3](/img/capturas-arantxa/26.png)


### 1.5 Acceso remoto 

El puerto 3306 es en el que trabaja MySQL por defecto. Para especificar al firewall qué ip puede acceder al servidor de base de datos hacemos lo siguiente:

`sudo ufw allow from ip_cliente to any port 3306`

Pero en mi caso, teniendo en cuenta que estamos en un entorno de prueba, permitiré que se pueda acceder desde cualquier ip.

`sudo ufw allow 3306`

Para verificar la conexión remota accedemos desde otra máquina al servidor de la siguiente forma. Accederemos con el usuario admin, y en mi caso he utilizado una máquina virtual en la que ya tenía instalado mariadb.

`mysql -u admin -h ip_servidor -p`

![acceso-remoto](/img/capturas-arantxa/27.png)




## 2. PostgreSQL

### 2.1 Instalación

Instalamos PostgreSQL en Debian 11 directamente con *apt*.

`sudo apt install postgresql`

Para entrar con el usuario *postgres*.

`sudo su postgres`

`psql`

También se puede usar:

`sudo -u postgres psql`


### 2.2 Creación usuario

Se puede hacer de dos formas:

#### Opción 1:

Hacemos **'sudo su postgres'** y seguidamente creamos el nuevo usuario con **createuser**.

`createuser admin`

Entramos en **psql** y se le asigna una contraseña cifrada.

`alter user admin with encrypted password 'admin';`

#### Opción 2: 

Entramos directamente a psql y usamos el siguiente comando:

`create user admin with encrypted password 'admin';`

Para borrar un usuario:

`drop user admin;`

En psql con **\du** podemos ver los usuarios creados.

![users-postgres](/img/capturas-arantxa/28.png)


### 2.3 Creación base de datos

Para crear la base de datos también se puede hacer de dos formas:

#### Opción 1:

Desde el usuario postgres:

`sudo su postgres`

`createdb maravilla`

#### Opción 2:

Directamente desde psql:

`create database maravilla;`

Podemos ver las bases de datos creadas con:

`\l`

Le damos permisos al usuario **admin** sobre la base de datos creada.

`grant all privileges on database maravilla to admin;`

![bd-postgres](/img/capturas-arantxa/29.png)


### 2.4 Creación tablas e inserción de datos

Nos conectamos a la base de datos *maravilla*:

`\c maravilla`

Creo las tablas.

```
create table actor
(
codigo varchar (5),
nombre varchar (15),
apellido varchar (15),
fechanac date,
constraint pk_actor primary key (codigo)
);

create table pelicula
(
codigo varchar (5),
titulo varchar (30),
fecha_estreno date,
puntuacion decimal (3,1),
lengua_original varchar (3),
constraint pk_pelicula primary key (codigo),
constraint puntuacion_ck check (puntuacion between 0 and 10)
);

create table pelicula_actor
(
codigo_actor varchar (5),
codigo_pelicula varchar (5),
tipo_personaje varchar (15),
constraint pk_pelicula_actor primary key (codigo_actor,codigo_pelicula),
constraint fk_codigo_actor foreign key (codigo_actor) references actor (codigo),
constraint fk_codigo_pelicula foreign key (codigo_pelicula) references pelicula (codigo),
constraint tipo_ck check (tipo_personaje in ('Principal','Secundario'))
);
```

Inserto los datos.

```
insert into pelicula values ('MA001', 'Iron Man', '2008-05-02', 7.9, 'EN');
insert into pelicula values ('MA002', 'Thor', '2011-05-06', 7.0, 'EN');
insert into pelicula values ('MA003', 'Captain America:first avenger', '2011-07-22', 6.9, 'EN');
insert into pelicula values ('MA004', 'The Avengers', '2012-05-04', 8, 'EN');
insert into pelicula values ('MA005', 'Guardians of the Galaxy', '2014-08-01', 8, 'EN');
insert into pelicula values ('MA006', 'Avengers: Age of Ultron', '2015-05-01', 7.3, 'EN');
insert into pelicula values ('MA007', 'Doctor Strange', '2016-11-04', 7.5, 'EN');
insert into pelicula values ('MA008', 'Spider-Man: Homecoming', '2017-07-07', 7.4, 'EN');
insert into pelicula values ('MA009', 'Black Panther', '2018-02-16', 7.3, 'EN');
insert into pelicula values ('MA010', 'Avengers: Infinity War', '2018-04-27', 8.4, 'EN');
insert into pelicula values ('MA011', 'Avengers: Endgame', '2019-04-26', 8.4, 'EN');

insert into actor values ('001','Robert', 'Downey', '1965-04-04');
insert into actor values ('002','Jon', 'Favreau', '1966-10-19');
insert into actor values ('003','Chris', 'Hemsworth', '1983-08-11');
insert into actor values ('004','Tom', 'Hiddleston', '1981-02-09');
insert into actor values ('005','Natalie', 'Portman', '1981-06-09');
insert into actor values ('006','Chris', 'Evans', '1981-06-13');
insert into actor values ('007','Sebastian', 'Stan', '1982-08-13');
insert into actor values ('008','Jeremy', 'Renner', '1971-01-07');
insert into actor values ('009','Scarlett', 'Johansson', '1984-11-22');
insert into actor values ('010','Mark', 'Ruffalo', '1967-11-22');
insert into actor values ('011','Clark', 'Gregg', '1962-04-02');
insert into actor values ('012','Samuel L.', 'Jackson', '1948-12-21');
insert into actor values ('013','Chris', 'Pratt', '1979-06-21');
insert into actor values ('014','Zoe', 'Saldaña', '1978-06-19');
insert into actor values ('015','Dave', 'Bautista', '1969-01-18');
insert into actor values ('016','Vin', 'Diesel', '1967-07-18');
insert into actor values ('017','Bradley', 'Cooper', '1975-01-05');
insert into actor values ('018','Michael', 'Rooker', '1955-04-06');
insert into actor values ('019','Elisabeth', 'Olsen', '1989-02-16');
insert into actor values ('020','Paul', 'Bettany', '1971-05-27');
insert into actor values ('021','Anthony', 'Mackie', '1978-09-23');
insert into actor values ('022','Bennedict', 'Cumberbatch', '1976-07-19');
insert into actor values ('023','Rachel', 'McAdams', '1978-11-17');
insert into actor values ('024','Tom', 'Holland', '1996-06-01');
insert into actor values ('025','Zendaya', 'Coleman', '1996-09-01');
insert into actor values ('026','Chadwick', 'Boseman', '1976-11-29');
insert into actor values ('027','Josh', 'Brolin', '1968-02-12');
insert into actor values ('028','Brie', 'Larson', '1989-10-01');

insert into pelicula_actor values ('001', 'MA001', 'Principal');
insert into pelicula_actor values ('002', 'MA001', 'Secundario');
insert into pelicula_actor values ('003', 'MA002', 'Principal');
insert into pelicula_actor values ('004', 'MA002', 'Principal');
insert into pelicula_actor values ('005', 'MA002', 'Principal');
insert into pelicula_actor values ('006', 'MA003', 'Principal');
insert into pelicula_actor values ('007', 'MA003', 'Secundario');
insert into pelicula_actor values ('001', 'MA004', 'Principal');
insert into pelicula_actor values ('002', 'MA004', 'Secundario');
insert into pelicula_actor values ('003', 'MA004', 'Principal');
insert into pelicula_actor values ('004', 'MA004', 'Principal');
insert into pelicula_actor values ('006', 'MA004', 'Principal');
insert into pelicula_actor values ('008', 'MA004', 'Principal');
insert into pelicula_actor values ('009', 'MA004', 'Principal');
insert into pelicula_actor values ('010', 'MA004', 'Principal');
insert into pelicula_actor values ('011', 'MA004', 'Secundario');
insert into pelicula_actor values ('012', 'MA004', 'Principal');
insert into pelicula_actor values ('013', 'MA005', 'Principal');
insert into pelicula_actor values ('014', 'MA005', 'Principal');
insert into pelicula_actor values ('015', 'MA005', 'Principal');
insert into pelicula_actor values ('016', 'MA005', 'Principal');
insert into pelicula_actor values ('017', 'MA005', 'Principal');
insert into pelicula_actor values ('018', 'MA005', 'Secundario');
insert into pelicula_actor values ('001', 'MA006', 'Principal');
insert into pelicula_actor values ('003', 'MA006', 'Principal');
insert into pelicula_actor values ('006', 'MA006', 'Principal');
insert into pelicula_actor values ('008', 'MA006', 'Principal');
insert into pelicula_actor values ('009', 'MA006', 'Principal');
insert into pelicula_actor values ('010', 'MA006', 'Principal');
insert into pelicula_actor values ('019', 'MA006', 'Principal');
insert into pelicula_actor values ('020', 'MA006', 'Principal');
insert into pelicula_actor values ('022', 'MA007', 'Principal');
insert into pelicula_actor values ('023', 'MA007', 'Secundario');
insert into pelicula_actor values ('024', 'MA008', 'Principal');
insert into pelicula_actor values ('025', 'MA008', 'Principal');
insert into pelicula_actor values ('001', 'MA008', 'Secundario');
insert into pelicula_actor values ('002', 'MA008', 'Secundario');
insert into pelicula_actor values ('026', 'MA009', 'Principal');
insert into pelicula_actor values ('027', 'MA010', 'Principal');
insert into pelicula_actor values ('001', 'MA010', 'Principal');
insert into pelicula_actor values ('003', 'MA010', 'Principal');
insert into pelicula_actor values ('004', 'MA010', 'Secundario');
insert into pelicula_actor values ('006', 'MA010', 'Principal');
insert into pelicula_actor values ('007', 'MA010', 'Secundario');
insert into pelicula_actor values ('009', 'MA010', 'Principal');
insert into pelicula_actor values ('010', 'MA010', 'Principal');
insert into pelicula_actor values ('012', 'MA010', 'Secundario');
insert into pelicula_actor values ('013', 'MA010', 'Principal');
insert into pelicula_actor values ('014', 'MA010', 'Principal');
insert into pelicula_actor values ('015', 'MA010', 'Principal');
insert into pelicula_actor values ('016', 'MA010', 'Principal');
insert into pelicula_actor values ('017', 'MA010', 'Principal');
insert into pelicula_actor values ('019', 'MA010', 'Principal');
insert into pelicula_actor values ('020', 'MA010', 'Principal');
insert into pelicula_actor values ('021', 'MA010', 'Secundario');
insert into pelicula_actor values ('022', 'MA010', 'Principal');
insert into pelicula_actor values ('024', 'MA010', 'Principal');
insert into pelicula_actor values ('026', 'MA010', 'Principal');
insert into pelicula_actor values ('001', 'MA011', 'Principal');
insert into pelicula_actor values ('003', 'MA011', 'Principal');
insert into pelicula_actor values ('006', 'MA011', 'Principal');
insert into pelicula_actor values ('008', 'MA011', 'Principal');
insert into pelicula_actor values ('009', 'MA011', 'Principal');
insert into pelicula_actor values ('010', 'MA011', 'Principal');
insert into pelicula_actor values ('027', 'MA011', 'Principal');
insert into pelicula_actor values ('007', 'MA011', 'Secundario');
insert into pelicula_actor values ('013', 'MA011', 'Secundario');
insert into pelicula_actor values ('014', 'MA011', 'Secundario');
insert into pelicula_actor values ('015', 'MA011', 'Secundario');
insert into pelicula_actor values ('016', 'MA011', 'Secundario');
insert into pelicula_actor values ('017', 'MA011', 'Secundario');
insert into pelicula_actor values ('019', 'MA011', 'Secundario');
insert into pelicula_actor values ('021', 'MA011', 'Secundario');
insert into pelicula_actor values ('022', 'MA011', 'Secundario');
insert into pelicula_actor values ('024', 'MA011', 'Secundario');
insert into pelicula_actor values ('026', 'MA011', 'Secundario');
insert into pelicula_actor values ('028', 'MA011', 'Secundario');
```

![tables-postgres](/img/capturas-arantxa/30.png)

![tables-postgres2](/img/capturas-arantxa/31.png)


### 2.5 Acceso remoto

Modificamos el fichero de configuración de PostgreSQL que se encuentra en la ruta **/etc/postgresql/13/main/postgresql.conf**. Descomentamos la opción **listen_addresses** y donde pone *localhost* ponemos un asterisco.

![remoto-postgres](/img/capturas-arantxa/32.png)

A continuación modificamos el fichero **pg_hba.conf**, en el mismo directorio de la anterior. Buscamos la línea que pone **# IPv4 local connections**, y donde pone *127.0.0.1/32* lo modificamos por *all*.

![remoto-postgres2](/img/capturas-arantxa/33.png)

Guardamos los cambios y reiniciamos el servicio de PostgreSQL.

`sudo systemctl restart postgresql`

En la **maquina cliente** instalamos **postgresql-client**.

`sudo apt install postgresql-client`

Nos conectamos a la base de datos *maravilla* con el cliente *admin* del siguiente modo (indicar la ip del servidor de base de datos):

`psql -h 192.168.122.98 -U admin -d maravilla`

![remoto-postgres3](/img/capturas-arantxa/34.png)




## 3. Oracle

### 3.1 Instalación

Descargamos Oracle 19c desde su página oficial. Si se hac con wget nos dará error a la hora de pasarlo a formato deb con alien. Vamos a su página y lo descargamos.

https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html

Vemos que está en formato rpm, así que utilizaremos la herramienta **alien** para pasarlo a formato deb.

>¡OJO! Si ya tienes Oracle pasado a .deb debes instalar los paquetes **libaio1** y **unixodbc**, que son librerías para Oracle. Si no lo instalas dará error cuando hagas **configure**.

`sudo apt install alien libaio1 unixodbc`

Usamos alien para convertir el paquete a deb. Este proceso puede ser un poco lento.

`sudo alien --scripts -d oracle-database-ee-19c-1.0-1.x86_64.rpm`

Ahora vamos a crear un script en sbin que tendrá el siguiente contenido.

`sudo nano /sbin/chkconfig`

```
#!/bin/bash
# Oracle XE Installer chkconfig for Debian 11
file=/etc/init.d/oracle-xe
if [[ ! `tail -n1 $file | grep INIT` ]]; then
echo >> $file
echo '### BEGIN INIT INFO' >> $file
echo '# Provides: OracleXE' >> $file
echo '# Required-Start: $remote_fs $syslog' >> $file
echo '# Required-Stop: $remote_fs $syslog' >> $file
echo '# Default-Start: 2 3 4 5' >> $file
echo '# Default-Stop: 0 1 6' >> $file
echo '# Short-Description: Oracle 19c Express Edition' >> $file
echo '### END INIT INFO' >> $file
fi
update-rc.d oracle-xe defaults 80 01
```

![script](/img/capturas-arantxa/6.png)

Damos los permisos al fichero creado.

`sudo chmod 777 /sbin/chkconfig`

Para que Oracle funcione se debe configurar el kernel por lo que añadiremos lo siguiente:

`sudo nano /etc/sysctl.d/60-oracle.conf`

```
# Oracle 19c XE kernel parameters  
fs.file-max=6815744  
net.ipv4.ip_local_port_range=9000 65000  
kernel.sem=250 32000 100 128
kernel.shmmax=536870912
```

![conf](/img/capturas-arantxa/7.png)

Cargamos los parámetros sin tener que reiniciar el sistema.

`sudo systemctl start procps.service`


Y ya podremos instalar Oracle 19c.

`sudo dpkg -i oracle-database-ee-19c_1.0-2_amd64.deb`

![instalacion-oracle](/img/capturas-arantxa/8.png)




### 3.2 Configuración

Creación de la base de datos y configuración de la contraseña del administrador.

`sudo /etc/init.d/oracledb_ORCLCDB-19c configure`


##### Posibles errores al hacer configure

_**ERROR 1. Fallo en la comprobación de la memoria**_

A mi me aparece el siguiente error.

![error-oracle](/img/capturas-arantxa/11.png)

Para solucionarlo configuramos el fichero **/etc/init.d/oracledb_ORCLCDB-19c**. Donde pone **configure_perform** añadimos lo siguiente a la línea **$SU**, como se ve en la captura.

`-J-Doracle.assistants.dbca.validate.ConfigurationParams=false`

![conf-oracle](/img/capturas-arantxa/9.png)

![conf-oracle](/img/capturas-arantxa/10.png)

_**ERROR 2. Fallo en la configuración de la red**_

También me apareció el siguiente problema.

![error-oracle2](/img/capturas-arantxa/12.png)

Para solucionarlo hay que instalar las net-tools.

`sudo apt install net-tools`

Y para terminar añadir a **/etc/hosts** nuestra ip, en mi caso **192.168.122.98** para *debian-oracle*. Quedaría de la siguiente forma.

`sudo nano /etc/hosts`

```
127.0.0.1 localhost
192.168.122.98 debian-oracle
```

![conf-ip](/img/capturas-arantxa/13.png)

Ya deberíamos poder realizar la configuración.

![configuracion-proceso](/img/capturas-arantxa/14.png)


### 3.3 Terminar configuración

Añadimos las variables de entorno bash.

`sudo nano ~/.bashrc`

Al final del fichero, en la última línea añadir lo siguiente:

```
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export ORACLE_SID=XE
export ORACLE_BASE=/opt/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
```

![variables-entorno](/img/capturas-arantxa/15.png)

Iniciamos las variables de entorno.

`. ~/.profile`

Arrancamos el servicio Oracle.

`sudo systemctl start oracledb_ORCLCDB-19c`

Si no conocemos el nombre exacto del servicio buscarlo con:

`sudo systemctl list-unit-files --type service | grep oracle`

![servicio-oracle-activo](/img/capturas-arantxa/16.png)

Podremos entrar a Oracle usando el siguiente comando:

`sqlplus / as sysdba`

Pero nuestro usuario local deberá estar en el grupo **dba**.

`sudo nano /etc/group`

![grupo-dba](/img/capturas-arantxa/17.png)

![acceso-usuario](/img/capturas-arantxa/18.png)


### 3.4 Iniciar la base de datos

Si al inicializar la base de datos aparece el siguiente error es porque no le hemos dado el nombre correcto a la base de datos.

![error-bd-init.ora](/img/capturas-arantxa/20.png)

Cambiamos el nombre ORACLE_SID, al nombre de nuestra base de datos, en este caso ORCLCDB.

`export ORACLE_SID=ORCLCDB`

Volvemos a iniciar las variables de entorno y reiniciamos el proceso.

`. ~/.profile`

`sudo systemctl start oracledb_ORCLCDB-19c`

Además si nos aparece error de que no encuentra los directorios deberemos crear las siguientes carpetas en el directorio base de Oracle, en mi caso **/opt/oracle/product/19c/dbhome_1**.

`sudo mkdir fast_recovery_area`

`sudo mkdir adump`

`sudo chown -R oracle:oinstall fast_recovery_area`

`sudo chown -R oracle:oinstall adump`

Entramos de nuevo como *"sqlplus / as sysdba"* e iniciamos la base de datos con **startup**.

![bd-montada](/img/capturas-arantxa/21.png)



### 3.5 




## 4. MongoDB

### 4.1 Instalación

### 4.2 Creación usuario



### 4.3 Creación base de datos



### 4.4 Creación tablas e inserción de datos



### 4.5 Acceso remoto