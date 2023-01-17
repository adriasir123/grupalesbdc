# Ejercicio 4

## 4. (ORACLE) Realiza un procedimiento que genere un script que cree un rol conteniendo todos los permisos que tenga el usuario cuyo nombre reciba como parámetro, le hayan sido asignados a aquél directamente o a traves de roles. El nuevo rol deberá llamarse BackupPrivsNombreUsuario

Este sería el procedimiento principal, al cual le daríamos un usuario y, si existe, nos mostrará por pantalla la creación del rol y la asignación de todos los privilegios al rol creado.

```sql
create or replace procedure CrearRol (p_usuario varchar2)
is
    v_validacion number:=0;
    v_nuevoRol varchar(50):='BackupPrivs'||p_usuario;
begin
    v_validacion:=ComprobarUsuario(p_usuario);
    if v_validacion=0 then
        dbms_output.put_line('CREATE ROLE BackupPrivs'||p_usuario);
        SysPrivs(p_usuario, v_nuevoRol);
        PrivsObjetos(p_usuario, v_nuevoRol);
    end if;
end CrearRol;
/
```

Procedimientos que buscan los privilegios por objeto y del sistema del usuario especificado en el procedimiento principal.

PrivilegiosSobreObjetos asigna los privilegios sobre objetos a través de un cursor que recupera los privilegios de DBA_TAB_PRIVS para el usuario especificado o para cualquier rol asignado al usuario, y PrivilegiosDelSistema asigna los privilegios del sistema de la misma manera a través de otro cursor que recupera los privilegios de DBA_SYS_PRIVS.

```sql
create or replace procedure PrivsObjetos(p_usuario varchar2, p_nuevoRol varchar2)
is
    cursor c_tabprivs is
        select distinct PRIVILEGE, OWNER, TABLE_NAME from DBA_TAB_PRIVS where GRANTEE=p_usuario or GRANTEE in
            (select distinct granted_role from DBA_ROLE_PRIVS start with GRANTEE=p_usuario connect by GRANTEE=prior GRANTED_ROLE);
    v_cursor c_tabprivs%ROWTYPE;
begin
    for v_cursor in c_tabprivs loop
        AgregarPrivsObjeto(v_cursor.PRIVILEGE, v_cursor.OWNER, v_cursor.TABLE_NAME, p_nuevoRol);
    end loop;
end PrivsObjetos;
/

create or replace procedure SysPrivs(p_usuario varchar2, p_nuevoRol varchar2)
is
    cursor c_sysprivs is
        select distinct PRIVILEGE, ADMIN_OPTION from DBA_SYS_PRIVS where GRANTEE=p_usuario or GRANTEE in
            (select distinct granted_role from DBA_ROLE_PRIVS start with GRANTEE = p_usuario connect by GRANTEE = prior GRANTED_ROLE);
    v_cursor c_sysprivs%ROWTYPE;
begin
    for v_cursor in c_sysprivs loop
        AgregarSysPrivs(v_cursor.PRIVILEGE, v_cursor.ADMIN_OPTION, p_nuevoRol);
    end loop;
end SysPrivs;
/
```

Los procedimientos AgregarPrivilegioSobreObjeto y AgregarPrivilegioDelSistema, con lo datos obtenido con los cursores de los procedimientos anteriores, devuelven el código concreto del script que se mostrará por pantalla al ejecutar el procedimiento principal.

```sql
create or replace procedure AgregarPrivsObjeto(p_privilegio DBA_TAB_PRIVS.PRIVILEGE%TYPE, p_propietario DBA_TAB_PRIVS.OWNER%TYPE, p_nombreTabla DBA_TAB_PRIVS.TABLE_NAME%TYPE, p_nuevoRol varchar2)
is
begin
    dbms_output.put_line('grant '||p_privilegio||' on '||p_propietario||'.'||p_nombreTabla||' to '||p_nuevoRol||';');
end AgregarPrivsObjeto;
/

create or replace procedure AgregarSysPrivs(p_privilegio USER_SYS_PRIVS.PRIVILEGE%TYPE, p_opcAdmin USER_SYS_PRIVS.ADMIN_OPTION%TYPE, p_nuevoRol varchar2)
is
begin
    if p_opcAdmin='YES' then 
        dbms_output.put_line('grant '||p_privilegio||' to '||p_nuevoRol||' with admin option;');
    else dbms_output.put_line('grant '||p_privilegio||' to '||p_nuevoRol||';');
    end if;
end AgregarSysPrivs;
/
```

La función ComprobarUsuario verifica si el usuario existe en la base de datos antes de asignarle los privilegios.

```sql
create or replace function ComprobarUsuario(p_usuario varchar2)
return number
is
    v_resultado varchar2(30);
begin
    select USERNAME into v_resultado
    from DBA_USERS
    where USERNAME=p_usuario;
    return 0;
exception
    when NO_DATA_FOUND then
        dbms_output.put_line('No existe el usuario '||p_usuario);
    return 1;
end ComprobarUsuario;
/
```

### Comprobación

En la captura de pantalla vemos que si usamos el procedimiento CrearRol con un usuario que no existe nos aparece el mensaje "No existe el usuario usuario". En cambio si lo usamos con el usuario ADMIN nos mostrará todos los grant que deberemos aplicar al nuevo rol creado, que en este caso serán todos, ya que el usuario admin tiene todos los privilegios otorgados.

![prueba](/img/capturas-arantxa/89.png)

Si creo un usuario nuevo y usamos el procedimiento sobre este solo creara el rol pero como no le he otorgado ningún privilegio no aparecerá nada más.

![prueba](/img/capturas-arantxa/90.png)

Dejo aquí el código devuelto para que se vea más claro:

```sql
exec CrearRol('usuario');
No existe el usuario usuario

Procedimiento PL/SQL terminado correctamente.

exec CrearRol('ADMIN');
CREATE ROLE BackupPrivsADMIN
grant CREATE ANY TABLE to BackupPrivsADMIN;
grant ALTER DATABASE to BackupPrivsADMIN;
grant DROP ANY OUTLINE to BackupPrivsADMIN;
grant ALTER ANY HIERARCHY to BackupPrivsADMIN;
grant ALTER ANY INDEX to BackupPrivsADMIN;
grant DROP ANY RULE to BackupPrivsADMIN;
grant CREATE MINING MODEL to BackupPrivsADMIN;
grant RESTRICTED SESSION to BackupPrivsADMIN;
grant SELECT ANY TABLE to BackupPrivsADMIN;
grant GRANT ANY ROLE to BackupPrivsADMIN;
grant ALTER ANY ROLE to BackupPrivsADMIN;
grant DROP ANY OPERATOR to BackupPrivsADMIN;
grant EXECUTE ANY PROGRAM to BackupPrivsADMIN;
grant SELECT ANY CUBE BUILD PROCESS to BackupPrivsADMIN;
grant ALTER ANY RULE to BackupPrivsADMIN;
grant DROP ANY ASSEMBLY to BackupPrivsADMIN;
grant ALTER ANY CUBE BUILD PROCESS to BackupPrivsADMIN;
grant DROP ANY HIERARCHY to BackupPrivsADMIN;
grant ALTER ANY DIMENSION to BackupPrivsADMIN;
grant CREATE ANY SQL PROFILE to BackupPrivsADMIN;
grant SELECT ANY CUBE to BackupPrivsADMIN;
grant CREATE CUBE BUILD PROCESS to BackupPrivsADMIN;
grant CREATE ANY CUBE BUILD PROCESS to BackupPrivsADMIN;
grant LOCK ANY TABLE to BackupPrivsADMIN;
grant ALTER ANY INDEXTYPE to BackupPrivsADMIN;
grant CREATE ANY OUTLINE to BackupPrivsADMIN;
grant EXECUTE ANY RULE SET to BackupPrivsADMIN;
grant DROP PUBLIC SYNONYM to BackupPrivsADMIN;
grant CREATE ANY VIEW to BackupPrivsADMIN;
grant MANAGE ANY FILE GROUP to BackupPrivsADMIN;
grant MERGE ANY VIEW to BackupPrivsADMIN;
grant SELECT ANY TRANSACTION to BackupPrivsADMIN;
grant ALTER RESOURCE COST to BackupPrivsADMIN;
grant GLOBAL QUERY REWRITE to BackupPrivsADMIN;
grant CREATE ASSEMBLY to BackupPrivsADMIN;
grant ALTER USER to BackupPrivsADMIN;
grant CREATE SYNONYM to BackupPrivsADMIN;
grant DROP ANY TRIGGER to BackupPrivsADMIN;
grant ALTER LOCKDOWN PROFILE to BackupPrivsADMIN;
grant CREATE ATTRIBUTE DIMENSION to BackupPrivsADMIN;
grant DROP ANY TABLE to BackupPrivsADMIN;
grant AUDIT ANY to BackupPrivsADMIN;
grant EXECUTE ANY OPERATOR to BackupPrivsADMIN;
grant CHANGE NOTIFICATION to BackupPrivsADMIN;
grant CREATE CUBE DIMENSION to BackupPrivsADMIN;
grant DELETE ANY CUBE DIMENSION to BackupPrivsADMIN;
grant ADMINISTER RESOURCE MANAGER to BackupPrivsADMIN;
grant EXECUTE ANY ASSEMBLY to BackupPrivsADMIN;
grant CREATE ANALYTIC VIEW to BackupPrivsADMIN;
grant DELETE ANY TABLE to BackupPrivsADMIN;
grant CREATE TRIGGER to BackupPrivsADMIN;
grant SELECT ANY SEQUENCE to BackupPrivsADMIN;
grant CREATE PROCEDURE to BackupPrivsADMIN;
grant ALTER ANY ATTRIBUTE DIMENSION to BackupPrivsADMIN;
grant ALTER ANY OUTLINE to BackupPrivsADMIN;
grant CREATE ANY EDITION to BackupPrivsADMIN;
grant INSERT ANY CUBE DIMENSION to BackupPrivsADMIN;
grant CREATE ANY MEASURE FOLDER to BackupPrivsADMIN;
grant UPDATE ANY CUBE BUILD PROCESS to BackupPrivsADMIN;
grant ADMINISTER SQL MANAGEMENT OBJECT to BackupPrivsADMIN;
grant ALTER ANY MEASURE FOLDER to BackupPrivsADMIN;
grant CREATE ANY SYNONYM to BackupPrivsADMIN;
grant GRANT ANY PRIVILEGE to BackupPrivsADMIN;
grant DROP ANY RULE SET to BackupPrivsADMIN;
grant CREATE CUBE to BackupPrivsADMIN;
grant DROP ANY SQL TRANSLATION PROFILE to BackupPrivsADMIN;
grant DROP ANY EVALUATION CONTEXT to BackupPrivsADMIN;
grant CREATE ANY RULE SET to BackupPrivsADMIN;
grant COMMENT ANY MINING MODEL to BackupPrivsADMIN;
grant CREATE ANY SEQUENCE to BackupPrivsADMIN;
grant CREATE EVALUATION CONTEXT to BackupPrivsADMIN;
grant CREATE LIBRARY to BackupPrivsADMIN;
grant ALTER ANY RULE SET to BackupPrivsADMIN;
grant DROP ANY MINING MODEL to BackupPrivsADMIN;
grant DROP ANY CUBE to BackupPrivsADMIN;
grant READ ANY TABLE to BackupPrivsADMIN;
grant CREATE TABLESPACE to BackupPrivsADMIN;
grant MANAGE TABLESPACE to BackupPrivsADMIN;
grant CREATE ROLE to BackupPrivsADMIN;
grant ANALYZE ANY to BackupPrivsADMIN;
grant EXECUTE ANY TYPE to BackupPrivsADMIN;
grant EXECUTE ANY LIBRARY to BackupPrivsADMIN;
grant CREATE OPERATOR to BackupPrivsADMIN;
grant USE ANY JOB RESOURCE to BackupPrivsADMIN;
grant CREATE PUBLIC SYNONYM to BackupPrivsADMIN;
grant ALTER ANY MINING MODEL to BackupPrivsADMIN;
grant SELECT ANY CUBE DIMENSION to BackupPrivsADMIN;
grant CREATE ANY ATTRIBUTE DIMENSION to BackupPrivsADMIN;
grant CREATE SEQUENCE to BackupPrivsADMIN;
grant DROP PROFILE to BackupPrivsADMIN;
grant CREATE JOB to BackupPrivsADMIN;
grant CREATE ANY CUBE to BackupPrivsADMIN;
grant CREATE MEASURE FOLDER to BackupPrivsADMIN;
grant DROP ANY ATTRIBUTE DIMENSION to BackupPrivsADMIN;
grant DROP ANY INDEXTYPE to BackupPrivsADMIN;
grant QUERY REWRITE to BackupPrivsADMIN;
grant MANAGE SCHEDULER to BackupPrivsADMIN;
grant SELECT ANY MINING MODEL to BackupPrivsADMIN;
grant DROP ANY CUBE DIMENSION to BackupPrivsADMIN;
grant ALTER ROLLBACK SEGMENT to BackupPrivsADMIN;
grant EXECUTE ANY EVALUATION CONTEXT to BackupPrivsADMIN;
grant DROP ANY SEQUENCE to BackupPrivsADMIN;
grant UNDER ANY VIEW to BackupPrivsADMIN;
grant EXECUTE ANY RULE to BackupPrivsADMIN;
grant ADVISOR to BackupPrivsADMIN;
grant LOGMINING to BackupPrivsADMIN;
grant DROP USER to BackupPrivsADMIN;
grant RESUMABLE to BackupPrivsADMIN;
grant ALTER ANY SQL PROFILE to BackupPrivsADMIN;
grant SELECT ANY MEASURE FOLDER to BackupPrivsADMIN;
grant CREATE ANY HIERARCHY to BackupPrivsADMIN;
grant UNLIMITED TABLESPACE to BackupPrivsADMIN;
grant EXECUTE ANY PROCEDURE to BackupPrivsADMIN;
grant CREATE ANY INDEX to BackupPrivsADMIN;
grant DEBUG ANY PROCEDURE to BackupPrivsADMIN;
grant CREATE ANY EVALUATION CONTEXT to BackupPrivsADMIN;
grant ALTER SYSTEM to BackupPrivsADMIN;
grant CREATE ANY TRIGGER to BackupPrivsADMIN;
grant DROP ANY DIMENSION to BackupPrivsADMIN;
grant CREATE RULE to BackupPrivsADMIN;
grant BACKUP ANY TABLE to BackupPrivsADMIN;
grant GRANT ANY OBJECT PRIVILEGE to BackupPrivsADMIN;
grant UPDATE ANY CUBE DIMENSION to BackupPrivsADMIN;
grant USE ANY SQL TRANSLATION PROFILE to BackupPrivsADMIN;
grant ALTER TABLESPACE to BackupPrivsADMIN;
grant CREATE MATERIALIZED VIEW to BackupPrivsADMIN;
grant DROP ANY ANALYTIC VIEW to BackupPrivsADMIN;
grant UPDATE ANY TABLE to BackupPrivsADMIN;
grant CREATE ANY LIBRARY to BackupPrivsADMIN;
grant ADMINISTER SQL TUNING SET to BackupPrivsADMIN;
grant ALTER ANY ASSEMBLY to BackupPrivsADMIN;
grant CREATE CLUSTER to BackupPrivsADMIN;
grant CREATE ANY MATERIALIZED VIEW to BackupPrivsADMIN;
grant DROP ANY MEASURE FOLDER to BackupPrivsADMIN;
grant CREATE ANY CREDENTIAL to BackupPrivsADMIN;
grant CREATE HIERARCHY to BackupPrivsADMIN;
grant ALTER SESSION to BackupPrivsADMIN;
grant DROP TABLESPACE to BackupPrivsADMIN;
grant CREATE VIEW to BackupPrivsADMIN;
grant DEBUG CONNECT SESSION to BackupPrivsADMIN;
grant CREATE ANY JOB to BackupPrivsADMIN;
grant ADMINISTER ANY SQL TUNING SET to BackupPrivsADMIN;
grant CREATE PROFILE to BackupPrivsADMIN;
grant CREATE SESSION to BackupPrivsADMIN;
grant REDEFINE ANY TABLE to BackupPrivsADMIN;
grant CREATE ANY INDEXTYPE to BackupPrivsADMIN;
grant DROP ANY INDEX to BackupPrivsADMIN;
grant EXECUTE ANY INDEXTYPE to BackupPrivsADMIN;
grant ALTER ANY EDITION to BackupPrivsADMIN;
grant CREATE ANY ASSEMBLY to BackupPrivsADMIN;
grant CREATE LOCKDOWN PROFILE to BackupPrivsADMIN;
grant CREATE DIMENSION to BackupPrivsADMIN;
grant ON COMMIT REFRESH to BackupPrivsADMIN;
grant READ ANY FILE GROUP to BackupPrivsADMIN;
grant ALTER ANY TRIGGER to BackupPrivsADMIN;
grant IMPORT FULL DATABASE to BackupPrivsADMIN;
grant ALTER ANY CLUSTER to BackupPrivsADMIN;
grant ALTER ANY SEQUENCE to BackupPrivsADMIN;
grant FORCE TRANSACTION to BackupPrivsADMIN;
grant ALTER ANY EVALUATION CONTEXT to BackupPrivsADMIN;
grant CREATE ANY MINING MODEL to BackupPrivsADMIN;
grant CREATE ANY CUBE DIMENSION to BackupPrivsADMIN;
grant AUDIT SYSTEM to BackupPrivsADMIN;
grant ALTER ANY TABLE to BackupPrivsADMIN;
grant COMMENT ANY TABLE to BackupPrivsADMIN;
grant FORCE ANY TRANSACTION to BackupPrivsADMIN;
grant CREATE ANY DIMENSION to BackupPrivsADMIN;
grant DROP ANY ROLE to BackupPrivsADMIN;
grant CREATE DATABASE LINK to BackupPrivsADMIN;
grant CREATE RULE SET to BackupPrivsADMIN;
grant CREATE ANY ANALYTIC VIEW to BackupPrivsADMIN;
grant DROP ANY SQL PROFILE to BackupPrivsADMIN;
grant ALTER ANY CUBE DIMENSION to BackupPrivsADMIN;
grant DROP ANY TYPE to BackupPrivsADMIN;
grant ALTER ANY MATERIALIZED VIEW to BackupPrivsADMIN;
grant CREATE ANY DIRECTORY to BackupPrivsADMIN;
grant ALTER ANY LIBRARY to BackupPrivsADMIN;
grant ENQUEUE ANY QUEUE to BackupPrivsADMIN;
grant CREATE ANY RULE to BackupPrivsADMIN;
grant CREATE SQL TRANSLATION PROFILE to BackupPrivsADMIN;
grant DROP ANY MATERIALIZED VIEW to BackupPrivsADMIN;
grant DROP ANY DIRECTORY to BackupPrivsADMIN;
grant CREATE ANY TYPE to BackupPrivsADMIN;
grant CREATE INDEXTYPE to BackupPrivsADMIN;
grant UNDER ANY TABLE to BackupPrivsADMIN;
grant FLASHBACK ANY TABLE to BackupPrivsADMIN;
grant DELETE ANY MEASURE FOLDER to BackupPrivsADMIN;
grant ALTER ANY ANALYTIC VIEW to BackupPrivsADMIN;
grant CREATE PUBLIC DATABASE LINK to BackupPrivsADMIN;
grant DROP PUBLIC DATABASE LINK to BackupPrivsADMIN;
grant EXPORT FULL DATABASE to BackupPrivsADMIN;
grant INSERT ANY TABLE to BackupPrivsADMIN;
grant CREATE EXTERNAL JOB to BackupPrivsADMIN;
grant EXECUTE ASSEMBLY to BackupPrivsADMIN;
grant CREATE ANY SQL TRANSLATION PROFILE to BackupPrivsADMIN;
grant SET CONTAINER to BackupPrivsADMIN;
grant CREATE ANY CLUSTER to BackupPrivsADMIN;
grant ALTER ANY CUBE to BackupPrivsADMIN;
grant FLASHBACK ARCHIVE ADMINISTER to BackupPrivsADMIN;
grant DROP LOCKDOWN PROFILE to BackupPrivsADMIN;
grant ALTER ANY PROCEDURE to BackupPrivsADMIN;
grant DROP ANY LIBRARY to BackupPrivsADMIN;
grant DEBUG CONNECT ANY to BackupPrivsADMIN;
grant DROP ANY EDITION to BackupPrivsADMIN;
grant CREATE ROLLBACK SEGMENT to BackupPrivsADMIN;
grant CREATE TABLE to BackupPrivsADMIN;
grant DROP ANY SYNONYM to BackupPrivsADMIN;
grant CREATE ANY PROCEDURE to BackupPrivsADMIN;
grant ALTER PROFILE to BackupPrivsADMIN;
grant DROP ANY CONTEXT to BackupPrivsADMIN;
grant CREATE PLUGGABLE DATABASE to BackupPrivsADMIN;
grant CREATE CREDENTIAL to BackupPrivsADMIN;
grant UNDER ANY TYPE to BackupPrivsADMIN;
grant ADMINISTER DATABASE TRIGGER to BackupPrivsADMIN;
grant CREATE USER to BackupPrivsADMIN;
grant CREATE TYPE to BackupPrivsADMIN;
grant DEQUEUE ANY QUEUE to BackupPrivsADMIN;
grant ALTER ANY SQL TRANSLATION PROFILE to BackupPrivsADMIN;
grant DROP ANY CLUSTER to BackupPrivsADMIN;
grant ALTER ANY OPERATOR to BackupPrivsADMIN;
grant CREATE ANY CONTEXT to BackupPrivsADMIN;
grant EXECUTE ANY CLASS to BackupPrivsADMIN;
grant EM EXPRESS CONNECT to BackupPrivsADMIN;
grant DROP ROLLBACK SEGMENT to BackupPrivsADMIN;
grant MANAGE ANY QUEUE to BackupPrivsADMIN;
grant UPDATE ANY CUBE to BackupPrivsADMIN;
grant DROP ANY PROCEDURE to BackupPrivsADMIN;
grant CREATE ANY OPERATOR to BackupPrivsADMIN;
grant MANAGE FILE GROUP to BackupPrivsADMIN;
grant INSERT ANY MEASURE FOLDER to BackupPrivsADMIN;
grant BECOME USER to BackupPrivsADMIN;
grant DROP ANY VIEW to BackupPrivsADMIN;
grant ALTER ANY TYPE to BackupPrivsADMIN;
grant DROP ANY CUBE BUILD PROCESS to BackupPrivsADMIN;
grant EXECUTE on SYS.UTL_SMTP to BackupPrivsADMIN;
grant EXECUTE on SYS.UTL_MAIL to BackupPrivsADMIN;

Procedimiento PL/SQL terminado correctamente.

create user arantxaora identified by arantxaora;

Usuario creado.

exec CrearRol('ARANTXAORA');
CREATE ROLE BackupPrivsARANTXAORA

Procedimiento PL/SQL terminado correctamente.
```