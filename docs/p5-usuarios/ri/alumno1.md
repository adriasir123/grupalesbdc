# Alumno 1 (ORACLE)

create user testing identified by 1234;

## Ejercicio 1

### Enunciado

Crear el rol `ROLPRACTICA1` con privilegios:

- Conexión a la base de datos
- Creación de tablas
- Creación de vistas
- Inserción de datos en la tabla EMP de SCOTT

### Realización

Creo el rol:

```sql
CREATE ROLE rolpractica1;
```

Le asigno los privilegios:

```sql
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW TO rolpractica1;
GRANT INSERT ON scott.emp TO rolpractica1;
```

### Comprobaciones

Compruebo que se ha creado el rol:

```sql
SELECT role
FROM DBA_ROLES
WHERE role = 'ROLPRACTICA1';
```

```sql
ROLE
--------------------------------------------------------------------------------------------------------------------------------
ROLPRACTICA1
```

Compruebo que los privilegios están asignados:

```sql
SELECT privilege
FROM DBA_SYS_PRIVS
WHERE grantee = 'ROLPRACTICA1';
```

```sql
PRIVILEGE
----------------------------------------
CREATE TABLE
CREATE VIEW
CREATE SESSION
```

```sql
SELECT privilege
FROM DBA_TAB_PRIVS
WHERE grantee = 'ROLPRACTICA1';
```

```sql
PRIVILEGE
----------------------------------------
INSERT
```

Le asigno el rol al usuario `testing`:

```sql
GRANT ROLPRACTICA1 TO testing;

Grant succeeded.
```

Me conecto:

```sql
connect testing/1234
Connected.
ORA-01031: insufficient privileges
ORA-01078: failure in processing system parameters

Session altered.
```

!!! Note

    Esos errores de privilegios no son relevantes, ya que como vemos he conectado correctamente.  
    Aclaro que simplemente aparecen porque tengo un script personalizado en sqlplus que se lanza en cada conexión, que hace ciertas acciones; algunas necesitan permisos de administrador.

Creo una tabla:

```sql
CREATE TABLE tabla (
  id NUMBER(10) NOT NULL,
  campo VARCHAR2(50) NOT NULL,
  CONSTRAINT tabla_pk PRIMARY KEY (id)
);

Table created.
```

Creo una vista:

```sql
CREATE OR REPLACE VIEW view_tabla AS
    SELECT *
    FROM tabla;
```

```sql
CREATE OR REPLACE VIEW view_tabla AS
  2      SELECT *
    FROM tabla;

View created.
```

Inserto un registro en `scott.emp`:

```sql
INSERT INTO scott.emp VALUES (8000,'TESTER','TESTER',7782,to_date('23-1-1982','dd-mm-yyyy'),1300,NULL,10);

1 row created.
```

## Ejercicio 2

### 2. Enunciado

Crear el usuario USRPRACTICA1 con el tablespace USERS por defecto y averiguar qué cuota se le ha asignado por defecto en el mismo. Sustitúyela por una cuota de 1M.

### 2. Realización

Creo el usuario:

```sql
CREATE USER USRPRACTICA1 IDENTIFIED BY 1234 DEFAULT TABLESPACE USERS;

User created.
```

Por defecto, la cuota de los usuarios sobre los tablespaces es 0:

```sql
SELECT tablespace_name,username,max_bytes
FROM DBA_TS_QUOTAS
WHERE username = 'USRPRACTICA1' AND tablespace_name = 'USERS';
```

```sql
SELECT tablespace_name,username,max_bytes
  2  FROM DBA_TS_QUOTAS
WHERE username = 'USRPRACTICA1' AND tablespace_name = 'USERS';

no rows selected
```

No hay registros porque `USRPRACTICA1` todavía no tiene cuota. Si quisiéramos insertar datos, no podríamos, aunque para que podamos hacer esto antes primero tenemos que dar algunos permisos:

```sql
GRANT CREATE SESSION, CREATE TABLE TO USRPRACTICA1;
```

Ahora ya sí, comprobamos que no podemos meter datos:

```sql
CREATE TABLE tabla (
  id NUMBER(10) NOT NULL,
  campo VARCHAR2(50) NOT NULL,
  CONSTRAINT tabla_pk PRIMARY KEY (id)
);

Table created.

INSERT INTO tabla VALUES (1,'test');
INSERT INTO tabla VALUES (1,'test')
            *
ERROR at line 1:
ORA-01950: no privileges on tablespace 'USERS'
```

!!! Info

    Aunque diga tabla creada, realmente no se ha creado porque no tenemos cuota.  
    Cuando pasa esto, estamos experimentando el mecanismo `deferred_segment_creation` de Oracle, que básicamente "retrasa" el allocation real de datos hasta que la tabla contiene datos.

Le cambio la cuota:

```sql
ALTER USER USRPRACTICA1 QUOTA 1M ON USERS;

User altered.
```

Compruebo que ya sí aparece la cuota registrada:

```sql
SELECT tablespace_name,username,max_bytes
FROM DBA_TS_QUOTAS
WHERE username = 'USRPRACTICA1' AND tablespace_name = 'USERS';
```

```sql
SELECT tablespace_name,username,max_bytes
  2  FROM DBA_TS_QUOTAS
WHERE username = 'USRPRACTICA1' AND tablespace_name = 'USERS';

TABLE USERNAME	    MAX_BYTES
----- ------------ ----------
USERS USRPRACTICA1    1048576
```

Ya podría insertar el registro:

```sql
INSERT INTO tabla VALUES (1,'test');

1 row created.

select * from tabla;

	ID CAMPO
---------- --------------------------------------------------
	 1 test
```

## Ejercicio 3

### 3. Enunciado

Modificar el usuario `USRPRACTICA1` para que tenga cuota 0 en el tablespace SYSTEM.

### 3. Realización

```sql
ALTER USER USRPRACTICA1 QUOTA 0 ON system;

User altered.
```

### 3. Comprobaciones

Muestro que no hay cuota:

```sql
SELECT tablespace_name,username,max_bytes
FROM DBA_TS_QUOTAS
WHERE username = 'USRPRACTICA1' AND tablespace_name = 'SYSTEM';

no rows selected
```

Muestro que no puedo insertar datos en `SYSTEM`:

```sql
CREATE TABLE tabla_system (
  id NUMBER(10) NOT NULL,
  campo VARCHAR2(50) NOT NULL,
  CONSTRAINT tabla_system_pk PRIMARY KEY (id)
) TABLESPACE SYSTEM;

Table created.

INSERT INTO tabla_system VALUES (1,'test');
INSERT INTO tabla_system VALUES (1,'test')
            *
ERROR at line 1:
ORA-01536: space quota exceeded for tablespace 'SYSTEM'
```

## Ejercicio 4

### 4. Enunciado

Conceder `ROLPRACTICA1` a `USRPRACTICA1`.











    5. Concede a USRPRACTICA1 el privilegio de crear tablas e insertar datos en el esquema de cualquier usuario. Prueba el privilegio. Comprueba si puede modificar la estructura o eliminar las tablas creadas.

    6. Concede a USRPRACTICA1 el privilegio de leer la tabla DEPT de SCOTT con la posibilidad de que lo pase a su vez a terceros usuarios.

    7. Comprueba que USRPRACTICA1 puede realizar todas las operaciones previstas en el rol.

    8. Quita a USRPRACTICA1 el privilegio de crear vistas. Comprueba que ya no puede hacerlo.

    9. Crea un perfil NOPARESDECURRAR que limita a dos el número de minutos de inactividad permitidos en una sesión.

    10. Activa el uso de perfiles en ORACLE.

    11. Asigna el perfil creado a USRPRACTICA1 y comprueba su correcto funcionamiento.

    12. Crea un perfil CONTRASEÑASEGURA especificando que la contraseña caduca mensualmente y sólo se permiten tres intentos fallidos para acceder a la cuenta. En caso de superarse, la cuenta debe quedar bloqueada indefinidamente.

    13. Asigna el perfil creado a USRPRACTICA1 y comprueba su funcionamiento. Desbloquea posteriormente al usuario.

    14. Consulta qué usuarios existen en tu base de datos.

    15. Elige un usuario concreto y consulta qué cuota tiene sobre cada uno de los tablespaces.

    16. Elige un usuario concreto y muestra qué privilegios de sistema tiene asignados.

    17. Elige un usuario concreto y muestra qué privilegios sobre objetos tiene asignados.

    18. Consulta qué roles existen en tu base de datos.

    19. Elige un rol concreto y consulta qué usuarios lo tienen asignado.

    20. Elige un rol concreto y averigua si está compuesto por otros roles o no.

    21. Consulta qué perfiles existen en tu base de datos.

    22. Elige un perfil y consulta qué límites se establecen en el mismo.

    23. Muestra los nombres de los usuarios que tienen limitado el número de sesiones concurrentes.

    24. Realiza un procedimiento que reciba un nombre de usuario y un privilegio de sistema y nos muestre el mensaje 'SI, DIRECTO' si el usuario tiene ese privilegio concedido directamente, 'SI, POR ROL' si el usuario tiene ese privilegio en alguno de los roles que tiene concedidos y un 'NO' si el usuario no tiene dicho privilegio.

    25. Realiza un procedimiento llamado MostrarNumSesiones que reciba un nombre de usuario y muestre el número de sesiones concurrentes que puede tener abiertas como máximo y las que tiene abiertas realmente.