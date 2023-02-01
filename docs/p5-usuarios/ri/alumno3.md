# Alumno 3

## MySQL:

### 1. Averigua que privilegios de sistema hay en MySQL y como se asignan a un usuario.

En Mysql no hay privilegios de sistema como tal.

MySQL permite otorgar privilegios en seis niveles diferentes, en orden descendente del alcance de los privilegios son los siguientes:

- Global: Los privilegios globales son administrativos o se aplican a todas las bases de datos en un servidor determinado. Se almacenan en la tabla del sistema mysql.user. Para asignar privilegios globales, se usa la sintaxis: ON *.*
- Nivel de base de datos: Los privilegios de la base de datos se aplican a todos los objetos de una base de datos determinada. Se almacenan en la tabla del sistema mysql.db. Para asignar privilegios a nivel de base de datos, use la sintaxis: ON db_name.*
- Privilegios de tabla: Los privilegios de tabla se aplican a todas las columnas de una tabla determinada. Se almacenan en la tabla del sistema mysql.tables_priv. Para asignar privilegios a nivel de tabla, use la sintaxis: ON db_name.tbl_name
- Privilegios de columna: Los privilegios de columna se aplican a columnas individuales en una tabla determinada. Se almacenan en la tabla del sistema mysql.columns_priv. Cada privilegio que se concederá a nivel de columna debe ir seguido de la columna o columnas, entre paréntesis.
- Privilegios de rutinas almacenadas: Los privilegios ALTER ROUTINE, CREATE ROUTINE, EXECUTE y GRANT OPTION se aplican a las rutinas almacenadas (procedimientos y funciones). Se pueden otorgar a nivel global y de base de datos. Excepto para CREAR RUTINA, estos privilegios se pueden otorgar a nivel de rutina para rutinas individuales. Se almacenan en la tabla del sistema mysql.procs_priv.
- Privilegios de usuario proxy: El privilegio PROXY permite que un usuario sea un proxy para otro. El usuario apoderado suplanta o toma la identidad del usuario apoderado; es decir, asume los privilegios del usuario proxy. Se alamacenan en la tabla del sistema mysql.proxies_priv.

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

GRANT privilegio ON basedatos.tabla TO 'usuario|rol'@'host'  [WITH GRANT OPTION]

>WITH GRANT OPTION: Si queremos que el usuario al que le damos el privilegio pueda dar ese mismo privilegio.

**Ejemplo:**

GRANT all ON bd1.* TO 'arantxa'@'localhost';

### 2. Averigua cual es la forma de asignar y revocar privilegios sobre una tabla concreta en MySQL.
       
### 3. Averigua si existe el concepto de rol en MySQL y señala las diferencias con los roles de ORACLE.
       
### 4. Averigua si existe el concepto de perfil como conjunto de límites sobre el uso de recursos o sobre la contraseña en MySQL y señala las diferencias con los perfiles de ORACLE.

### 5. Realiza consultas al diccionario de datos de MySQL para averiguar todos los privilegios que tiene un usuario concreto.

### 6. Realiza consultas al diccionario de datos en MySQL para averiguar qué usuarios pueden consultar una tabla concreta.



## ORACLE:
       
### 1. Realiza un procedimiento llamado PermisosdeAsobreB que reciba dos nombres de usuario y muestre los permisos que tiene el primero de ellos sobre objetos del segundo.

### 2. Realiza un procedimiento llamado MostrarInfoPerfil que reciba el nombre de un perfil y muestre su composición y los usuarios que lo tienen asignado.