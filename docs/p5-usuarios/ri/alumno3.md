# Alumno 3

## MySQL:

### 1. Averigua qué privilegios de sistema hay en MySQL y como se asignan a un usuario.

En Mysql no hay privilegios de sistema como tal.

MySQL permite otorgar privilegios en seis niveles diferentes, en orden descendente del alcance de los privilegios son los siguientes:

- **Global**: los privilegios globales son administrativos o se aplican a todas las bases de datos en un servidor determinado. Se almacenan en la tabla del sistema mysql.user, y para asignarlos se usa la sintaxis: ON *.*
- **Nivel de base de datos**: los privilegios de base de datos se aplican a todos los objetos de una base de datos determinada. Se almacenan en la tabla del sistema mysql.db. Para asignarlos se usa la sintaxis: ON db_name.*
- **Privilegios de tabla**: los privilegios de tabla se aplican a todas las columnas de una tabla determinada. Se almacenan en la tabla del sistema mysql.tables_priv. Para asignar privilegios a nivel de tabla, hay que usar la sintaxis: ON db_name.tbl_name
- **Privilegios de columna**: los privilegios de columna se aplican a columnas individuales en una tabla determinada. Se almacenan en la tabla del sistema mysql.columns_priv. Cada privilegio que se concederá a nivel de columna debe ir seguido de la columna o columnas entre paréntesis.
- **Privilegios de rutinas almacenadas**: los privilegios ALTER ROUTINE, CREATE ROUTINE, EXECUTE y GRANT OPTION se aplican a las rutinas almacenadas (procedimientos y funciones). Se pueden otorgar a nivel global y de base de datos. Excepto para CREATE RUTINE, estos privilegios se pueden otorgar para rutinas individuales. Se almacenan en la tabla del sistema mysql.procs_priv.
- **Privilegios de usuario proxy**: el privilegio PROXY permite que un usuario sea proxy para otro. El usuario apoderado suplanta o toma la identidad del usuario apoderado; es decir, asume los privilegios del usuario proxy. Se alamacenan en la tabla del sistema mysql.proxies_priv.

Para ver los privilegios del usuario con el que hayamos accedido a la base de datos:

```sql
show grants;
```

Para ver los privilegios de un usuario específico:

```sql
show grants for arantxa;
```

Para mostrar la lista de todos los privilegios que tiene MySQL:

```sql
show privileges;
```

**Sintaxis:**

```sql
GRANT privilegio ON basedatos.tabla TO 'usuario|rol'@'host'  [WITH GRANT OPTION]
```

>WITH GRANT OPTION: Si queremos que el usuario al que le damos el privilegio pueda dar ese mismo privilegio.

**Ejemplo:**

```sql
GRANT all ON bd1.* TO 'arantxa'@'localhost';
```

### 2. Averigua cual es la forma de asignar y revocar privilegios sobre una tabla concreta en MySQL.

Como vimos en el punto anterior, para asignar privilegios a nivel de tabla, se usa la sintaxis **ON db_name.tbl_name**. Por ejemplo:

```sql
grant select on maravilla.actor to 'arantxa'@'localhost';
```

Para revocar permisos a nivel de tabla se usaría la siguiente sintaxis:

```sql
REVOKE privilegio ON basedatos.tabla FROM 'usuario|rol'@'host';
```

Ejemplo:

```sql
revoke select on maravilla.actor from 'arantxa'@'loaclhost';
```

### 3. Averigua si existe el concepto de rol en MySQL y señala las diferencias con los roles de ORACLE.

En MySQL sí existen los roles. Los roles de MySQL son colecciones de privilegios. Al igual que a los usuarios, a los roles se le pueden otorgar o revocar privilegios.

A un usuario se le puede otorgar roles, lo que otorga al usuario los privilegios asociados con cada rol asociado. Esto permite la asignación de conjuntos de privilegios a los usuarios.

A continuación se ven las posibilidades de gestión de roles proporcionadas por MySQL:

- CREATE ROLE y DROP ROLE: crean y eliminan roles.

- GRANT y REVOKE: asignan y revocan privilegios de usuarios y roles.

- SHOW GRANTS: muestra los roles y privilegios que pertenecen a un usuario o un rol.

- SET DEFAULT ROLE: especifica qué roles de cuenta están activos de forma predeterminada.

- SET ROLE: cambia los roles activos dentro de la sesión actual.

- La función CURRENT_ROLE() muestra los roles activos dentro de la sesión actual.

- Las variables de sistema mandatory_roles y activate_all_roles_on_login permiten definir roles obligatorios y la activación automática de roles otorgados cuando los usuarios inician sesión en el servidor.

En Oracle la gestión de los roles es muy similar a MySQL, la diferencia está en que tiene más funcionalidades.

Para empezar en Oracle los roles no solo son colecciones de privilegios sino que también puede agrupar otros roles.

Además en Oracle se proporcionan algunos roles ya predefinidos para ayudar en la administración. Estos roles se definen automáticamente cuando se crea la base de datos de Oracle. Algunos roles predefinidos son: CONNECT, RESOURCE, DBA, EXP_FULL_DATABASE, SELECT_CATALOG_ROLE, EXECUTE_CATALOG_ROLE...

Otra diferencia es que en Oracle, a la hora de crear un rol se le puede especificar la forma de identificación. Por ejemplo mediante una contraseña que tendrá que poner el usuario antes de usar el rol.

```sql
CREATE ROLE nombrerol IDENTIFIED BY contraseña;
```

O identificándose mediante una aplicación.

```sql
CREATE ROLE nombrerolapp IDENTIFIED USING nombrepaquetePLSQL;
```

> IDENTIFIED USING package_name: permite crear un rol de aplicación, y especificar qué paquete PL/SQL está autorizado para habilitar el rol.

Otras formas de crear un rol mediante autorización son las siguientes:

```sql
CREATE ROLE nombrerol IDENTIFIED EXTERNALLY;
CREATE ROLE nombrerol IDENTIFIED GLOBALLY;
```

Una diferencia entre ambos SGBD es la forma de consultar los privilegios asociados a un rol o los roles asociados a un usuario también es algo diferente.

**En MySQL:**

```sql
SHOW GRANTS FOR usuario|rol [USING rol];
```

> La cláusula USING es opcional, y le prmite a SHOW GRANTS examinar los privilegios asociados con los roles del usuario (el rol mencionado en la cláusula USING debe haberse otorgado al usuario).

**En Oracle:**

- Listar todos los roles otorgados:

```sql
SELECT * FROM DBA_ROLE_PRIVS;
```

- Listar los roles de la sesión actual.

```sql
SELECT * FROM SESSION_ROLES;
```

- Listar roles de la base de datos.

```sql
SELECT * FROM DBA_ROLES;
```

Por último, en MySQL he visto que existen los roles obligatorios (mandatory roles). Los roles obligatorios permiten especificar roles como obligatorios nombrándolos en el valor de la variable de sistema mandatory_roles. El servidor trata un rol obligatorio como si fuera otorgado a todos los usuarios, por lo que no es necesario otorgarlo explícitamente a ninguna cuenta. Para especificar funciones obligatorias al inicio del servidor se definen los mandatory_roles en el archivo my.cnf del servidor.

### 4. Averigua si existe el concepto de perfil como conjunto de límites sobre el uso de recursos o sobre la contraseña en MySQL y señala las diferencias con los perfiles de ORACLE.

En MySQL no existe la creación de perfiles en sí, la forma de limitar los recursos a los usuarios se hace de forma diferente a como se hace en Oracle.

Para restringir el uso que los usuarios hacen del servidor MySQL se debe establecer la variable del sistema **max_user_connections** en un valor distinto a cero. Esto limita la cantidad de conexiones simultáneas que puede realizar una cuenta determinada, pero no impone límites a lo que un cliente puede hacer una vez conectado. Configurar max_user_connections no habilita la administración de cuentas individuales, opción que es interesante para los administradores y que sí tenemos en Oracle.

Para contrarrestar esta limitación MySQL, a la hora de usar alguno de los siguientes recursos, sí permite poner límites para cuentas individuales. Esos recursos son:

- El número de consultas que una cuenta puede emitir por hora.
- La cantidad de actualizaciones que una cuenta puede emitir por hora.
- La cantidad de veces que una cuenta puede conectarse al servidor por hora.
- El número de conexiones simultáneas al servidor por una cuenta.

Cualquier declaración que un cliente pueda emitir tiene en contra el límite de consultas. Solo las declaraciones que modifican bases de datos o tablas cuentan para el límite de actualización.

Se pueden limitar los recursos de una cuenta a la vez que se utiliza CREATE USER o ALTER USER, usándolo junto con WITH y el recurso a limitar. Por ejemplo:

```sql
CREATE USER 'arantxa'@'localhost' IDENTIFIED BY '1234'
    ->     WITH MAX_QUERIES_PER_HOUR 20
    ->          MAX_UPDATES_PER_HOUR 10
    ->          MAX_CONNECTIONS_PER_HOUR 5
    ->          MAX_USER_CONNECTIONS 2;

ALTER USER 'arantxa'@'localhost' WITH MAX_QUERIES_PER_HOUR 100;

--Para eliminar la limitación de un recurso habrá que ponerlo a 0
ALTER USER 'arantxa'@'localhost' WITH MAX_CONNECTIONS_PER_HOUR 0;
```

En cuanto a las contraseñas, MySQL permite las siguientes opciones de administración:

- Caducidad de la contraseña, para exigir que las contraseñas se cambien periódicamente.
- Restricciones de reutilización de contraseñas, para evitar que se vuelvan a elegir contraseñas antiguas.
- Verificación de contraseña, para solicitar que los cambios de contraseña también especifiquen la contraseña actual para ser reemplazada.
- Contraseñas duales, para permitir que los clientes se conecten usando una contraseña principal o secundaria.
- Evaluación de la seguridad de la contraseña, para exigir contraseñas seguras.
- Generación aleatoria de contraseñas, como alternativa a la necesidad de contraseñas literales explícitas especificadas por el administrador.
- Seguimiento de fallas de contraseña, para habilitar el bloqueo temporal de la cuenta después de demasiadas fallas consecutivas de inicio de sesión con contraseña incorrecta.

Para más información sobre cómo se gestionan las anteriores limitaciones y la sintaxis que se utiliza se puede consultar la [web oficial](https://dev.mysql.com/doc/mysql-security-excerpt/8.0/en/password-management.html).

En Oracle la creación de perfiles se utiliza para limitar los recursos que pueda utilizar el usuario. A cada usuario se le puede asignar un perfil ya creado, y cada perfil puede tener diferentes limitaciones, por ejemplo limitaciones de los recursos del kernel (uso de la CPU, duración de la sesión...) y limitaciones en el uso de las contraseñas de acceso (duración, intentos de acceso, cambio de contraseñas, longitud y caráceres permitidos...).

Que las limitaciones se puedan agrupar en un perfil concreto es una gran ventaja para Oracle, ya que no habrá que asignar las limitaciones a cada usuario concreto, se podrá crear un perfil que sirva para muchos usuarios. Además Oracle tiene más opciones de limitación y permite al administrador llevar un mayor y mejor control de lo que hacen los usuarios en la base de datos.

### 5. Realiza consultas al diccionario de datos de MySQL para averiguar todos los privilegios que tiene un usuario concreto.

Como hemos dicho anteriormente, para consultar los privilegios de un usuario concreto habrá que utilizar SHOW GRANT. En ese caso vamos a consultar los privilegios que tiene mi usuario llamado "arantxa".

```sql
mysql> show grants for arantxa;
+--------------------------------------------------------+
| Grants for arantxa@%                                   |
+--------------------------------------------------------+
| GRANT USAGE ON *.* TO `arantxa`@`%`                    |
| GRANT ALL PRIVILEGES ON `maravilla`.* TO `arantxa`@`%` |
| GRANT SELECT ON `maravilla`.`actor` TO `arantxa`@`%`   |
+--------------------------------------------------------+
3 rows in set (0,00 sec)
```

### 6. Realiza consultas al diccionario de datos en MySQL para averiguar qué usuarios pueden consultar una tabla concreta.

En el primer ejercicio vimos que los privilegios a nivel de tabla se almacenan en la tabla del sistema **mysql.tables_priv**. Por tanto, para comprobar los usuarios que pueden consultar una tabla concreta habrá que hacer una consulta a mysql.tables_priv.

```sql
mysql> select * from mysql.tables_priv;
+-----------+-----------+---------------+------------+----------------+---------------------+------------+-------------+
| Host      | Db        | User          | Table_name | Grantor        | Timestamp           | Table_priv | Column_priv |
+-----------+-----------+---------------+------------+----------------+---------------------+------------+-------------+
| %         | maravilla | arantxa       | actor      | root@localhost | 2023-02-02 18:17:23 | Select     |             |
| localhost | mysql     | mysql.session | user       | boot@          | 2022-10-26 09:23:03 | Select     |             |
| localhost | sys       | mysql.sys     | sys_config | root@localhost | 2022-10-26 09:23:03 | Select     |             |
+-----------+-----------+---------------+------------+----------------+---------------------+------------+-------------+
3 rows in set (0,00 sec)
```

En el anterior caso comprobamos que mi usuario puede consultar (select) la tabla actor de la base de datos maravilla, y que el usuario que le ha dado los permisos es root.

## ORACLE:

### 1. Realiza un procedimiento llamado PermisosdeAsobreB que reciba dos nombres de usuario y muestre los permisos que tiene el primero de ellos sobre objetos del segundo.

```sql
CREATE OR REPLACE PROCEDURE PermisosdeAsobreB (p_usuarioA VARCHAR2, p_usuarioB VARCHAR2)
AS
BEGIN
    FOR obj IN (SELECT grantee, owner, table_name, privilege
                FROM dba_tab_privs
                WHERE grantee = p_usuarioA AND owner = p_usuarioB)
    LOOP
        DBMS_OUTPUT.PUT_LINE('El usuario ' || p_usuarioA || ' tiene el privilegio ' || obj.privilege || ' sobre la tabla ' || obj.table_name || ' de ' || obj.owner);
    END LOOP;
END;
/

exec PermisosdeAsobreB ('ARANTXA','ADMIN');
```

**Comprobación:**

> Los nombres de los usuarios deben estar en mayúsculas, ya que así es como se guardan en las vistas del diccionario de datos.

![comprobacion](/img/capturas-arantxa/91.png)

### 2. Realiza un procedimiento llamado MostrarInfoPerfil que reciba el nombre de un perfil y muestre su composición y los usuarios que lo tienen asignado.

```sql
create or replace procedure MostrarInfoPerfil (p_perfil_name dba_profiles.profile%type)
is
    cursor c_informacion is
        select resource_name, resource_type, limit
        from dba_profiles
        where profile = p_perfil_name;
    cursor c_usuarios is
        select username
        from dba_users
        where profile = p_perfil_name;

begin
    dbms_output.put_line(CHR(10)||'Composicion del perfil '|| p_perfil_name );
    dbms_output.put_line('--------------------------');
    for i in c_informacion loop
        dbms_output.put_line('Recurso: '|| i.resource_name ||CHR(9)||CHR(9)||'Tipo de recurso: '|| i.resource_type ||CHR(9)||CHR(9)||' Limite: '|| i.limit);
    end loop;
    dbms_output.put_line(CHR(10)||'Usuarios con ese perfil: ');
    dbms_output.put_line('--------------------------');
    for i in c_usuarios loop
        dbms_output.put_line(i.username);
    end loop;
end MostrarInfoPerfil;
/
```

![compilacion](/img/capturas-arantxa/94.png)

**Comprobación:**

Habilitar los perfiles de ususario en Oracle:

```sql
alter session set "_ORACLE_SCRIPT"=true;
ALTER SYSTEM SET RESOURCE_LIMIT=TRUE;
```

He creado un perfil de prueba para el usuario 'arantxa' y 'admin' que solo permite tres sesiones concurrentes, el tiempo de conexión es de 240 minutos y el tiempo de inactividad no es mayor a 5 minutos. Los demás valores serán por defecto. Le asigno el nuevo perfil a los usuarios.

```sql
CREATE PROFILE admin_perfil_prueba
LIMIT
  SESSIONS_PER_USER 3
  CONNECT_TIME 240
  IDLE_TIME 5;

ALTER USER arantxa PROFILE admin_perfil_prueba;
ALTER USER admin PROFILE admin_perfil_prueba;

```

![comprobacion](/img/capturas-arantxa/92.png)

Probamos el procedimiento creado anteriormente:

```sql
exec MostrarInfoPerfil('ADMIN_PERFIL_PRUEBA');
```

![comprobacion](/img/capturas-arantxa/93.png)
