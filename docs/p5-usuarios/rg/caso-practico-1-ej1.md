# Ejercicio 1

## Oracle

Creo el usuario `becario`:

```sql
CREATE USER becario IDENTIFIED BY 1234;
```

Le doy la siguiente lista de privilegios...

Conexión a la base de datos:

```sql
GRANT CREATE SESSION TO becario;
```

Modificación del número de errores en la introducción de contraseña de cualquier usuario:

```sql
CREATE PROFILE errores_contrasena LIMIT FAILED_LOGIN_ATTEMPTS 3;
GRANT ALTER PROFILE TO becario;
GRANT ALTER USER TO becario;
```

Modificación de índices en cualquier esquema, pudiendo pasarlo a quien quiera:

```sql
GRANT ALTER ANY INDEX TO becario WITH GRANT OPTION;
```

Inserción de filas en `scott.emp`, pudiendo pasarlo a quien quiera:

```sql
GRANT INSERT ON scott.emp TO becario WITH GRANT OPTION;
```

Uso de almacenamiento ilimitado en cualquier tablespace:

```sql
GRANT UNLIMITED TABLESPACE TO becario;
```

Gestión completa de usuarios:

```sql
GRANT CREATE USER TO becario;
GRANT ALTER USER TO becario;
GRANT DROP USER TO becario;
```

Gestión completa de privilegios:

```sql
GRANT GRANT ANY PRIVILEGE TO becario;
GRANT GRANT ANY OBJECT PRIVILEGE TO becario;
```

Gestión completa de roles:

```sql
GRANT CREATE ROLE TO becario;
GRANT ALTER ANY ROLE TO becario;
GRANT DROP ANY ROLE TO becario;
GRANT GRANT ANY ROLE TO becario;
```

## 2. PostgreSQL

### 2. Ejercicio 1

> Crear el usuario `becario`

En debian:

```sql
sudo adduser becario
```

En PostgreSQL:

```sql
CREATE USER becario WITH PASSWORD '1234';
```

!!! Info

    `CREATE USER` lleva implícito el atributo `LOGIN`, por lo que no será necesario darle permisos de conexión a la base de datos

### 2. Ejercicio 2

> Dar el privilegio de modificar el número de errores en la introducción de contraseña de cualquier usuario

En PostgreSQL [según este post](https://stackoverflow.com/questions/73925483/postgresql-failed-login-attempts), nativamente al menos, no existe un equivalente al `FAILED_LOGIN_ATTEMPTS` de Oracle. Existe `auth_delay`, pero no llegaríamos al mismo resultado.

Podríamos llegar al mismo resultado, pero tendríamos que una de dos:

- Usar herramientas externas como [Fail2Ban](https://github.com/fail2ban/fail2ban)
- Usar versiones modificadas de PostgreSQL como [Postgres Pro](https://postgrespro.com/)

### 2. Ejercicio 3

> Dar el privilegio de modificar índices en cualquier esquema, pudiendo pasarlo a quien quiera

Según [este](https://stackoverflow.com/questions/32432069/add-index-on-table-owned-by-other-user-in-postgres) y [este otro](https://dba.stackexchange.com/questions/114735/postgres-how-can-i-allow-index-creation-but-no-table-mutations-or-table-drops-b) post, la creación y modificación de índices en PostgreSQL funcionan un poco diferente a como es en Oracle.

Primero de todo, no existen directamente estos privilegios en PostgreSQL. Sólo el superusuario y el propietario de una tabla pueden crear/modificar índices sobre ella.

Por ejemplo, si intentamos crear un índice de una tabla que no es de ese usuario, aparece este error:

```sql
scott=> CREATE INDEX test ON dept(dname);
ERROR:  must be owner of table dept
```

### 2. Ejercicio 4

> Dar el privilegio de inserción de filas en la tabla emp de scott, pudiendo pasarlo a quien quiera

Según [este post](https://dba.stackexchange.com/questions/261991/grant-usage-to-a-schema-from-another-database) no podemos dar permisos sobre objetos de una base de datos mientras estamos conectados a otra, así que primero nos conectamos a `scott`:

```sql
\c scott
```

Damos el privilegio:

```sql
GRANT INSERT ON emp TO becario WITH GRANT OPTION;
```

!!! Info

    Por este mismo funcionamiento, no podríamos insertar datos si no estamos conectados a la misma base de datos

### 2. Ejercicio 5

> Dar el privilegio de almacenamiento ilimitado en cualquier tablespace

Aunque los tablespaces existan en PostgreSQL y se puedan listar con `\db`, no existe una equivalencia para el privilegio `UNLIMITED TABLESPACE`.

### 2. Ejercicio 6

> Dar gestión completa de usuarios

```sql
ALTER USER becario WITH CREATEROLE;
```

!!! Info

    Un usuario con el parámetro `CREATEROLE` también podrá hacer ALTER y DROP de ellos

### 2. Ejercicio 7

> Dar gestión completa de privilegios

En PostgreSQL no existe una equivalencia directa a `GRANT ANY PRIVILEGE` ni `GRANT ANY OBJECT PRIVILEGE`, pero sí que existe `WITH GRANT OPTION` al otorgar privilegios.

Si el usuario `becario` tuviera [todos los privilegios posibles](https://www.postgresql.org/docs/current/sql-grant.html) con `WITH GRANT OPTION`, llegaríamos a una situación lo más similar posible a la que teníamos en Oracle, así que eso hacemos:

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA schema_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA schema_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON DATABASE database_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON DOMAIN domain_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON FOREIGN DATA WRAPPER fdw_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON FOREIGN SERVER server_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON ALL FUNCTIONS IN SCHEMA schema_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON ALL PROCEDURES IN SCHEMA schema_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON ALL ROUTINES IN SCHEMA schema_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON LANGUAGE lang_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON LARGE OBJECT loid TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON PARAMETER configuration_parameter TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON SCHEMA schema_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON TABLESPACE tablespace_name TO becario WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON TYPE type_name TO becario WITH GRANT OPTION;
```

### 2. Ejercicio 8

> Dar gestión completa de roles

En PostgreSQL, los usuarios y los roles son lo mismo; y más específicamente no existen los usuarios, sólo los roles.

Por esto, cuando tuvimos que dar gestión completa de usuarios lo que dimos realmente fue gestión completa de roles, por lo que este paso no es necesario, ya se ha hecho.

## 3. MariaDB

### 3. Ejercicio 1

> Crear el usuario `becario`

```sql
CREATE USER 'becario'@'%' IDENTIFIED BY '1234';
```

### 3. Ejercicio 2

> Dar el privilegio de conexión a la base de datos

El privilegio `CREATE SESSION` no existe en MariaDB, porque este proceso funciona un poco diferente a Oracle.

Nada más crear un usuario se permiten las conexiones pero no deja hacer nada, porque sólo se le otorga al usuario el privilegio `USAGE`, que no tiene ningún uso real:

![usage](https://i.imgur.com/ZgKDS9S.png)

```sql
MariaDB [(none)]> show grants;
+--------------------------------------------------------------------------------------------------------+
| Grants for becario@%                                                                                   |
+--------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `becario`@`%` IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' |
+--------------------------------------------------------------------------------------------------------+
1 row in set (0.001 sec)
```

Para que un usuario pueda acceder a una base de datos en MariaDB, tiene que tener algún tipo de privilegio relacionado con los objetos.

El mínimo que le podemos dar para llegar a la equivalencia más cercana con `CREATE SESSION` es `SELECT`:

```sql
GRANT SELECT ON *.* TO 'becario'@'%';
```

### 3. Ejercicio 3

> Dar el privilegio de modificar el número de errores en la introducción de contraseña de cualquier usuario

En MariaDB los perfiles no existen de la misma manera que existen en Oracle, de hecho no se pueden crear manualmente porque **no son el mismo concepto**.

Sólo existen los comandos de visualización `SHOW PROFILE` y `SHOW PROFILES`, que incluso serán eliminados porque están obsoletos:

![deprecated](https://i.imgur.com/2Y5uZCG.png)

Se puede llegar a una equivalencia con Oracle pero no se podría hacer ni con privilegios ni con perfiles, tendría que ser fuera de la base de datos añadiendo lo siguiente a este fichero:

```shell
sudo nano /etc/mysql/my.cnf
```

```shell
[mariadb]
max_password_errors=3
```

Reinicio:

```shell
sudo systemctl restart mariadb
```

Compruebo que ha funcionado:

```sql
MariaDB [(none)]> SELECT @@max_password_errors;
+-----------------------+
| @@max_password_errors |
+-----------------------+
|                     3 |
+-----------------------+
1 row in set (0.001 sec)
```

### 3. Ejercicio 4

> Dar el privilegio de modificar índices en cualquier esquema, pudiendo pasarlo a quien quiera

En MariaDB no existe un `ALTER INDEX` como en Oracle, simplemente existe un privilegio general:

```sql
GRANT INDEX ON *.* TO 'becario'@'%' IDENTIFIED BY '1234' WITH GRANT OPTION;
```

### 3. Ejercicio 5

> Dar el privilegio de inserción de filas en scott.emp, pudiendo pasarlo a quien quiera

```sql
GRANT INSERT ON scott.emp TO 'becario'@'%' WITH GRANT OPTION;
```

### 3. Ejercicio 6

> Dar el privilegio de almacenamiento ilimitado en cualquier tablespace

No existe una equivalencia para el privilegio `UNLIMITED TABLESPACE` en MariaDB.

### 3. Ejercicio 7

> Dar gestión completa de usuarios

```sql
GRANT CREATE USER ON *.* TO 'becario'@'%';
```

El privilegio `CREATE USER` en MariaDB incluye las funciones de `ALTER USER` y `DROP USER` de Oracle.

### 3. Ejercicio 8

> Dar gestión completa de privilegios

En MariaDB tampoco existe `GRANT ANY PRIVILEGE` ni `GRANT ANY OBJECT PRIVILEGE`, pero como existe `WITH GRANT OPTION` podemos llegar a una solución similar a lo que hicimos en PostgreSQL.

La diferencia es que en MariaDB existe `ALL PRIVILEGES`, facilitándonos el trabajo en una línea:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'becario'@'%' WITH GRANT OPTION;
```

### 3. Ejercicio 9

> Dar gestión completa de roles

Para gestionar los roles lo único que se necesita es el privilegio `CREATE USER`, que ya dimos antes.
