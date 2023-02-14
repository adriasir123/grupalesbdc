# Alumno 1


1. Muestra los espacios de tablas existentes en tu base de datos y la ruta de los ficheros que los componen. ¿Están las extensiones gestionadas localmente o por diccionario?



select FILE_NAME, TABLESPACE_NAME, BLOCKS from dba_data_files UNION select FILE_NAME, TABLESPACE_NAME, BLOCKS from dba_temp_files;


2. Usa la vista del diccionario de datos v$datafile para mirar cuando fue la última vez que se ejecutó el proceso CKPT en tu base de datos.


select max(CHECKPOINT_TIME) from v$datafile;


select min(CHECKPOINT_TIME) from v$datafile;


!![alumno1.png](/img/capturas-antonio/practica5-ejercicio1-1.png)


!!! note "Qué es CKPT?"

    CKPT (Checkpoint): Este proceso escribe en los ficheros de control los checkpoints. Estos puntos de sincronización son referencias al estado coherente de todos los ficheros de la BD en un instante determinado, en un punto de sincronización. Esto significa que los bloques sucios de la BD se vuelcan a los ficheros de BD, asegurándose de que todos los bloques de datos modificados desde el último checkpoint se escriben realmente en los ficheros de datos y no sólo en los ficheros redo log; y que los ficheros de redo log también almacenan los registros de redo log hasta este instante. La secuencia de puntos de control se almacena en los ficheros de datos, redo log y control. Los checkpoints se produce cuando:
    * un espacio de tabla se pone inactivo, offline,
    * se llena el fichero de redo log activo,
    * se para la BD,
    * el número de bloques escritos en el redo log desde el último checkpoint alcanza el límite definido en el parámetro LOG_CHECKPOINT_INTERVAL,
    * cuando transcurra el número de segundos indicado por el parámetro LOG_CHECKPOINT_TIMEOUT desde el último checkpoint.

3. Intenta crear el tablespace TS1 con un fichero de 2M en tu disco que crezca automáticamente cuando sea necesario. ¿Puedes hacer que la gestión de extensiones sea por diccionario? Averigua la razón.

```sql
create tablespace ts1 
datafile 'ts1.dbf' size 2M 
autoextend on;
```

A partir de Oracle9 los extends se gestionan de manera local, con el fin de evitar operaciones recurrentes al diccionario de datos, es lo más óptimo a la hora de administrar la base de datos.


Ejemplo de creación de un tablespace con gestión de extensiones por diccionario:

```sql
create tablespace ts1 
datafile 'ts1.dbf' size 2M 
EXTENT MANAGEMENT DICTIONARY
DEFAULT STORAGE (
INITIAL 64K
NEXT 64K
MINEXTENTS 2
MAXEXTENTS 121
PCTINCREASE 0);
```

4. Averigua el tamaño de un bloque de datos en tu base de datos. Cámbialo al doble del valor que tenga.

```sql
select distinct bytes/blocks from user_segments;

o bien

select name, block_size, current_size from v$buffer_pool;


select tablespace_name, block_size from dba_tablespaces;
```


```sql
ALTER SYSTEM SET DB_16k_CACHE_SIZE=80M;
```

Ahora podemos ver reflejado el cambio en el tamaño de los bloques de datos:

![alumno1.png](/img/capturas-antonio/practica5-ejercicio4-1.png)


create tablespace ts_doble datafile 'tsdoble.dbf' size 2M blocksize 16K;


select tablespace_name, block_size from dba_tablespaces where tablespace_name='ts_doble';


![alumno1.png](/img/capturas-antonio/practica5-ejercicio4-2.png)


5. Realiza un procedimiento MostrarObjetosdeUsuarioenTS que reciba el nombre de un tablespace y el de un usuario y muestre qué objetos tiene el usuario en dicho tablespace y qué tamaño tiene cada uno de ellos.

```sql
create or replace procedure MostrarObjetosdeUsuarioenTS(p_tablespace VARCHAR2,p_usuario VARCHAR2)
is

cursor c_tablespace is select segment_name, bytes from dba_segments where tablespace_name=p_tablespace and owner=p_usuario;

begin
	dbms_output.put_line('User:  '||p_usuario||CHR(9) ||'Size: ');
	for v_tablespace in c_tablespace loop
		dbms_output.put_line(v_tablespace.segment_name||chr(9)||v_tablespace.bytes);
	end loop;
end;
/
```

![alumno1.png](/img/capturas-antonio/practica5-ejercicio5-1.png)

6. Realiza un procedimiento llamado MostrarUsrsCuotaIlimitada que muestre los usuarios que puedan escribir de forma ilimitada en más de uno de los tablespaces que cuentan con ficheros en la unidad C:


```sql
create or replace procedure MostrarUsrsCuotaIlimitada
is
cursor c_cuota is
select grantee
from DBA_SYS_PRIVS
where privilege = 'UNLIMITED TABLESPACE' union select
username from DBA_TS_QUOTAS where tablespace_name in (select tablespace_name
from DBA_DATA_FILES where file_name = '/home/oracle');
begin
	dbms_output.put_line('Usuarios con cuota ilimitada');
	for v_cuota in c_cuota loop
		dbms_output.put_line(v_cuota.grantee);
	end loop;
end;
/
```

Aquí relacionamos al usuario con la cuota ilimitada y los tablespaces que tiene en /home/oracle, de esta manera podemos ver la lista de usuarios que tienen cuota ilimitada en /home/oracle;

![alumno1.png](/img/capturas-antonio/practica5-ejercicio6-1.png)


Postgres:
       
7. Averigua si existe el concepto de tablespace en Postgres, en qué consiste y las diferencias con los tablespaces de ORACLE.

Los tablespaces en Postgres es la versión reducida de la de Oracle, pero presenta una mejora, ya que se pueden almacenar estos tablespaces en diferentes directorios, lo que permite una mejor gestión de los mismos.

La gran diferencia con Oracle es que los tablespaces de Postgres son objetos de nivel de cluster y no hay posibilidad de restringir el tamaño del tablespace (por lo tanto, del cluster/base de datos). No tienes que lidiar con datafiles, sino con directorios.

Ejemplo de crear un tablespace en Postgres:

```sql
CREATE TABLESPACE tablespace_name
OWNER user_name
LOCATION directory_path;
```


MySQL:

1. Averigua si pueden establecerse claúsulas de almacenamiento para las tablas o los espacios de tablas en MySQL.

Existe, pero es una extensión de la base de datos llamada InnoDB, que es la que permite el uso de estos tablespaces.

Ejemplo de creación de un tablespace en MySQL:

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;
```

Como podemos observar, la sintaxis es bastante parecida a la que encontramos en Oracle, esto es debido a que los desarrolladores de MySQL han tomado como base los tablespaces que podemos encontrar en Oracle.


MongoDB:

9. Averigua si existe el concepto de índice en MongoDB y las diferencias con los índices de ORACLE. Explica los distintos tipos de índice que ofrece MongoDB.


En Oracle y MongoDB, los índices son responsables de acelerar la consulta de búsqueda proporcionando un acceso rápido. Los índices en el sistema de gestión de bases de datos relacional o no relacional no siempre son relevantes. Los índices son excelentes para reducir el tiempo que requiere buscar y obtener datos utilizando las declaraciones SELECT.

A veces los índices pueden disminuir el rendimiento de las consultas de actualización, inserción o eliminación. En Oracle, el índice se crea normalmente para obtener el primer conjunto primario de filas rápidamente cuando la tabla contiene demasiados datos. Esto es responsable del rendimiento general de la base de datos.

Predefinidamente, mongodb tiene un index que filtra por '_id' del documento, este index es único pero podemos crear otros para mejorar nuestra búsqueda: 

```sql
db.SOULSWEAPONS.find({name : "Black Knight Sword"}).explain("executionStats")
```

![alumno1.png](/img/capturas-antonio/practica5-ejercicio8-1.png)

```sql
db.SOULSWEAPONS.createIndex({name : 1})
```


```sql
db.SOULSWEAPONS.find({name : "Black Knight Sword"}).explain("executionStats")
```

![alumno1.png](/img/capturas-antonio/practica5-ejercicio8-2.png)



Con esto podemos ver como el tiempo de ejecución se ha reducido al mínimo, esto es debido a que hemos ajustado el índice a partir del nombre de forma ascendente.


Tipos de índices en MongoDB:

Índices simples o de un solo campo:

Estos índices se aplican a un solo campo de nuestra colección. Para declarar un índice de este tipo debemos usar un comando similar a este:

```sql
db.users.ensureIndex( { "user_id" : 1 } )
```

El número indica que queremos que el índice se ordene de forma ascendente. Si quisiéramos un orden descendente, el parámetro será un -1.

Índices compuestos
En este caso el índice se generará sobre varios campos.

```sql
db.users.ensureIndex( { "user_name" : 1, "age":-1 } )
```
El índice que se generará con la instrucción anterior, agrupará los datos primero por el campo user_name y luego por el campo age. Es decir, se generaría algo así:

```sql
    "Antonio", 35
    "Antonio", 18
    "María", 56
    "María", 30
    "María", 21
    "Pedro", 19
    "Unai", 34
    "Unai", 27
```

Índices únicos
Los índices simples y múltiples, pueden estar obligados a contener valores únicos. Esto lo conseguimos añadiendo el parámetro unique a la hora de crearlos.

```sql
    db.users.ensureIndex( { "user_id" : 1 }, {"unique":true} )
```

En el caso de que tratemos de insertar valores repetidos en algún campo con un índice único, MongoDB nos devolverá un error.

Índices sparse
Los índices que hemos mencionado antes, incluyen todos los documentos. También los documentos que no contengan el campo indexado. MongoDB no obliga a que usemos un esquema, así que no hay obligatoriedad en que los campos existan en el documento. Vamos, que el campo user_name puede estar en el documento, o no.

Para crear índices que solo incluyan los documentos cuyo campo indexado existe, utilizaremos la opción sparse.


```sql
db.users.ensureIndex( { "user_name" : 1 }, {"sparse":true} )
```

En este caso lo que hacemos es crear un índice que no tiene porque contener todos los documentos de la colección.