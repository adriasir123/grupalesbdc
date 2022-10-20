# Alumno 1

## Escenario

```shell

```

## Parte 1

> Instalación de un servidor Oracle 19c sobre Debian y configuración para permitir el acceso remoto desde la red local















## 2. PostgreSQL

### 2.1

> Instalar el servidor

```shell
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Compruebo que funciona:

```shell
vagrant@servidorpostgresql:~$ sudo systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2022-10-18 10:12:46 UTC; 2min 50s ago
   Main PID: 2772 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 527)
     Memory: 0B
        CPU: 0
     CGroup: /system.slice/postgresql.service

Oct 18 10:12:46 servidorpostgresql systemd[1]: Starting PostgreSQL RDBMS...
Oct 18 10:12:46 servidorpostgresql systemd[1]: Finished PostgreSQL RDBMS.
```

### 2.2

> Acceder con el usuario administrador localmente

Añado la contraseña `1234` a `postgres` para habilitarlo:

```shell
sudo passwd postgres
```

Pruebo el acceso con `postgres`:

```shell
vagrant@servidorpostgresql:~$ su - postgres
Password:
postgres@servidorpostgresql:~$ psql
psql (13.8 (Debian 13.8-0+deb11u1))
Type "help" for help.

postgres=#
```

### 2.3

> Crear la bd

```sql
CREATE DATABASE bibliofilos;
```

Compruebo que se ha creado:

```shell
postgres=# \l
                               List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |   Access privileges
-------------+----------+----------+---------+---------+-----------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
             |          |          |         |         | postgres=CTc/postgres
(4 rows)
```

Me conecto:

```shell
postgres=# \connect bibliofilos
You are now connected to database "bibliofilos" as user "postgres".
bibliofilos=#
```

### 2.4

> Crear tablas

```shell
CREATE TABLE bibliotecas (
  id serial PRIMARY KEY,
  ciudad VARCHAR ( 50 ) NOT NULL,
  calle VARCHAR ( 50 ) NOT NULL
);

CREATE TABLE libros (
  id serial PRIMARY KEY,
  id_biblioteca serial NOT NULL,
  n_paginas int NOT NULL,
  autor VARCHAR ( 50 ) NOT NULL,
  FOREIGN KEY (id_biblioteca)
    REFERENCES Bibliotecas (id)
);

CREATE TABLE trabajadores (
  id serial PRIMARY KEY,
  id_biblioteca serial NOT NULL,
  nombre VARCHAR ( 50 ) NOT NULL,
  apellido VARCHAR ( 50 ) NOT NULL,
  FOREIGN KEY (id_biblioteca)
    REFERENCES Bibliotecas (id)
);
```

Compruebo que se han creado:

```shell
bibliofilos=# \dt
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | bibliotecas  | table | postgres
 public | libros       | table | postgres
 public | trabajadores | table | postgres
(3 rows)
```

### 2.5

> Introducir registros

```sql
INSERT INTO bibliotecas (ciudad, calle) VALUES('Utrera', 'Alvarez Quintero');
INSERT INTO bibliotecas (ciudad, calle) VALUES('Dos Hermanas', 'Plaza Huerta Palacios');

INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES(1, 320, 'Dale Carnegie');
INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES(2, 714, 'Anne Frank');

INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES(1, 'Pepe', 'Pepito');
INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES(2, 'Jose', 'Joselito');
```

Compruebo que se han añadido:

```shell
bibliofilos=# select * from bibliotecas;
 id |    ciudad    |         calle
----+--------------+-----------------------
  1 | Utrera       | Alvarez Quintero
  2 | Dos Hermanas | Plaza Huerta Palacios
(2 rows)

bibliofilos=# select * from libros;
 id | id_biblioteca | n_paginas |     autor
----+---------------+-----------+---------------
  1 |             1 |       320 | Dale Carnegie
  2 |             2 |       714 | Anne Frank
(2 rows)

bibliofilos=# select * from trabajadores;
 id | id_biblioteca | nombre | apellido
----+---------------+--------+----------
  1 |             1 | Pepe   | Pepito
  2 |             2 | Jose   | Joselito
(2 rows)
```

### 2.6

> Crear usuario con todos los privilegios sobre la base de datos anterior

En debian:

```shell
sudo adduser bibliofilos_admin
```

En PostgreSQL:

```sql
create user bibliofilos_admin with encrypted password '1234';
```

Compruebo que se ha creado:

```shell
postgres=# \du
                                       List of roles
     Role name     |                         Attributes                         | Member of
-------------------+------------------------------------------------------------+-----------
 bibliofilos_admin |                                                            | {}
 postgres          | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

Le doy privilegios sobre la base de datos:

```sql
GRANT all privileges ON DATABASE bibliofilos TO bibliofilos_admin;
```

Al listar las bases de datos podemos ver los privilegios de acceso:

```shell
postgres=# \l
                                   List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |       Access privileges
-------------+----------+----------+---------+---------+--------------------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | bibliofilos_admin=CTc/postgres
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
(4 rows)
```

Le doy privilegios sobre las tablas:

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO bibliofilos_admin;
```

Muestro que se han aplicado los privilegios:

```sql
bibliofilos=# SELECT * from information_schema.table_privileges WHERE grantee = 'bibliofilos_admin';
 grantor  |      grantee      | table_catalog | table_schema |  table_name  | privilege_type | is_grantable | with_hierarchy
----------+-------------------+---------------+--------------+--------------+----------------+--------------+----------------
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | INSERT         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | SELECT         | NO           | YES
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | UPDATE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | DELETE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | TRUNCATE       | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | REFERENCES     | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | TRIGGER        | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | INSERT         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | SELECT         | NO           | YES
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | UPDATE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | DELETE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | TRUNCATE       | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | REFERENCES     | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | TRIGGER        | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | INSERT         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | SELECT         | NO           | YES
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | UPDATE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | DELETE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | TRUNCATE       | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | REFERENCES     | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | TRIGGER        | NO           | NO
(21 rows)
```

### 2.7

> Permitir el acceso remoto

Modifico la siguiente línea en `/etc/postgresql/13/main/postgresql.conf`:

```shell
listen_addresses = '*'
```

Añado las siguientes líneas a `/etc/postgresql/13/main/pg_hba.conf`:

```shell
# Remote connections
host    all             all             0.0.0.0/0               md5
```

Reinicio:

```shell
sudo systemctl restart postgresql 
```

### 2.8

> Instalar el cliente de PostgreSQL en `clientepostgresql`

```shell
sudo apt update
sudo apt install postgresql-client
```

### 2.9

> Probar el acceso remoto

```shell
vagrant@clientepostgresql:~$ psql -U bibliofilos_admin -h 10.0.1.2 -p 5432 bibliofilos
Password for user bibliofilos_admin:
psql (13.8 (Debian 13.8-0+deb11u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

bibliofilos=> \l
                                   List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |       Access privileges
-------------+----------+----------+---------+---------+--------------------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | bibliofilos_admin=CTc/postgres
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
(4 rows)

bibliofilos=> \dt
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | bibliotecas  | table | postgres
 public | libros       | table | postgres
 public | trabajadores | table | postgres
(3 rows)

bibliofilos=> select * from bibliotecas;
 id |    ciudad    |         calle
----+--------------+-----------------------
  1 | Utrera       | Alvarez Quintero
  2 | Dos Hermanas | Plaza Huerta Palacios
(2 rows)
```

## 3. MariaDB

### 3.1

> Instalar el servidor

```shell
sudo apt update
sudo apt install mariadb-server
```

Compruebo que funciona:

```shell
vagrant@servidormariadb:~$ sudo systemctl status mariadb
● mariadb.service - MariaDB 10.5.15 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-10-19 11:01:36 UTC; 11min ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 2111 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysqld (code=exited, status=0/SUCCESS)
    Process: 2112 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 2114 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`cd /usr/bin/..; /usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-envi>
    Process: 2174 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 2176 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
   Main PID: 2161 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 8 (limit: 527)
     Memory: 61.7M
        CPU: 425ms
     CGroup: /system.slice/mariadb.service
             └─2161 /usr/sbin/mariadbd

Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] Server socket created on IP: '127.0.0.1'.
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] InnoDB: Buffer pool(s) load completed at 221019 11:01:36
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] Reading of all Master_info entries succeeded
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] Added new Master_info '' to hash table
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] /usr/sbin/mariadbd: ready for connections.
Oct 19 11:01:36 servidormariadb mariadbd[2161]: Version: '10.5.15-MariaDB-0+deb11u1'  socket: '/run/mysqld/mysqld.sock'  port: 3306  Debian 11
Oct 19 11:01:36 servidormariadb systemd[1]: Started MariaDB 10.5.15 database server.
Oct 19 11:01:36 servidormariadb /etc/mysql/debian-start[2178]: Upgrading MySQL tables if necessary.
Oct 19 11:01:36 servidormariadb /etc/mysql/debian-start[2189]: Checking for insecure root accounts.
Oct 19 11:01:36 servidormariadb /etc/mysql/debian-start[2193]: Triggering myisam-recover for all MyISAM tables and aria-recover for all Aria tables
```

### 3.2

> Acceder con el usuario administrador localmente

```shell
vagrant@servidormariadb:~$ sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 34
Server version: 10.5.15-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

### 3.3

> Crear la bd

```sql
create database bibliofilos;
```

Compruebo que se ha creado:

```shell
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| bibliofilos        |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec)
```

Me conecto:

```shell
MariaDB [(none)]> use bibliofilos;
Database changed
MariaDB [bibliofilos]>
```

### 3.4

> Crear tablas

```sql
create table bibliotecas(
  id int auto_increment,
  ciudad varchar(50) not null,
  calle varchar(50) not null,
  primary key(id)
);

create table libros(
  id int auto_increment,
  id_biblioteca int not null,
  n_paginas int not null,
  autor varchar(50) not null,
  primary key(id),
  foreign key(id_biblioteca)
    references bibliotecas(id)
);

create table trabajadores(
  id int auto_increment,
  id_biblioteca int not null,
  nombre varchar(50) not null,
  apellido varchar(50) not null,
  primary key(id),
  foreign key(id_biblioteca)
    references bibliotecas(id)
);
```

Compruebo que se han creado:

```shell
MariaDB [bibliofilos]> show tables;
+-----------------------+
| Tables_in_bibliofilos |
+-----------------------+
| bibliotecas           |
| libros                |
| trabajadores          |
+-----------------------+
3 rows in set (0.001 sec)
```

### 3.5

> Introducir registros

```sql
INSERT INTO bibliotecas (ciudad, calle) VALUES ('Utrera', 'Alvarez Quintero');
INSERT INTO bibliotecas (ciudad, calle) VALUES ('Dos Hermanas', 'Plaza Huerta Palacios');

INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES (1, 320, 'Dale Carnegie');
INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES (2, 714, 'Anne Frank');

INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES (1, 'Pepe', 'Pepito');
INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES (2, 'Jose', 'Joselito');
```

Compruebo que se han añadido:

```shell
MariaDB [bibliofilos]> select * from bibliotecas;
+----+--------------+-----------------------+
| id | ciudad       | calle                 |
+----+--------------+-----------------------+
|  1 | Utrera       | Alvarez Quintero      |
|  2 | Dos Hermanas | Plaza Huerta Palacios |
+----+--------------+-----------------------+
2 rows in set (0.001 sec)

MariaDB [bibliofilos]> select * from libros;
+----+---------------+-----------+---------------+
| id | id_biblioteca | n_paginas | autor         |
+----+---------------+-----------+---------------+
|  1 |             1 |       320 | Dale Carnegie |
|  2 |             2 |       714 | Anne Frank    |
+----+---------------+-----------+---------------+
2 rows in set (0.001 sec)

MariaDB [bibliofilos]> select * from trabajadores;
+----+---------------+--------+----------+
| id | id_biblioteca | nombre | apellido |
+----+---------------+--------+----------+
|  1 |             1 | Pepe   | Pepito   |
|  2 |             2 | Jose   | Joselito |
+----+---------------+--------+----------+
2 rows in set (0.001 sec)
```

### 3.6

> Crear usuario con todos los privilegios sobre la base de datos anterior

```sql
CREATE USER 'bibliofilos_admin'@'localhost' IDENTIFIED BY '1234';
CREATE USER 'bibliofilos_admin'@'%' IDENTIFIED BY '1234';
```

Compruebo que se ha creado:

```shell
MariaDB [(none)]> SELECT user, host FROM mysql.user;
+-------------------+-----------+
| User              | Host      |
+-------------------+-----------+
| bibliofilos_admin | %         |
| bibliofilos_admin | localhost |
| mariadb.sys       | localhost |
| mysql             | localhost |
| root              | localhost |
+-------------------+-----------+
5 rows in set (0.002 sec)
```

Le doy privilegios:

```sql
GRANT ALL ON bibliofilos.* TO 'bibliofilos_admin'@'localhost';
GRANT ALL ON bibliofilos.* TO 'bibliofilos_admin'@'%';
FLUSH PRIVILEGES;
```

Muestro que se han aplicado:

```sql
MariaDB [(none)]> SHOW GRANTS FOR 'bibliofilos_admin';
+------------------------------------------------------------------------------------------------------------------+
| Grants for bibliofilos_admin@%                                                                                   |
+------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `bibliofilos_admin`@`%` IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' |
| GRANT ALL PRIVILEGES ON `bibliofilos`.* TO `bibliofilos_admin`@`%`                                               |
+------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)

MariaDB [(none)]> SHOW GRANTS FOR 'bibliofilos_admin'@'localhost';
+--------------------------------------------------------------------------------------------------------------------------+
| Grants for bibliofilos_admin@localhost                                                                                   |
+--------------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `bibliofilos_admin`@`localhost` IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' |
| GRANT ALL PRIVILEGES ON `bibliofilos`.* TO `bibliofilos_admin`@`localhost`                                               |
+--------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)
```

### 3.7

> Permitir el acceso remoto

Modifico la siguiente línea en `/etc/mysql/mariadb.conf.d/50-server.cnf`:

```shell
bind-address            = 0.0.0.0
```

Reinicio:

```shell
sudo systemctl restart mariadb
```

Pruebo que el cambio se aplicó:

```shell
vagrant@servidormariadb:~$ netstat -ant | grep 3306
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN
```

### 3.8

> Instalar el cliente de MariaDB en `clientemariadb`

```shell
sudo apt update
sudo apt install mariadb-client
```

### 3.9

> Probar el acceso remoto

```shell
vagrant@clientemariadb:~$ mariadb -u bibliofilos_admin -h 10.0.2.2 -p bibliofilos
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 35
Server version: 10.5.15-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [bibliofilos]> show databases;
+--------------------+
| Database           |
+--------------------+
| bibliofilos        |
| information_schema |
+--------------------+
2 rows in set (0.002 sec)

MariaDB [bibliofilos]> show tables;
+-----------------------+
| Tables_in_bibliofilos |
+-----------------------+
| bibliotecas           |
| libros                |
| trabajadores          |
+-----------------------+
3 rows in set (0.001 sec)

MariaDB [bibliofilos]> select * from bibliotecas;
+----+--------------+-----------------------+
| id | ciudad       | calle                 |
+----+--------------+-----------------------+
|  1 | Utrera       | Alvarez Quintero      |
|  2 | Dos Hermanas | Plaza Huerta Palacios |
+----+--------------+-----------------------+
2 rows in set (0.002 sec)
```

## 4. MongoDB

### 4.1

> Instalar el servidor

Añado el repositorio:

```shell
echo "deb http://repo.mongodb.org/apt/debian bullseye/mongodb-org/5.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

Añado la clave GPG:

```shell
curl -sSL https://www.mongodb.org/static/pgp/server-5.0.asc  -o mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --import ./mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --export > ./mongoserver_key.gpg
sudo mv mongoserver_key.gpg /etc/apt/trusted.gpg.d/
```

Actualizo:

```shell
sudo apt update
```

Instalo:

```shell
sudo apt install mongodb-org
```

Inicio el servidor:

```shell
sudo systemctl enable --now mongod
```

Compruebo que funciona:

```shell
vagrant@servidormongodb:~$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-10-20 08:20:59 UTC; 2min 26s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 2695 (mongod)
     Memory: 120.9M
        CPU: 1.866s
     CGroup: /system.slice/mongod.service
             └─2695 /usr/bin/mongod --config /etc/mongod.conf

Oct 20 08:20:59 servidormongodb systemd[1]: Started MongoDB Database Server.
```

### 4.2

> Probar el acceso

```shell
vagrant@servidormongodb:~$ mongosh
Current Mongosh Log ID:63510a480bb54d2af15cd761
Connecting to:	        mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.6.0
Using MongoDB:	        5.0.13
Using Mongosh:	        1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2022-10-20T08:21:00.034+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2022-10-20T08:21:00.617+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
------

------
   Enable MongoDB's free cloud-based monitoring service, which will then receive and display
   metrics about your deployment (disk utilization, CPU, operation statistics, etc).

   The monitoring data will be available on a MongoDB website with a unique URL accessible to you
   and anyone you share the URL with. MongoDB may use this information to make product
   improvements and to suggest MongoDB products and deployment options to you.

   To enable free monitoring, run the following command: db.enableFreeMonitoring()
   To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
------

Warning: Found ~/.mongorc.js, but not ~/.mongoshrc.js. ~/.mongorc.js will not be loaded.
  You may want to copy or rename ~/.mongorc.js to ~/.mongoshrc.js.
test>
```

### 4.3

> Crear la bd

Si no existe la bd con el nombre que le digamos, la creará:

```sql
test> use bibliofilos
switched to db bibliofilos
bibliofilos>
```

Compruebo que se ha creado:

```shell
bibliofilos> show dbs
admin    40.00 KiB
config  108.00 KiB
local    40.00 KiB
```

La bd no aparece listada por ahora porque necesita contener datos para aparecer.

### 4.4

> Crear colecciones

```sql
db.createCollection("bibliotecas")

db.createCollection("libros")

db.createCollection("trabajadores")
```

Compruebo que se han creado:

```shell
bibliofilos> show collections
bibliotecas
libros
trabajadores
```

### 4.5

> Insertar documentos

```sql
db.bibliotecas.insertMany( [
  { ciudad: "Utrera", calle: "Alvarez Quintero" },
  { ciudad: "Dos Hermanas", calle: "Plaza Huerta Palacios" }
] )

db.libros.insertMany( [
  { n_paginas: 320, autor: "Dale Carnegie" },
  { n_paginas: 714, autor: "Anne Frank" }
] )

db.trabajadores.insertMany( [
  { nombre: "Pepe", apellido: "Pepito" },
  { nombre: "Jose", apellido: "Joselito" },
] )
```

Compruebo que se han insertado:

```shell
bibliofilos> db.bibliotecas.find()
[
  {
    _id: ObjectId("635126caf4c5855bf45547d3"),
    ciudad: 'Utrera',
    calle: 'Alvarez Quintero'
  },
  {
    _id: ObjectId("635126caf4c5855bf45547d4"),
    ciudad: 'Dos Hermanas',
    calle: 'Plaza Huerta Palacios'
  }
]
bibliofilos> db.libros.find()
[
  {
    _id: ObjectId("63512712f4c5855bf45547d5"),
    n_paginas: 320,
    autor: 'Dale Carnegie'
  },
  {
    _id: ObjectId("63512712f4c5855bf45547d6"),
    n_paginas: 714,
    autor: 'Anne Frank'
  }
]
bibliofilos> db.trabajadores.find()
[
  {
    _id: ObjectId("6351271cf4c5855bf45547d7"),
    nombre: 'Pepe',
    apellido: 'Pepito'
  },
  {
    _id: ObjectId("6351271cf4c5855bf45547d8"),
    nombre: 'Jose',
    apellido: 'Joselito'
  }
]
```

### 4.6

> Crear usuario con todos los privilegios sobre la base de datos anterior

```sql
use bibliofilos
db.createUser(
  {
    user: "bibliofilos_admin",
    pwd: "1234",
    roles: [ "dbOwner" ]
  }
)
```

Compruebo que se ha creado:

```shell
bibliofilos> db.getUsers()
{
  users: [
    {
      _id: 'bibliofilos.bibliofilos_admin',
      userId: new UUID("e420bea0-b85f-4914-a395-2f2b20b3aa48"),
      user: 'bibliofilos_admin',
      db: 'bibliofilos',
      roles: [ { role: 'dbOwner', db: 'bibliofilos' } ],
      mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
    }
  ],
  ok: 1
}
```

### 4.7

> Permitir el acceso remoto

Modifico la siguiente línea en `/etc/mongod.conf`:

```shell
bindIp: 0.0.0.0
```

Añado las siguientes líneas en `/etc/mongod.conf`:

```shell
security:
  authorization: enabled
```

Reinicio:

```shell
sudo systemctl restart mongod
```

Pruebo que el cambio se aplicó:

```shell
vagrant@servidormongodb:~$ netstat -ant | grep 27017
tcp        0      0 0.0.0.0:27017           0.0.0.0:*               LISTEN
```

### 4.8

> Instalar el cliente de MongoDB en `clientemongodb`

Añado el repositorio:

```shell
echo "deb http://repo.mongodb.org/apt/debian bullseye/mongodb-org/5.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

Añado la clave GPG:

```shell
curl -sSL https://www.mongodb.org/static/pgp/server-5.0.asc  -o mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --import ./mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --export > ./mongoserver_key.gpg
sudo mv mongoserver_key.gpg /etc/apt/trusted.gpg.d/
```

Actualizo:

```shell
sudo apt update
```

Instalo:

```shell
sudo apt install mongodb-org-shell mongodb-mongosh
```

### 4.9

> Probar el acceso remoto

```shell
vagrant@clientemongodb:~$ mongosh -u bibliofilos_admin -p 1234 10.0.3.2/bibliofilos
Current Mongosh Log ID:6351bcb93f3b38d91c3c973b
Connecting to:	        mongodb://<credentials>@10.0.3.2:27017/bibliofilos?directConnection=true&appName=mongosh+1.6.0
Using MongoDB:	        5.0.13
Using Mongosh:	        1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

Warning: Found ~/.mongorc.js, but not ~/.mongoshrc.js. ~/.mongorc.js will not be loaded.
  You may want to copy or rename ~/.mongorc.js to ~/.mongoshrc.js.
bibliofilos> show dbs
bibliofilos  120.00 KiB
bibliofilos> show collections
bibliotecas
libros
trabajadores
bibliofilos> db.bibliotecas.find()
[
  {
    _id: ObjectId("635126caf4c5855bf45547d3"),
    ciudad: 'Utrera',
    calle: 'Alvarez Quintero'
  },
  {
    _id: ObjectId("635126caf4c5855bf45547d4"),
    ciudad: 'Dos Hermanas',
    calle: 'Plaza Huerta Palacios'
  }
]
```

## Parte 5

> Prueba desde un cliente remoto del intérprete de comandos de Postgres.



















## Parte 6

> Realización de una aplicación web en cualquier lenguaje que conecte con el servidor MongoDB desde un cliente remoto tras autenticarse y muestre alguna información almacenada en el mismo.


Se aportará el código de las aplicaciones realizadas y prueba de funcionamiento de las mismas.






