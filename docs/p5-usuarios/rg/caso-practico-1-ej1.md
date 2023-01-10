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

## PostgreSQL





## MariaDB


