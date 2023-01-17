# Alumno 2

## Postgres:

---

### Averigua que privilegios de sistema hay en Postgres y como se asignan a un usuario.

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

### Averigua cual es la forma de asignar y revocar privilegios sobre una tabla concreta en Postgres.

La forma de asignar privilegios en sobre una tabla en concreto tiene la siguiente sintaxis:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE table1, table2...
```

Para revocar los privilegios de una tabla:

```sql
REVOKE * ON TABLE table1, table2...
```

### Averigua si existe el concepto de rol en Postgres y señala las diferencias con los roles de ORACLE.

En postgres el concepto de rol si existe, pero a diferencia de ORACLE, se puede contemplar lo siguiente:

- La principal diferencia del concepto de rol de postgres con respecto a Oracle, principalmente 

### Averigua si existe el concepto de perfil como conjunto de límites sobre el uso de recursos o sobre la contraseña en Postgres y señala las diferencias con los perfiles de ORACLE.



### Realiza consultas al diccionario de datos de Postgres para averiguar todos los privilegios que tiene un usuario concreto.



### Realiza consultas al diccionario de datos en Postgres para averiguar qué usuarios pueden consultar una tabla concreta.

----

## ORACLE:

### Realiza una función de verificación de contraseñas que compruebe que la contraseña difiere en más de tres caracteres de la anterior y que la longitud de la misma es diferente de la anterior. Asígnala al perfil CONTRASEÑASEGURA. Comprueba que funciona correctamente.



### Realiza un procedimiento llamado MostrarPrivilegiosdelRol que reciba el nombre de un rol y muestre los privilegios de sistema y los privilegios sobre objetos que lo componen.

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