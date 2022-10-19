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
INSERT INTO bibliotecas (ciudad, calle) VALUES('Utrera','Alvarez Quintero');
INSERT INTO bibliotecas (ciudad, calle) VALUES('Dos Hermanas','Plaza Huerta Palacios');

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



































## Parte 3

> Instalación de un servidor MariaDB sobre Debian y configuración para permitir el acceso remoto desde la red local




## Parte 4

> Instalación de un servidor MongoDB sobre Debian y configuración para permitir el acceso remoto desde la red local














## Parte 5

> Prueba desde un cliente remoto del intérprete de comandos de Postgres.







## Parte 6

> Realización de una aplicación web en cualquier lenguaje que conecte con el servidor MongoDB desde un cliente remoto tras autenticarse y muestre alguna información almacenada en el mismo.


Se aportará el código de las aplicaciones realizadas y prueba de funcionamiento de las mismas.






