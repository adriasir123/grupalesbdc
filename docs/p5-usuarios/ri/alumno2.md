# Alumno 2

## Postgres:

---

### EJERCICIO 1

Averigua que privilegios de sistema hay en Postgres y como se asignan a un usuario.

Para asignar privilegios de sistema a un usuario o grupos de usuario en concreto, en Postgres, debemos realizar la creación de roles, a los cuáles se les irán otorgando permisos. 

Podemos asignar los siguientes privilegios a un rol:

- superuser status: Se otorga a un rol los privilegios de un super usuario.

- database creation: Se otorga a un rol los privilegios de crear una base de datos.

- role creation:  Se le otorga al rol el privilegio de crear usuarios o roles. Realmente el comando `create user` es un alias de `create role`, ya que al crear un usuario, estamos creando un rol.

- initiating replication: Se otorga a un rol el privilegio 

- password: Se otorga a un rol el privilegio de generar o crear contraseñas.

- inheritance of privileges: Se otorga el privilegio de otorgar privilegios.

- bypassing row-level security: Se otorga a un rol el privilegio de indicar que la contraseña de los usuarios sean seguras.

- connection limit: Se otorga a un rol el privilegio de limitar el numero de conexiones a la base de datos.

### EJERCICIO 2

Averigua cual es la forma de asignar y revocar privilegios sobre una tabla concreta en Postgres.

La forma de asignar privilegios en sobre una tabla en concreto tiene la siguiente sintaxis:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE table1, table2...
```

Para revocar los privilegios de una tabla:

```sql
REVOKE * ON TABLE table1, table2...
```

### EJERCICIO 3

Averigua si existe el concepto de rol en Postgres y señala las diferencias con los roles de ORACLE.

Sí, existe el concepto de "rol" en PostgreSQL. Los roles en PostgreSQL son similares a los roles en Oracle en el sentido de que ambos se utilizan para controlar el acceso a los objetos de la base de datos. Sin embargo, hay algunas diferencias clave entre los roles de PostgreSQL y los roles de Oracle.

1. En PostgreSQL, los roles son similares a los usuarios, ya que ambos tienen credenciales de inicio de sesión y contraseñas. Los roles también pueden ser miembros de otros roles, lo que permite la herencia de privilegios.

2. En Oracle, los roles se utilizan principalmente para otorgar privilegios a los usuarios. Los usuarios se asignan a roles y los roles se les otorgan los privilegios necesarios.

3. En PostgreSQL los roles pueden tener diferentes tipos de autenticación como pueden ser password, ident, trust entre otros, mientras que en Oracle solo pueden ser autenticados con contraseña.

4. En PostgreSQL, los roles pueden tener diferentes privilegios en diferentes bases de datos y esquemas, mientras que en Oracle los privilegios se asignan a nivel de usuario.

5. Los roles en PostgreSQL son independientes de los usuarios, es decir, un usuario puede ser miembro de varios roles, mientras que en Oracle un usuario solo puede tener un rol.

En resumen, los roles en PostgreSQL y Oracle son similares en cuanto a su función principal: controlar el acceso a los objetos de la base de datos. Sin embargo, hay algunas diferencias importantes en la forma en que se utilizan y se administran en cada sistema.

### EJERCICIO 4

Averigua si existe el concepto de perfil como conjunto de límites sobre el uso de recursos o sobre la contraseña en Postgres y señala las diferencias con los perfiles de ORACLE.

Sí, existe el concepto de perfil en PostgreSQL, pero funciona de manera diferente a como se utiliza en Oracle.

En Oracle, los perfiles son conjuntos de límites que se establecen para controlar el uso de recursos y las contraseñas de los usuarios. Los límites de recursos pueden incluir cosas como el tiempo de sesión máximo, el uso de CPU y el uso de memoria. Los límites de contraseña pueden incluir cosas como la longitud mínima de la contraseña, la cantidad de días antes de que expire una contraseña y el número de intentos fallidos permitidos antes de bloquear una cuenta.

En PostgreSQL, no existe un sistema de perfiles integrado para establecer límites de recursos y contraseñas. Sin embargo, se pueden establecer límites de recursos utilizando la configuración del sistema operativo o utilizando herramientas de terceros. Además, se pueden establecer límites de contraseñas utilizando las características de autenticación de PostgreSQL.

En resumen, en Oracle existe un sistema de perfiles integrado que se utiliza para establecer límites de recursos y contraseñas, mientras que en PostgreSQL no existe un sistema de perfiles integrado, pero se pueden establecer límites de recursos y contraseñas utilizando configuraciones del sistema operativo o herramientas de terceros.

### EJERCICIO 5

Realiza consultas al diccionario de datos de Postgres para averiguar todos los privilegios que tiene un usuario concreto.

```sql
SELECT grantee, privilege_type, table_name
FROM information_schema.role_table_grants
WHERE grantee = 'joseju';
```

### EJERCICIO 6

Realiza consultas al diccionario de datos en Postgres para averiguar qué usuarios pueden consultar una tabla concreta.

Para obtener una lista de usuarios y roles que tienen permisos de SELECT en una tabla específica, ejecutamos la siguiente consulta:

```sql
postgres=# SELECT grantee, privilege_type
FROM information_schema.table_privileges
WHERE table_name = 'pg_aggregate' AND privilege_type = 'SELECT';

 grantee  | privilege_type 
----------+----------------
 postgres | SELECT
 PUBLIC   | SELECT
(2 filas)
```

Debemos tener en cuenta que tenemos que reemplazar 'nombre_tabla' con el nombre de la tabla para la cual deseas obtener información de permisos.

Es importante mencionar que estas consultas estan basadas en la estructura del diccionario de datos de PostgreSQL, puede variar dependiendo de la version y configuracion de tu base de datos.

----

## ORACLE:

### EJERCICIO 7

Realiza una función de verificación de contraseñas que compruebe que la contraseña difiere en más de tres caracteres de la anterior y que la longitud de la misma es diferente de la anterior. Asígnala al perfil CONTRASEÑASEGURA. Comprueba que funciona correctamente.

```sql
CREATE OR REPLACE FUNCTION verificar_contrasenas (old_password VARCHAR2, new_password VARCHAR2) 
RETURN BOOLEAN 
IS
  different_characters INTEGER;
BEGIN
    different_characters := 0;
    FOR i IN 1..LENGTH(old_password) LOOP
        IF SUBSTR(old_password, i, 1) != SUBSTR(new_password, i, 1) THEN
            different_characters := different_characters + 1;
        END IF;
    END LOOP;
    IF different_characters > 3 AND LENGTH(old_password) != LENGTH(new_password) THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
END verificar_contrasenas;
/
```

Para asignar la función que acabamos de crear al perfil CONTRASEÑASEGURA, ejecutamos el siguiente comando:

```sql
ALTER PROFILE "CONTRASENASEGURA" LIMIT PASSWORD_VERIFY_FUNCTION "verificar_contrasenas";
```

Comprobamos que se ha asignado correctamente realizando la siguiente consulta:

```sql
SQL> SELECT profile, resource_name, limit 
FROM dba_profiles
WHERE profile = 'CONTRASENASEGURA' AND resource_name = 'PASSWORD_VERIFY_FUNCTION';

verificar_contrasenas
```

### EJERCICIO 8

Realiza un procedimiento llamado MostrarPrivilegiosdelRol que reciba el nombre de un rol y muestre los privilegios de sistema y los privilegios sobre objetos que lo componen.

```sql
create or replace procedure MostrarPrivilegiosdeRol(p_rol dba_roles.role%type)
is
begin
    dbms_output.put_line( 'Rol: ' || p_rol);
    MostrarPrivSistema(p_rol);
    MostrarPrivObjetos(p_rol);
end MostrarPrivilegiosdeRol;
/

create or replace procedure MostrarPrivSistema(p_rol dba_roles.role%type)
is
cursor c_sistema is select privilege from ROLE_SYS_PRIVS where role=p_rol;
begin
dbms_output.put_line( ' ');
dbms_output.put_line( 'Privilegios de sistema: ' || p_rol);
dbms_output.put_line( ' ');
for i in c_sistema loop
    dbms_output.put_line(i.privilege);
end loop;
end MostrarPrivSistema;
/

create or replace procedure MostrarPrivObjetos(p_rol dba_roles.role%type)
is
cursor c_objetos is select privilege, table_name from ROLE_TAB_PRIVS where role=p_rol;
begin
dbms_output.put_line( ' ');
dbms_output.put_line( 'Privilegios sobre objetos: ' || p_rol);
dbms_output.put_line( ' ');
for d in c_objetos loop
    dbms_output.put_line( 'Nombre objeto: ' || d.table_name || ' -------------- Privilegio: ' || d.privilege);
end loop;
end MostrarPrivObjetos;
/
```

### Prueba de funcionamiento

```sql
SQL> exec MostrarPrivilegiosdeRol('DBA');

Rol: DBA

Privilegios de sistema: DBA

CREATE ANY HIERARCHY
ALTER ANY ATTRIBUTE DIMENSION
CREATE ANY ATTRIBUTE DIMENSION
CREATE ATTRIBUTE DIMENSION
ALTER ANY MEASURE FOLDER
LOGMINING
CREATE ANY CREDENTIAL
CREATE CREDENTIAL
DROP LOCKDOWN PROFILE
CREATE LOCKDOWN PROFILE
CREATE ANY SQL TRANSLATION PROFILE
CREATE SQL TRANSLATION PROFILE
CREATE CUBE BUILD PROCESS
SELECT ANY CUBE
DROP ANY CUBE DIMENSION
SELECT ANY MINING MODEL
DROP ANY MINING MODEL
SELECT ANY TRANSACTION
ON COMMIT REFRESH
CREATE ANY OUTLINE
CREATE ANY CONTEXT
UNDER ANY VIEW
ALTER ANY MATERIALIZED VIEW
DROP PROFILE
SELECT ANY SEQUENCE
CREATE ANY SYNONYM
CREATE ANY CLUSTER
ALTER ROLLBACK SEGMENT
ALTER USER
ALTER TABLESPACE
CREATE TABLESPACE
ALTER SESSION
ALTER ANY CUBE BUILD PROCESS
SELECT ANY MEASURE FOLDER
CREATE PLUGGABLE DATABASE
FLASHBACK ARCHIVE ADMINISTER
UPDATE ANY CUBE BUILD PROCESS
DELETE ANY MEASURE FOLDER
CREATE MEASURE FOLDER
DELETE ANY CUBE DIMENSION
ADMINISTER ANY SQL TUNING SET
CREATE ANY JOB
ALTER ANY RULE SET
CREATE RULE SET
DROP ANY EVALUATION CONTEXT
ALTER ANY EVALUATION CONTEXT
CREATE EVALUATION CONTEXT
DEBUG CONNECT ANY
DROP ANY CONTEXT
CREATE LIBRARY
CREATE ANY MATERIALIZED VIEW
ALTER PROFILE
ALTER ANY TRIGGER
CREATE TRIGGER
CREATE PROCEDURE
FORCE TRANSACTION
DROP ANY VIEW
UPDATE ANY TABLE
LOCK ANY TABLE
DROP ANY TABLE
RESTRICTED SESSION
ALTER SYSTEM
ALTER ANY ANALYTIC VIEW
CREATE ANY ANALYTIC VIEW
CREATE HIERARCHY
USE ANY JOB RESOURCE
DROP ANY SQL TRANSLATION PROFILE
DROP ANY CUBE
CREATE CUBE
ADMINISTER SQL TUNING SET
MANAGE SCHEDULER
ANALYZE ANY DICTIONARY
ALTER ANY RULE
EXECUTE ANY EVALUATION CONTEXT
CREATE ANY EVALUATION CONTEXT
DROP ANY DIMENSION
CREATE DIMENSION
ALTER ANY OPERATOR
ALTER ANY LIBRARY
UNDER ANY TYPE
EXECUTE ANY TYPE
GRANT ANY PRIVILEGE
CREATE PROFILE
ALTER DATABASE
CREATE PUBLIC DATABASE LINK
CREATE ANY SEQUENCE
CREATE ANY VIEW
CREATE PUBLIC SYNONYM
CREATE ANY INDEX
CREATE CLUSTER
DELETE ANY TABLE
DROP ANY ANALYTIC VIEW
ALTER ANY HIERARCHY
CREATE ANY CUBE BUILD PROCESS
INSERT ANY MEASURE FOLDER
CREATE ANY CUBE
SELECT ANY CUBE DIMENSION
EXECUTE ASSEMBLY
EXECUTE ANY ASSEMBLY
CREATE ASSEMBLY
MANAGE ANY FILE GROUP
EXECUTE ANY CLASS
CREATE RULE
EXPORT FULL DATABASE
CREATE ANY RULE SET
DEBUG CONNECT SESSION
MERGE ANY VIEW
DROP ANY OUTLINE
MANAGE ANY QUEUE
CREATE ANY DIMENSION
QUERY REWRITE
DROP ANY DIRECTORY
DROP ANY TRIGGER
CREATE ANY TRIGGER
FORCE ANY TRANSACTION
CREATE DATABASE LINK
DROP ANY INDEX
ALTER ANY CLUSTER
COMMENT ANY TABLE
BACKUP ANY TABLE
CREATE TABLE
DROP ROLLBACK SEGMENT
DROP TABLESPACE
DROP ANY HIERARCHY
EM EXPRESS CONNECT
ALTER ANY SQL TRANSLATION PROFILE
ADMINISTER SQL MANAGEMENT OBJECT
CREATE ANY MEASURE FOLDER
INSERT ANY CUBE DIMENSION
CREATE CUBE DIMENSION
ALTER ANY ASSEMBLY
CREATE ANY EDITION
CREATE EXTERNAL JOB
READ ANY FILE GROUP
MANAGE FILE GROUP
CREATE ANY RULE
EXECUTE ANY RULE SET
FLASHBACK ANY TABLE
DEBUG ANY PROCEDURE
RESUMABLE
ADMINISTER RESOURCE MANAGER
ALTER ANY OUTLINE
UNDER ANY TABLE
CREATE OPERATOR
DROP ANY MATERIALIZED VIEW
CREATE MATERIALIZED VIEW
ANALYZE ANY
ALTER RESOURCE COST
DROP ANY ROLE
CREATE ROLE
DROP ANY SEQUENCE
DROP PUBLIC SYNONYM
CREATE SYNONYM
ALTER ANY INDEX
DROP ANY CLUSTER
REDEFINE ANY TABLE
SELECT ANY TABLE
CREATE ROLLBACK SEGMENT
CREATE USER
CREATE SESSION
CREATE ANALYTIC VIEW
DROP ANY ATTRIBUTE DIMENSION
SELECT ANY CUBE BUILD PROCESS
ALTER LOCKDOWN PROFILE
UPDATE ANY CUBE
ALTER ANY MINING MODEL
CREATE ANY MINING MODEL
CREATE MINING MODEL
ALTER ANY EDITION
EXECUTE ANY PROGRAM
DROP ANY RULE
DROP ANY RULE SET
GRANT ANY OBJECT PRIVILEGE
SELECT ANY DICTIONARY
ADMINISTER DATABASE TRIGGER
ALTER ANY DIMENSION
GLOBAL QUERY REWRITE
CREATE INDEXTYPE
EXECUTE ANY OPERATOR
CREATE TYPE
CREATE VIEW
DROP ANY SYNONYM
INSERT ANY TABLE
DROP USER
BECOME USER
READ ANY TABLE
DROP ANY MEASURE FOLDER
COMMENT ANY MINING MODEL
CREATE ANY ASSEMBLY
DROP ANY EDITION
CHANGE NOTIFICATION
CREATE ANY SQL PROFILE
ALTER ANY SQL PROFILE
DROP ANY SQL PROFILE
CREATE JOB
EXECUTE ANY RULE
IMPORT FULL DATABASE
EXECUTE ANY INDEXTYPE
DROP ANY INDEXTYPE
CREATE ANY INDEXTYPE
DROP ANY OPERATOR
CREATE ANY OPERATOR
EXECUTE ANY LIBRARY
DROP ANY TYPE
ALTER ANY TYPE
AUDIT ANY
DROP PUBLIC DATABASE LINK
ALTER ANY SEQUENCE
ALTER ANY TABLE
AUDIT SYSTEM
SET CONTAINER
USE ANY SQL TRANSLATION PROFILE
UPDATE ANY CUBE DIMENSION
DROP ANY CUBE BUILD PROCESS
ALTER ANY CUBE
CREATE ANY CUBE DIMENSION
ALTER ANY CUBE DIMENSION
DROP ANY ASSEMBLY
ADVISOR
DEQUEUE ANY QUEUE
ENQUEUE ANY QUEUE
ALTER ANY INDEXTYPE
DROP ANY LIBRARY
CREATE ANY LIBRARY
CREATE ANY TYPE
CREATE ANY DIRECTORY
EXECUTE ANY PROCEDURE
DROP ANY PROCEDURE
ALTER ANY PROCEDURE
CREATE ANY PROCEDURE
ALTER ANY ROLE
GRANT ANY ROLE
CREATE SEQUENCE
CREATE ANY TABLE
MANAGE TABLESPACE

Privilegios sobre objetos: DBA

Nombre objeto: ADR_LOG_MSG_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_ADR_CONTROL -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_PACKAGE_FILE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_REMOTE_PACKAGE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PACKAGE_HISTORY -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM -------------- Privilegio: SELECT
Nombre objeto: CDB_XS_SESSION_NS_ATTRIBUTES -------------- Privilegio: SELECT
Nombre objeto: DBMS_FLASHBACK_ARCHIVE -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_MONITOR -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_WORKLOAD_REPOSITORY -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_ADR -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_APP_CONT -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_APP_CONT_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: XDB$D_LINK -------------- Privilegio: DELETE
Nombre objeto: XDB$ACL -------------- Privilegio: INSERT
Nombre objeto: XDB$RESCONFIG -------------- Privilegio: INSERT
Nombre objeto: XDB$CONFIG -------------- Privilegio: DELETE
Nombre objeto: DBA_REGISTRY_SQLPATCH_RU_INFO -------------- Privilegio: SELECT
Nombre objeto: OJDS_CONTEXT -------------- Privilegio: EXECUTE
Nombre objeto: XSDB$SCHEMA_ACL -------------- Privilegio: SELECT
Nombre objeto: ADR_INCIDENT_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_INCCKEY -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIEWCOL -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_HM_MESSAGE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_PACKAGE_INCIDENT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_FILE_COPY_LOG -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_PACKAGE_HISTORY -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_ALERT_EXT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_RELMD_EXT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DFW_PATCH_CAPTURE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VINCIDENT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_FILE_COPY_LOG -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PACKAGE_SIZE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_INCCOUNT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM_LASTINC -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_INC_METER_INFO_PROB -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPDB_PROBLEM -------------- Privilegio: SELECT
Nombre objeto: DBA_XS_SESSIONS -------------- Privilegio: SELECT
Nombre objeto: DBA_XS_ACTIVE_SESSIONS -------------- Privilegio: SELECT
Nombre objeto: DBA_XS_SESSION_NS_ATTRIBUTES -------------- Privilegio: SELECT
Nombre objeto: DBMS_TDB -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_SERVER_ALERT -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_HM -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_AWR_WAREHOUSE_SERVER -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_UMF -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_GSM_FIX -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_INMEMORY_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_NETWORK_ACL_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: XDB$H_INDEX -------------- Privilegio: INSERT
Nombre objeto: XDB$H_INDEX -------------- Privilegio: UPDATE
Nombre objeto: XDB$D_LINK -------------- Privilegio: INSERT
Nombre objeto: XDB$D_LINK -------------- Privilegio: SELECT
Nombre objeto: XDB$RESOURCE -------------- Privilegio: DELETE
Nombre objeto: X$PT46MP5MDR0M04NE0KWN0SK0K1LN -------------- Privilegio: INSERT
Nombre objeto: DBA_REGISTRY_SQLPATCH -------------- Privilegio: DELETE
Nombre objeto: DBA_REGISTRY_SQLPATCH -------------- Privilegio: UPDATE
Nombre objeto: DBMS_XDBT -------------- Privilegio: EXECUTE
Nombre objeto: XSDB$SCHEMA_ACL -------------- Privilegio: UPDATE
Nombre objeto: ADR_INCIDENT_INFO_T -------------- Privilegio: EXECUTE
Nombre objeto: ADR_LOG_MSG_SUPPL_ATTR_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_PICKLEERR -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DDE_USR_ACT_PARAM -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DDE_USR_INC_ACT_MAP -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_CONFIGURATION -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_INC_METER_IMPT_DEF -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_EM_USER_ACTIVITY -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PACKAGE_FILE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM1 -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM_BUCKET -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VADR_CONTROL -------------- Privilegio: SELECT
Nombre objeto: CDB_XS_SESSION_ROLES -------------- Privilegio: SELECT
Nombre objeto: DBMS_MANAGEMENT_BOOTSTRAP -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_AWR_WAREHOUSE_SOURCE -------------- Privilegio: EXECUTE
Nombre objeto: OUTLN_PKG -------------- Privilegio: EXECUTE
Nombre objeto: XDB$H_LINK -------------- Privilegio: DELETE
Nombre objeto: DBMS_XDB_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: X$PT46MP5MDR0M04NE0KWN0SK0K1LN -------------- Privilegio: DELETE
Nombre objeto: X$PT46MP5MDR0M04NE0KWN0SK0K1LN -------------- Privilegio: UPDATE
Nombre objeto: XDB$ACL -------------- Privilegio: SELECT
Nombre objeto: XDB$CHECKOUTS -------------- Privilegio: SELECT
Nombre objeto: DBA_REGISTRY_SQLPATCH_RU_INFO -------------- Privilegio: INSERT
Nombre objeto: GRANT_RDF_OWNER_DR_PRIVS -------------- Privilegio: EXECUTE
Nombre objeto: MAP_OBJECT -------------- Privilegio: DELETE
Nombre objeto: ADR_INCIDENT_FILES_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_VIEW -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DDE_USER_ACTION_DEF -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_PKG_UNPACK_HIST -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_EM_DIAG_JOB -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VSHOWINCB_I -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_INCFCOUNT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_NFCINC -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_IPSPRBCNT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DIAGV_INCIDENT -------------- Privilegio: SELECT
Nombre objeto: CDB_XS_SESSIONS -------------- Privilegio: SELECT
Nombre objeto: XS_MTCACHE_INT -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_SERVICE -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_IR -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_FEATURE_USAGE_REPORT -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_UNDO_ADV -------------- Privilegio: EXECUTE
Nombre objeto: LOAD_UNDO_STAT -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_CACHEUTIL -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_DNFS -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_FS -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_XS_PRINCIPALS -------------- Privilegio: EXECUTE
Nombre objeto: XDB$H_INDEX -------------- Privilegio: SELECT
Nombre objeto: XDB$H_LINK -------------- Privilegio: SELECT
Nombre objeto: XDB$RESOURCE -------------- Privilegio: SELECT
Nombre objeto: DBMS_CSX_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: X$PT46MP5MDR0M04NE0KWN0SK0K1LN -------------- Privilegio: SELECT
Nombre objeto: XDB$ACL -------------- Privilegio: DELETE
Nombre objeto: XDB$CONFIG -------------- Privilegio: SELECT
Nombre objeto: DBA_REGISTRY_SQLPATCH -------------- Privilegio: INSERT
Nombre objeto: DBA_REGISTRY_SQLPATCH -------------- Privilegio: SELECT
Nombre objeto: DBA_REGISTRY_SQLPATCH_RU_INFO -------------- Privilegio: UPDATE
Nombre objeto: RDF_NETWORK_CREATOR_PRIVS -------------- Privilegio: EXECUTE
Nombre objeto: MAP_OBJECT -------------- Privilegio: UPDATE
Nombre objeto: ADR_LOG_MSG_SUPPL_ATTRS_T -------------- Privilegio: EXECUTE
Nombre objeto: ADR_LOG_MSG_ECID_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_INCIDENT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_INCIDENT_FILE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_HM_RUN -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_HM_FINDING -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DDE_USER_ACTION -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_PACKAGE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_PROGRESS_LOG -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_INC_METER_PK_IMPTS -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DFW_CONFIG_ITEM -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DFW_PURGE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_ADR_CONTROL_AUX -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PKG_INC_DTL1 -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VEM_USER_ACTLOG -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM2 -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM_BUCKET_COUNT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PKG_MAIN_PROBLEM -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_ACTPROB -------------- Privilegio: SELECT
Nombre objeto: DBMS_FLASHBACK -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_WORKLOAD_REPLAY -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_ROLLING -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_TNS -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_AUTO_TASK_IMMEDIATE -------------- Privilegio: EXECUTE
Nombre objeto: AS_REPLAY -------------- Privilegio: EXECUTE
Nombre objeto: XDB$H_INDEX -------------- Privilegio: DELETE
Nombre objeto: XDB$D_LINK -------------- Privilegio: UPDATE
Nombre objeto: XDB$RESCONFIG -------------- Privilegio: DELETE
Nombre objeto: MAP_OBJECT -------------- Privilegio: SELECT
Nombre objeto: DBMS_LOGSTDBY -------------- Privilegio: EXECUTE
Nombre objeto: ADR_LOG_MSG_ERRID_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_SWEEPERR -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_HM_RECOMMENDATION -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_HM_INFO -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DDE_USR_INC_TYPE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_IPS_FILE_METADATA -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DFW_CONFIG_CAPTURE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PKG_FILE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VINCIDENT_FILE -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM_INT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VEM_USER_ACTLOG1 -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VPROBLEM_BUCKET1 -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VHM_RUN -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_SWPERRCOUNT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VNOT_EXIST_INCIDENT -------------- Privilegio: SELECT
Nombre objeto: DBMS_AUTO_SQLTUNE -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_RESULT_CACHE -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_WORKLOAD_CAPTURE -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_RAT_MASK -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_SERVICE_CONST -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_SERVICE_ERR -------------- Privilegio: EXECUTE
Nombre objeto: XDB$NLOCKS -------------- Privilegio: DELETE
Nombre objeto: XDB$NLOCKS -------------- Privilegio: INSERT
Nombre objeto: DBA_REGISTRY_SQLPATCH_RU_INFO -------------- Privilegio: DELETE
Nombre objeto: ADR_HOME_T -------------- Privilegio: EXECUTE
Nombre objeto: ADR_INCIDENT_CORR_KEYS_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_PROBLEM -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_HM_FDG_SET -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_INC_METER_SUMMARY -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_INC_METER_CONFIG -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_EM_TARGET_INFO -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_AMS_XACTION -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DFW_PATCH_ITEM -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_PDB_PROBLEM -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_PDB_SPACE_MGMT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VSHOWINCB -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VSHOWCATVIEW -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PKG_INC_DTL -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PACKAGE_MAIN_INT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_ACTINC -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_PKG_INC_CAND -------------- Privilegio: SELECT
Nombre objeto: CDB_XS_ACTIVE_SESSIONS -------------- Privilegio: SELECT
Nombre objeto: DBMS_STORAGE_MAP -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_SERVER_TRACE -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_PERF -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_WRR_STATE -------------- Privilegio: EXECUTE
Nombre objeto: RESET_UNDO_STAT -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_ILM_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_MEMOPTIMIZE_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_AUTO_INDEX -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_AUTO_TASK_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: HM_SQLTK_INTERNAL -------------- Privilegio: EXECUTE
Nombre objeto: XDB$NLOCKS -------------- Privilegio: SELECT
Nombre objeto: XDB$ACL -------------- Privilegio: UPDATE
Nombre objeto: XDB$RESCONFIG -------------- Privilegio: SELECT
Nombre objeto: XDB$RESCONFIG -------------- Privilegio: UPDATE
Nombre objeto: XDB$CHECKOUTS -------------- Privilegio: DELETE
Nombre objeto: XDB$CHECKOUTS -------------- Privilegio: INSERT
Nombre objeto: XDB$CONFIG -------------- Privilegio: INSERT
Nombre objeto: XDB$CONFIG -------------- Privilegio: UPDATE
Nombre objeto: ORD_ADMIN -------------- Privilegio: EXECUTE
Nombre objeto: CARTRIDGE -------------- Privilegio: EXECUTE
Nombre objeto: MAP_OBJECT -------------- Privilegio: INSERT
Nombre objeto: XSDB$SCHEMA_ACL -------------- Privilegio: INSERT
Nombre objeto: ADR_LOG_MSG_ARG_T -------------- Privilegio: EXECUTE
Nombre objeto: ADR_LOG_MSG_ARGS_T -------------- Privilegio: EXECUTE
Nombre objeto: V_$DIAG_ADR_INVALIDATION -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DDE_USR_ACT_PARAM_DEF -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_INC_METER_INFO -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DIR_EXT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_LOG_EXT -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_DFW_PURGE_ITEM -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VINC_METER_INFO -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VIPS_FILE_METADATA -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_V_IPSPRBCNT1 -------------- Privilegio: SELECT
Nombre objeto: V_$DIAG_VTEST_EXISTS -------------- Privilegio: SELECT
Nombre objeto: DBA_XS_SESSION_ROLES -------------- Privilegio: SELECT
Nombre objeto: DBMS_RESUMABLE -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_SERVICE_PRVT -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_UADV_ARR -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_ADR_APP -------------- Privilegio: EXECUTE
Nombre objeto: DBMS_HANG_MANAGER -------------- Privilegio: EXECUTE
Nombre objeto: XDB$H_LINK -------------- Privilegio: INSERT
Nombre objeto: XDB$H_LINK -------------- Privilegio: UPDATE
Nombre objeto: XDB$RESOURCE -------------- Privilegio: INSERT
Nombre objeto: XDB$RESOURCE -------------- Privilegio: UPDATE
Nombre objeto: XDB$NLOCKS -------------- Privilegio: UPDATE
Nombre objeto: XDB$CHECKOUTS -------------- Privilegio: UPDATE

PL/SQL procedure successfully completed.
```