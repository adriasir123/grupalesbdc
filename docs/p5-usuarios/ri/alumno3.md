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

### 5. Realiza consultas al diccionario de datos de MySQL para averiguar todos los privilegios que tiene un usuario concreto.

### 6. Realiza consultas al diccionario de datos en MySQL para averiguar qué usuarios pueden consultar una tabla concreta.



## ORACLE:
       
### 1. Realiza un procedimiento llamado PermisosdeAsobreB que reciba dos nombres de usuario y muestre los permisos que tiene el primero de ellos sobre objetos del segundo.

### 2. Realiza un procedimiento llamado MostrarInfoPerfil que reciba el nombre de un perfil y muestre su composición y los usuarios que lo tienen asignado.