# Alumno 1 (ORACLE)

create user testing identified by 1234;

## Ejercicio 1

### 1. Enunciado

Crear el rol `ROLPRACTICA1` con privilegios:

- Conexión a la base de datos
- Creación de tablas
- Creación de vistas
- Inserción de datos en la tabla EMP de SCOTT

### 1. Realización

Creo el rol:

```sql
CREATE ROLE rolpractica1;
```

Le asigno los privilegios:

```sql
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW TO rolpractica1;
GRANT INSERT ON scott.emp TO rolpractica1;
```

### 1. Comprobaciones

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

### 4. Realización

```sql
GRANT ROLPRACTICA1 TO USRPRACTICA1;

Grant succeeded.
```

### 4. Comprobaciones

Muestro que tiene el rol asignado:

```sql
SELECT granted_role,grantee
FROM DBA_ROLE_PRIVS
WHERE granted_role = 'ROLPRACTICA1' AND grantee = 'USRPRACTICA1';
```

```sql
GRANTED_ROLE GRANTEE
------------ --------------------------------------------------------------------------------------------------------------------------------
ROLPRACTICA1 USRPRACTICA1
```

!!! Info

    El funcionamiento del rol ya se ha probado en el ejercicio 1

## Ejercicio 5

### 5. Enunciado

Conceder a USRPRACTICA1 el privilegio de crear tablas e insertar datos en el esquema de cualquier usuario. Probar los privilegios. Comprobar si se puede modificar la estructura o eliminar la tabla creada.

### 5. Realización

Doy privilegios:

```sql
GRANT CREATE ANY TABLE, INSERT ANY TABLE, CREATE ANY INDEX TO USRPRACTICA1;

Grant succeeded.
```

!!! Info

    No se menciona en el enunciado, pero el privilegio `CREATE ANY INDEX` es necesario

Le doy cuota a `TESTING` para que luego se puedan insertar datos:

```sql
ALTER USER TESTING QUOTA 1M ON USERS;
```

### 5. Comprobaciones

Creo una tabla e inserto un registro en `TESTING` desde `USRPRACTICA1`:

```sql
CREATE TABLE TESTING.tabla (
  id NUMBER(10) NOT NULL,
  campo VARCHAR2(50) NOT NULL,
  CONSTRAINT tabla_testing_pk PRIMARY KEY (id)
);

Table created.

INSERT INTO TESTING.tabla VALUES (1,'test');

1 row created.
```

Como desde `USRPRACTICA1` no tengo privilegios de SELECT sobre esa tabla, la muestro desde `SYS`:

```sql
connect sys/sys as sysdba
Connected.
ORA-01081: cannot start already-running ORACLE - shut it down first

Session altered.

select * from TESTING.tabla;

        ID CAMPO
---------- --------------------------------------------------
         1 test
```

Vuelvo a `USRPRACTICA1` y compruebo si se puede modificar la estructura de la tabla creada o eliminarla:

```sql
connect USRPRACTICA1/1234
Connected.
ORA-01031: insufficient privileges
ORA-01078: failure in processing system parameters

Session altered.

ALTER TABLE TESTING.tabla DROP COLUMN CAMPO;
ALTER TABLE TESTING.tabla DROP COLUMN CAMPO
*
ERROR at line 1:
ORA-01031: insufficient privileges


DROP TABLE TESTING.tabla;
DROP TABLE TESTING.tabla
                   *
ERROR at line 1:
ORA-01031: insufficient privileges
```

No se puede, porque en ningún momento le hemos dado a `USRPRACTICA1` los privilegios necesarios para estas acciones.

## Ejercicio 6

### 6. Enunciado

Conceder a `USRPRACTICA1` el privilegio de leer la tabla DEPT de SCOTT con la posibilidad de que lo pase a su vez a terceros usuarios.

### 6. Realización

```sql
GRANT READ ON scott.dept TO USRPRACTICA1 WITH GRANT OPTION;

Grant succeeded.
```

!!! Info

    El privilegio `READ` se añadió en la versión 12.1 de Oracle, y es equivalente a `SELECT` pero más seguro; no se da implícitamente privilegio de LOCK sobre el objeto.  
    Si se otorga el privilegio `SELECT` a un usuario sobre una tabla por ejemplo, éste podría bloquearla, pero con `READ` no podría.

### 6. Comprobaciones

Me conecto a `USRPRACTICA1` y pruebo el privilegio:

```sql
connect USRPRACTICA1/1234
Connected.
ORA-01031: insufficient privileges
ORA-01078: failure in processing system parameters

Session altered.

SELECT * FROM scott.dept;

    DEPTNO DNAME          LOC
---------- -------------- -------------
        10 ACCOUNTING     NEW YORK
        20 RESEARCH       DALLAS
        30 SALES          CHICAGO
        40 OPERATIONS     BOSTON
```

Pruebo a pasarle el privilegio a `TESTING`:

```sql
GRANT READ ON scott.dept TO TESTING;

Grant succeeded.
```

Me conecto a `TESTING` y pruebo que puedo hacer lo mismo:

```sql
connect TESTING/1234
Connected.
ORA-01031: insufficient privileges
ORA-01078: failure in processing system parameters

Session altered.

SELECT * FROM scott.dept;

    DEPTNO DNAME          LOC
---------- -------------- -------------
        10 ACCOUNTING     NEW YORK
        20 RESEARCH       DALLAS
        30 SALES          CHICAGO
        40 OPERATIONS     BOSTON
```

## Ejercicio 7

### 7. Enunciado

Comprobar que `USRPRACTICA1` puede realizar todas las operaciones previstas en el rol `ROLPRACTICA1` (se dio en el [ejercicio 4](#ejercicio-4)).

### 7. Realización

!!! Info

    El funcionamiento del rol ya se ha probado en el [ejercicio 1](#ejercicio-1)

## Ejercicio 8

### 8. Enunciado

Quitar a `USRPRACTICA1` el privilegio de crear vistas. Comprobar que ya no puede hacerlo.

### 8. Realización

```sql
REVOKE CREATE VIEW FROM ROLPRACTICA1;

Revoke succeeded.
```

!!! Info

    El privilegio `CREATE VIEW` se dio a `ROLPRACTICA1`, no a `USRPRACTICA1`.  
    Este usuario puede crear vistas por el rol, no porque tenga el privilegio otorgado directamente, así que el privilegio se tiene que quitar del rol.

### 8. Comprobaciones

Me conecto a `USRPRACTICA1`, y aprovechando que en el [ejercicio 6](#ejercicio-6) se le dio permisos de lectura sobre `scott.dept`, intento crear una vista de esa tabla:

```sql
connect USRPRACTICA1/1234
Connected.
ORA-01031: insufficient privileges
ORA-01078: failure in processing system parameters

Session altered.
```

```sql
CREATE OR REPLACE VIEW view_dept AS
    SELECT *
    FROM scott.dept;
```

```sql
CREATE OR REPLACE VIEW view_dept AS
                       *
ERROR at line 1:
ORA-01031: insufficient privileges
```

## Ejercicio 9

### 9. Enunciado

Crear el perfil `NOPARESDECURRAR` que limite a dos el número de minutos de inactividad permitidos en una sesión.

### 9. Realización

```sql
CREATE PROFILE NOPARESDECURRAR LIMIT IDLE_TIME 2;

Profile created.
```

### 9. Comprobaciones

Compruebo que se ha creado:

```sql
SELECT profile,resource_name,limit
FROM DBA_PROFILES
WHERE profile = 'NOPARESDECURRAR';

PROFILE                                                                                                                          RESOURCE_NAME                    LIMIT
-------------------------------------------------------------------------------------------------------------------------------- -------------------------------- --------------------------------------------------------------------------------------------------------------------------------
NOPARESDECURRAR                                                                                                                  COMPOSITE_LIMIT                  DEFAULT
NOPARESDECURRAR                                                                                                                  SESSIONS_PER_USER                DEFAULT
NOPARESDECURRAR                                                                                                                  CPU_PER_SESSION                  DEFAULT
NOPARESDECURRAR                                                                                                                  CPU_PER_CALL                     DEFAULT
NOPARESDECURRAR                                                                                                                  LOGICAL_READS_PER_SESSION        DEFAULT
NOPARESDECURRAR                                                                                                                  LOGICAL_READS_PER_CALL           DEFAULT
NOPARESDECURRAR                                                                                                                  IDLE_TIME                        2
NOPARESDECURRAR                                                                                                                  CONNECT_TIME                     DEFAULT
NOPARESDECURRAR                                                                                                                  PRIVATE_SGA                      DEFAULT
NOPARESDECURRAR                                                                                                                  FAILED_LOGIN_ATTEMPTS            DEFAULT
NOPARESDECURRAR                                                                                                                  PASSWORD_LIFE_TIME               DEFAULT
NOPARESDECURRAR                                                                                                                  PASSWORD_REUSE_TIME              DEFAULT
NOPARESDECURRAR                                                                                                                  PASSWORD_REUSE_MAX               DEFAULT
NOPARESDECURRAR                                                                                                                  PASSWORD_VERIFY_FUNCTION         DEFAULT
NOPARESDECURRAR                                                                                                                  PASSWORD_LOCK_TIME               DEFAULT
NOPARESDECURRAR                                                                                                                  PASSWORD_GRACE_TIME              DEFAULT
NOPARESDECURRAR                                                                                                                  INACTIVE_ACCOUNT_TIME            DEFAULT

17 rows selected.
```

!!! Info

    Aunque no se lo indiquemos, Oracle nos añade unos límites por defecto, pero lo importante es que el límite que nosotros hemos añadido es correcto

## Ejercicio 10

### 10. Enunciado

Activar el uso de perfiles.

### 10. Realización

```sql
ALTER SYSTEM SET RESOURCE_LIMIT=TRUE;

System altered.
```

### 10. Comprobaciones

Compruebo el estado del parámetro:

```sql
show parameter resource_limit;

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
resource_limit                       boolean     TRUE
```

## Ejercicio 11

### 11. Enunciado

Asignar el perfil creado a `USRPRACTICA1` y comprobar su correcto funcionamiento.

### 11. Realización

```sql
ALTER USER USRPRACTICA1 PROFILE NOPARESDECURRAR;

User altered.
```

### 11. Comprobaciones

Me conecto a `USRPRACTICA1` y tras 2 minutos, esto es lo que pasa:

```sql
connect USRPRACTICA1/1234
Connected.
ORA-01031: insufficient privileges
ORA-01078: failure in processing system parameters

Session altered.

select user from dual;
select user from dual
*
ERROR at line 1:
ORA-02396: exceeded maximum idle time, please connect again
```

!!! Warning

    Cuando sobrepasamos el tiempo de inactividad no se corta la conexión automáticamente.  
    El error aparecerá cuando intentemos realizar alguna acción, por eso he escrito un comando de ejemplo para la prueba

## Ejercicio 12

### 12. Enunciado

Crear el perfil `CONTRASENASEGURA` con los límites:

- Caducidad mensual
- Máximo tres intentos fallidos permitidos de acceso
- Bloqueo de cuenta indefinido si se superan los intentos

### 12. Realización

```sql
CREATE PROFILE CONTRASENASEGURA LIMIT
    PASSWORD_LIFE_TIME 30
    FAILED_LOGIN_ATTEMPTS 4
    PASSWORD_LOCK_TIME UNLIMITED;
```

```sql
Profile created.
```

### 12. Comprobaciones

Muestro que se ha creado con los límites dichos:

```sql
SELECT profile,resource_name,limit
FROM DBA_PROFILES
WHERE profile = 'CONTRASENASEGURA'
  AND resource_name IN ('PASSWORD_LIFE_TIME','FAILED_LOGIN_ATTEMPTS','PASSWORD_LOCK_TIME');
```

```sql
PROFILE                                                                                                                          RESOURCE_NAME                    LIMIT
-------------------------------------------------------------------------------------------------------------------------------- -------------------------------- --------------------------------------------------------------------------------------------------------------------------------
CONTRASENASEGURA                                                                                                                 FAILED_LOGIN_ATTEMPTS            4
CONTRASENASEGURA                                                                                                                 PASSWORD_LIFE_TIME               30
CONTRASENASEGURA                                                                                                                 PASSWORD_LOCK_TIME               UNLIMITED
```

## Ejercicio 13

### 13. Enunciado

Asignar el perfil `CONTRASENASEGURA` a `USRPRACTICA1` y comprobar su funcionamiento. Desbloquear posteriormente al usuario.

### 13. Realización

```sql
ALTER USER USRPRACTICA1 PROFILE CONTRASENASEGURA;

User altered.
```

### 13. Comprobaciones

Me equivoco 4 veces en la contraseña para probar el funcionamiento:

```sql
connect USRPRACTICA1/123
ERROR:
ORA-01017: invalid username/password; logon denied


Warning: You are no longer connected to ORACLE.
connect USRPRACTICA1/123
ERROR:
ORA-01017: invalid username/password; logon denied


connect USRPRACTICA1/123
ERROR:
ORA-01017: invalid username/password; logon denied


connect USRPRACTICA1/123
ERROR:
ORA-01017: invalid username/password; logon denied


connect USRPRACTICA1/1234
ERROR:
ORA-28000: The account is locked.
```

Como vemos, aunque Oracle no nos avise explícitamente, al 4 intento fallido la cuenta se ha bloqueado.  

Lo sabemos porque si intentamos acceder una 5 vez, aunque sea con la contraseña correcta, nos dará el error `ORA-28000: The account is locked.`

Desbloqueo la cuenta:

```sql
ALTER USER USRPRACTICA1 IDENTIFIED BY 1234 ACCOUNT UNLOCK;

User altered.
```

Pruebo que se ha desbloqueado:

```sql
connect USRPRACTICA1/1234
Connected.
ORA-01031: insufficient privileges
ORA-01078: failure in processing system parameters

Session altered.
```

## Ejercicio 14

### 14. Enunciado

Consultar qué usuarios existen en la base de datos.

### 14. Realización

```sql
SELECT USERNAME
FROM DBA_USERS;

USERNAME
--------------------------------------------------------------------------------------------------------------------------------
SYS
SYSTEM
XS$NULL
OJVMSYS
LBACSYS
OUTLN
SYS$UMF
DBSNMP
APPQOSSYS
DBSFWUSER
GGSYS
ANONYMOUS
CTXSYS
DVSYS
DVF
GSMADMIN_INTERNAL
MDSYS
OLAPSYS
XDB
WMSYS
GSMCATUSER
MDDATA
BECARIO
SYSBACKUP
REMOTE_SCHEDULER_AGENT
GSMUSER
SYSRAC
GSMROOTUSER
SI_INFORMTN_SCHEMA
AUDSYS
EXAMENBD34
DIP
ORDPLUGINS
TESTING
SYSKM
ORDDATA
ORACLE_OCM
SCOTT
HIPODROMO
SYSDG
ORDSYS
USRPRACTICA1

42 rows selected.
```

## Ejercicio 15

### 15. Enunciado

Elegir un usuario concreto y consultar qué cuota tiene sobre cada uno de los tablespaces.

### 15. Realización

Elijo `AUDSYS`:

```sql
SELECT tablespace_name,username,max_bytes
FROM DBA_TS_QUOTAS
WHERE username = 'AUDSYS';

TABLESPACE_NAME                USERNA  MAX_BYTES
------------------------------ ------ ----------
SYSAUX                         AUDSYS         -1
```

Sólo tiene cuota sobre 1 tablespace, `SYSAUX`, y el tamaño -1 quiere decir que es ilimitada.

## Ejercicio 16

### 16. Enunciado

Elegir un usuario concreto y mostrar qué privilegios de sistema tiene asignados.

### 16. Realización

Elijo `SYSTEM`:

```sql
SELECT privilege
FROM DBA_SYS_PRIVS
WHERE grantee = 'SYSTEM';
```

```sql
PRIVILEGE
----------------------------------------
GLOBAL QUERY REWRITE
CREATE TABLE
DEQUEUE ANY QUEUE
ENQUEUE ANY QUEUE
SELECT ANY TABLE
MANAGE ANY QUEUE
UNLIMITED TABLESPACE
CREATE MATERIALIZED VIEW

8 rows selected.
```

## Ejercicio 17

### 17. Enunciado

Elegir un usuario concreto y mostrar qué privilegios sobre objetos tiene asignados.

### 17. Realización

Elijo `SYS`:

```sql
SELECT privilege,table_name
FROM DBA_TAB_PRIVS
WHERE grantee = 'SYS';
```

```sql
PRIVILEGE                                TABLE_NAME
---------------------------------------- --------------------------------------------------------------------------------------------------------------------------------
EXECUTE                                  LBAC_STANDARD
EXECUTE                                  LBAC_SERVICES
SELECT                                   DBA_DV_STATUS
EXECUTE                                  CONFIGURE_DV_INTERNAL
SELECT                                   OL$
SELECT                                   OL$HINTS
SELECT                                   OL$NODES

7 rows selected.
```

## Ejercicio 18

### 18. Enunciado

Consultar qué roles existen en la base de datos.

### 18. Realización

```sql
SELECT role FROM DBA_ROLES;
```

[Output largo](https://gist.github.com/adriasir123/51f018d18d794f9d66db8c72aa6a7134)

## Ejercicio 19

### 19. Enunciado

Elegir un rol concreto y consultar qué usuarios lo tienen asignado.

### 19. Realización

Elijo `CONNECT`:

```sql
SELECT grantee
FROM DBA_ROLE_PRIVS
WHERE granted_role = 'CONNECT';
```

```sql
GRANTEE
--------------------------------------------------------------------------------------------------------------------------------
SYS
DV_ACCTMGR
GSMADMIN_ROLE
GSM_POOLADMIN_ROLE
GSMROOTUSER_ROLE
SCOTT

6 rows selected.
```

## Ejercicio 20

### 20. Enunciado

Elegir un rol concreto y averiguar si está compuesto por otros roles o no.

### 20. Realización

Elijo `DBA`:

```sql
SELECT granted_role
FROM DBA_ROLE_PRIVS
WHERE grantee = 'DBA';
```

```sql
GRANTED_ROLE
--------------------------------------------------------------------------------------------------------------------------------
EXECUTE_CATALOG_ROLE
DATAPUMP_IMP_FULL_DATABASE
SCHEDULER_ADMIN
XDBADMIN
OLAP_DBA
CAPTURE_ADMIN
SELECT_CATALOG_ROLE
DATAPUMP_EXP_FULL_DATABASE
GATHER_SYSTEM_STATISTICS
WM_ADMIN_ROLE
EXP_FULL_DATABASE
OPTIMIZER_PROCESSING_RATE
EM_EXPRESS_ALL
IMP_FULL_DATABASE
XDB_SET_INVOKER
JAVA_ADMIN
OLAP_XS_ADMIN

17 rows selected.
```

Esos son todos los roles por los que está compuesto el rol `DBA`.

## Ejercicio 21

### 21. Enunciado

Consultar qué perfiles existen en la base de datos.

### 21. Realización

```sql
SELECT DISTINCT profile
FROM DBA_PROFILES;

PROFILE
--------------------------------------------------------------------------------------------------------------------------------
CONTRASENASEGURA
DEFAULT
NOPARESDECURRAR
ORA_STIG_PROFILE
```

## Ejercicio 22

### 22. Enunciado

Elegir un perfil y consultar qué límites se establecen en el mismo.

### 22. Realización

Elijo `DEFAULT`:

```sql
SELECT resource_name,limit
FROM DBA_PROFILES
WHERE profile = 'DEFAULT';
```

```sql
RESOURCE_NAME                    LIMIT
-------------------------------- --------------------------------------------------------------------------------------------------------------------------------
COMPOSITE_LIMIT                  UNLIMITED
SESSIONS_PER_USER                UNLIMITED
CPU_PER_SESSION                  UNLIMITED
CPU_PER_CALL                     UNLIMITED
LOGICAL_READS_PER_SESSION        UNLIMITED
LOGICAL_READS_PER_CALL           UNLIMITED
IDLE_TIME                        UNLIMITED
CONNECT_TIME                     UNLIMITED
PRIVATE_SGA                      UNLIMITED
FAILED_LOGIN_ATTEMPTS            10
PASSWORD_LIFE_TIME               180
PASSWORD_REUSE_TIME              UNLIMITED
PASSWORD_REUSE_MAX               UNLIMITED
PASSWORD_VERIFY_FUNCTION         NULL
PASSWORD_LOCK_TIME               1
PASSWORD_GRACE_TIME              7
INACTIVE_ACCOUNT_TIME            UNLIMITED

17 rows selected.
```

## Ejercicio 23

### 23. Enunciado

Mostrar los nombres de los usuarios que tienen limitado el número de sesiones concurrentes.

### 23. Realización

Ahora mismo no tengo ningún perfil que tenga `SESSIONS_PER_USER` limitado:

```sql
SELECT profile
FROM DBA_PROFILES
WHERE resource_name = 'SESSIONS_PER_USER'
  AND limit NOT IN ('UNLIMITED', 'DEFAULT');
```

```sql
no rows selected
```

Por lo tanto, por ejemplo, voy a limitar `SESSIONS_PER_USER` en el perfil `NOPARESDECURRAR`:

```sql
ALTER PROFILE NOPARESDECURRAR LIMIT SESSIONS_PER_USER 1;
```

Si vuelvo a lanzar la consulta anterior, ya aparece el perfil:

```sql
SELECT profile
  2  FROM DBA_PROFILES
WHERE resource_name = 'SESSIONS_PER_USER'
  AND limit NOT IN ('UNLIMITED', 'DEFAULT');

PROFILE
--------------------------------------------------------------------------------------------------------------------------------
NOPARESDECURRAR
```

Además, ahora mismo tampoco tengo ningún usuario con el perfil `NOPARESDECURRAR` asignado:

```sql
SELECT username
FROM DBA_USERS
WHERE profile = 'NOPARESDECURRAR';
```

```sql
no rows selected
```

Así que, por ejemplo, se lo vuelvo a asignar a `USRPRACTICA1`:

```sql
ALTER USER USRPRACTICA1 PROFILE NOPARESDECURRAR;

User altered.
```

Si vuelvo a lanzar la consulta anterior, ya aparece el usuario con el perfil:

```sql
SELECT username
  2  FROM DBA_USERS
WHERE profile = 'NOPARESDECURRAR';

USERNAME
--------------------------------------------------------------------------------------------------------------------------------
USRPRACTICA1
```

### 23. Comprobaciones

Después de hacer todo lo anterior, ya podemos mostrar directamente los usuarios que tienen limitado el número de sesiones (que será 1, el que he "forzado" para que la siguiente consulta no saliera vacía):

```sql
SELECT username
FROM DBA_USERS
WHERE profile IN (
        SELECT profile
        FROM DBA_PROFILES
        WHERE resource_name = 'SESSIONS_PER_USER'
          AND limit NOT IN ('UNLIMITED', 'DEFAULT')
);
```

```sql
USERNAME
--------------------------------------------------------------------------------------------------------------------------------
USRPRACTICA1
```

Además, puedo comprobar que efectivamente las sesiones concurrentes de este usuario están limitadas a 1:

![sesiones1](https://i.imgur.com/r3QIFDq.png)

## Ejercicio 24

### 24. Enunciado

Realizar un procedimiento que reciba un nombre de usuario y un privilegio de sistema y nos muestre el mensaje 'SI, DIRECTO' si el usuario tiene ese privilegio concedido directamente, 'SI, POR ROL' si el usuario tiene ese privilegio en alguno de los roles que tiene concedidos y un 'NO' si el usuario no tiene dicho privilegio.

### 24. Realización

```sql
CREATE OR REPLACE PROCEDURE comprobar_privilegio (
        p_username IN VARCHAR2,
        p_privilege IN VARCHAR2
) AS
        v_grant_directo BOOLEAN;
        v_role_grant   BOOLEAN;
BEGIN
 -- Comprobar si el privilegio está concedido directamente al usuario
        SELECT COUNT(*) INTO v_grant_directo
        FROM DBA_SYS_PRIVS
        WHERE grantee = p_username AND privilege = p_privilege;
 -- Comprobar si el privilegio está concedido a través de algún rol del usuario
        SELECT COUNT(*) INTO v_role_grant
        FROM DBA_ROLE_PRIVS
        WHERE grantee = p_username AND privilege = p_privilege;
 -- Mostrar resultado
        IF v_grant_directo = 1 THEN
                dbms_output.put_line('SI, DIRECTO');
        ELSIF v_role_grant = 1 THEN
                dbms_output.put_line('SI, POR ROL');
        ELSE
                dbms_output.put_line('NO');
        END IF;
END;
/
```











## Ejercicio 25

### 25. Enunciado

Realizar un procedimiento llamado `MostrarNumSesiones` que:

- Reciba un nombre de usuario
- Muestre el número de sesiones concurrentes que puede tener abiertas como máximo
- Muestre el número de sesiones concurrentes que tiene abiertas actualmente

### 25. Código

```sql
CREATE OR REPLACE PROCEDURE MostrarNumSesiones (
        usuarioentrante IN VARCHAR2
) IS
        v_perfilsacado     VARCHAR2(30);
        v_limitmaxsesiones VARCHAR2(40);
        v_numsesiones      NUMBER;
BEGIN
 -- Sacamos el perfil del usuario solicitado
        SELECT profile INTO v_perfilsacado
        FROM DBA_USERS
        WHERE username = usuarioentrante;
 -- Sacamos el número máximo de sesiones permitidas para ese usuario
        SELECT limit INTO v_limitmaxsesiones
        FROM DBA_PROFILES
        WHERE profile = v_perfilsacado
                AND resource_name = 'SESSIONS_PER_USER';
 -- Sacamos las sesiones abiertas por ese usuario actualmente
        SELECT COUNT(username) INTO v_numsesiones
        FROM V$SESSION
        WHERE username = usuarioentrante;
        DBMS_OUTPUT.PUT_LINE('El usuario '|| usuarioentrante || ' tiene un limite de sesiones concurrentes permitidas de: ' || v_limitmaxsesiones);
        DBMS_OUTPUT.PUT_LINE('Pero el numero real de sesiones que tiene abiertas son: '|| v_numsesiones);
END;
/
```

### 25. Comprobaciones

Primero dejo abiertas 2 sesiones con el usuario `TESTING`:

![2sesionestesting](https://i.imgur.com/P3tPeN3.png)

Lanzo el procedimiento y pruebo que funciona:

```sql
EXEC MostrarNumSesiones('TESTING');
El usuario TESTING tiene un limite de sesiones concurrentes permitidas de: UNLIMITED
Pero el numero real de sesiones que tiene abiertas son: 2

PL/SQL procedure successfully completed.
```
