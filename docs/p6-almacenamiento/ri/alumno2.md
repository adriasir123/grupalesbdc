# Alumno 2

## Oracle

1-  Establece que los objetos que se creen en el TS1 (creado por Alumno 1) tengan un tamaño inicial de 200K, y que cada extensión sea del doble del tamaño que la anterior. El número máximo de extensiones debe ser de 3.

Puedes establecer el tamaño inicial de los objetos en el Tablespace TS1 utilizando el siguiente comando SQL:

```sql
SQL> ALTER TABLESPACE TS1 ADD DATAFILE 'TS1_01.dbf' SIZE 200K AUTOEXTEND ON NEXT 200K MAXSIZE 800K;

Tablespace modificado.
```

Esto especifica que el tamaño inicial del objeto es de 200K y que cada vez que se llene, se agregará automáticamente una extensión del doble del tamaño anterior, hasta un máximo de 3 extensiones (800K).

2-  Crea dos tablas en el tablespace recién creado e inserta un registro en cada una de ellas. 

Puedes crear dos tablas en el tablespace TS1 y insertar un registro en cada una de ellas mediante los siguientes comandos SQL:

```sql 
SQL> CREATE TABLE tabla1 (
  id NUMBER PRIMARY KEY,
  nombre VARCHAR2(50),
  fecha DATE
) TABLESPACE TS1;

CREATE TABLE tabla2 (
  id NUMBER PRIMARY KEY,
  descripcion VARCHAR2(100),
  valor NUMBER
) TABLESPACE TS1;

INSERT INTO tabla1 (id, nombre, fecha) VALUES (1, 'Registro 1', SYSDATE);
INSERT INTO tabla2 (id, descripcion, valor) VALUES (1, 'Registro 1', 123.45);  

SQL>

Tabla creada.

SQL>

Tabla creada.

SQL>  
1 fila creada.

SQL> 
1 fila creada.
```

Esto creará dos tablas llamadas tabla1 y tabla2 en el tablespace TS1 y agregará un registro en cada una de ellas.

Comprueba el espacio libre existente en el tablespace:

```sql
SQL> SELECT TABLESPACE_NAME, SUM(BYTES) / 1024 / 1024 AS "Size (MB)", 
  (SUM(BYTES) - SUM(MAXBYTES)) / 1024 / 1024 AS "Free (MB)"
FROM DBA_DATA_FILES
WHERE TABLESPACE_NAME = 'TS1'
GROUP BY TABLESPACE_NAME;

TABLESPACE_NAME 		Size (MB)  Free (MB)
------------------------------ ---------- ----------
TS1				        2,1953125 -18,585938
```

Este comando te proporcionará el nombre del tablespace, su tamaño en MB y la cantidad de espacio libre en MB.

Borra una de las tablas y comprueba si ha aumentado el espacio disponible en el tablespace:  

```sql
SQL> DROP TABLE tabla1;

Tabla borrada.

SQL> SELECT TABLESPACE_NAME, SUM(BYTES) / 1024 / 1024 AS "Size (MB)", 
  (SUM(BYTES) - SUM(MAXBYTES)) / 1024 / 1024 AS "Free (MB)"
FROM DBA_DATA_FILES
WHERE TABLESPACE_NAME = 'TS1'
GROUP BY TABLESPACE_NAME;

TABLESPACE_NAME 		Size (MB)  Free (MB)
------------------------------ ---------- ----------
TS1				        2,1953125 -18,585938

```

Este comando borrará la tabla tabla1 y luego mostrará el nombre del tablespace, su tamaño en MB y la cantidad de espacio libre en MB, lo que te permitirá ver si el espacio disponible en el tablespace ha aumentado después de borrar la tabla.

Explica la razón.
```
Es posible que el espacio liberado por la eliminación de la tabla no se haya reflejado en el tablespace inmediatamente después de la eliminación debido a que Oracle no libera de forma automática el espacio ocupado por las tablas eliminadas. 

Oracle utiliza un mecanismo llamado "asignación en bloques" que reserva bloques de espacio en el tablespace para una tabla y, aunque la tabla se elimine, los bloques asignados a ella pueden continuar ocupados hasta que se necesiten para nuevos datos.

Para liberar de forma explícita el espacio ocupado por las tablas eliminadas en el tablespace, puedes ejecutar el comando SHRINK SPACE en la base de datos. Sin embargo, tenga en cuenta que este proceso puede ser costoso y requiere una cantidad significativa de recursos. 

Por lo tanto, solo se recomienda ejecutar este comando cuando sea necesario y después de haber evaluado cuidadosamente sus posibles impactos en el rendimiento de la base de datos.
```

3- Convierte a TS1 en un tablespace de sólo lectura.

```sql
SQL> ALTER TABLESPACE TS1 READ ONLY;

Tablespace modificado.
```

Este comando hace que el tablespace TS1 sea de sólo lectura, lo que significa que los usuarios no podrán realizar ninguna operación de escritura en las tablas almacenadas en ese tablespace. Sin embargo, todavía podrán leer los datos almacenados en ese tablespace.

Hay que tener en cuenta que para convertir un tablespace en sólo lectura, todas las tablas en ese tablespace deben estar cerradas y no deben estar siendo accedidas por ningún proceso en el sistema.

Intenta insertar registros en la tabla existente. 

```sql
SQL> INSERT INTO tabla2 (id, descripcion, valor) VALUES (2, 'Registro 2', 143.45);

ERROR en linea 1:
ORA-00372: el archivo 14 no se puede modificar en este momento
ORA-01110: archivo de datos 14: '/opt/oracle/product/19c/dbhome_1/dbs/TS1.dbf'
```

¿Qué ocurre?
```
Si intentamos insertar un registro en una tabla que se encuentra en un tablespace que se ha convertido en sólo lectura, recibirás un error indicando que el tablespace es de sólo lectura y no se pueden realizar operaciones de escritura en el mismo. 

Esto significa que los datos en el tablespace no se pueden modificar o insertar nuevos datos.
```
Intenta ahora borrar la tabla.

```sql
SQL> drop table tabla2;

Tabla borrada.
```

¿Qué ocurre?
```
Que he eliminado la tabla 'tabla2'.
```
¿Porqué crees que pasa eso?
```
Puede ser debido a que estamos utilizando una versión más reciente de Oracle que permite esta acción. 

En versiones anteriores de Oracle, los tablespaces de sólo lectura no permitían realizar operaciones de escritura, incluyendo la eliminación de tablas. Sin embargo, en versiones más recientes, Oracle ha mejorado la gestión de los tablespaces de sólo lectura y es posible que permitan realizar ciertas operaciones de escritura, como la eliminación de tablas.
```
4- Crea un espacio de tablas TS2 con dos ficheros en rutas diferentes de 1M cada uno no autoextensibles. 

```sql
SQL> CREATE TABLESPACE TS2
DATAFILE '/home/oracle/ts21/ts2_1.dbf' SIZE 1M,
'/home/oracle/ts22/ts2_2.dbf' SIZE 1M
EXTENT MANAGEMENT LOCAL
SEGMENT SPACE MANAGEMENT AUTO;

Tablespace creado.
```

Crea en el citado tablespace una tabla con la clausula de almacenamiento que quieras.

```sql
SQL> CREATE TABLE employees (
   id NUMBER(10),
   name VARCHAR2(50),
   department VARCHAR2(30),
   salary NUMBER(10,2)
)
STORAGE (
   INITIAL 10K
   NEXT 10K
   MINEXTENTS 2
   MAXEXTENTS UNLIMITED
   PCTINCREASE 0
)
TABLESPACE ts2;

Tabla creada.
```

Inserta registros hasta que se llene el tablespace. 

```sql
SQL> DECLARE
   v_count NUMBER := 0;
BEGIN
   LOOP
      v_count := v_count + 1;
      INSERT INTO employees (id, name, department, salary)
      VALUES (v_count, 'Employee ' || v_count, 'Department ' || v_count, 1000 * v_count);
      EXIT WHEN v_count = 1000;
   END LOOP;
   COMMIT;
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/ 

Procedimiento PL/SQL terminado correctamente.
```

¿Qué ocurre?
```
Lo que ocurre depende de varios factores, como la configuración y el tamaño de su base de datos, la cantidad de espacio disponible en el disco duro y la configuración del espacio de tablas.

Si el espacio de tablas no está lleno, el bucle seguirá insertando registros hasta que se alcance el número especificado de registros o hasta que se produzca un error.

Si el espacio de tablas está lleno, se producirá un error y el bucle se detendrá. En el ejemplo proporcionado, el error se maneja en el bloque EXCEPTION y se imprime en la salida.

Es importante monitorear regularmente el uso de espacio de tablas para evitar problemas de rendimiento y errores debido a la falta de espacio. 

También es recomendable tener una política de gestión de espacio de tablas para prevenir la falta de espacio y garantizar la integridad de los datos.

El bucle LOOP se ejecutará hasta que se hayan insertado 1000 registros o hasta que se produzca un error. Cada registro se inserta en la tabla "EMPLOYEES" con un ID incremental, un nombre generado, un departamento generado y un salario generado. 

El bloque EXCEPTION maneja cualquier error que pueda ocurrir durante la ejecución del bucle.
```
5- Hacer un procedimiento llamado MostrarUsuariosporTablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios que tienen asignado cada uno de ellos por defecto y el número de los mismos, así:
```

Tablespace xxxx:

	Usr1
	Usr2
	...

Total Usuarios Tablespace xxxx: n1

Tablespace yyyy:

	Usr1
	Usr2
	...

Total Usuarios Tablespace yyyy: n2
....
Total Usuarios BD: nn
```

No olvides incluir los tablespaces temporales y de undo.

Primero, creamos y compilamos el código:

```sql
SQL> CREATE OR REPLACE PROCEDURE MostrarUsuariosporTablespace AS
  2    CURSOR c_ts IS SELECT tablespace_name, username FROM dba_ts_quotas UNION SELECT default_tablespace, username FROM dba_users WHERE default_tablespace NOT IN ('SYSTEM', 'SYSAUX') ORDER BY tablespace_name;
  3    v_ts_name VARCHAR2(30);
  4    v_ts_prev VARCHAR2(30) := NULL;
  5    v_us_count NUMBER := 0;
  6    v_tot_count NUMBER := 0;
  7  BEGIN
  8    FOR r_ts IN c_ts LOOP
  9      IF v_ts_prev IS NULL OR v_ts_prev != r_ts.tablespace_name THEN
  10        IF v_ts_prev IS NOT NULL THEN
  11          DBMS_OUTPUT.PUT_LINE('Total Usuarios Tablespace ' || v_ts_prev || ': ' || v_us_count);
  12        END IF;
  13        DBMS_OUTPUT.PUT_LINE('Tablespace ' || r_ts.tablespace_name || ':');
  14        v_ts_prev := r_ts.tablespace_name;
  15        v_us_count := 0;
  16      END IF;
  17      DBMS_OUTPUT.PUT_LINE('    ' || r_ts.username);
  18      v_us_count := v_us_count + 1;
  19      v_tot_count := v_tot_count + 1;
  20    END LOOP;
  21    DBMS_OUTPUT.PUT_LINE('Total Usuarios Tablespace ' || v_ts_prev || ': ' || v_us_count);
  22    DBMS_OUTPUT.PUT_LINE('Total Usuarios BD: ' || v_tot_count);
  23  END MostrarUsuariosporTablespace;
  24  /

PROCEDURE created.

Commit complete.
```

Posteriormente, lo ejecutamos y comprobamos que funciona:

```sql
SQL> exec MostrarUsuariosporTablespace;

Tablespace SYSAUX:
APPQOSSYS
AUDSYS
DBSFWUSER
GGSYS
GSMADMIN_INTERNAL
MDSYS
OLAPSYS
Total Usuarios Tablespace SYSAUX: 7
Tablespace SYSTEM:
LBACSYS
MDSYS
OUTLN
Total Usuarios Tablespace SYSTEM: 3
Tablespace TS2:
ANONYMOUS
APPQOSSYS
AUDSYS
C###JOSEJU10
CTXSYS
DBSFWUSER
DBSNMP
DIP
DVF
DVSYS
EXAMENPLSQL
GGSYS
GSMADMIN_INTERNAL
GSMCATUSER
GSMROOTUSER
GSMUSER
HIPODROMO
JOSEJU
JUVENIL
LBACSYS
MDDATA
MDSYS
OJVMSYS
OLAPSYS
ORACLE_OCM
ORDDATA
ORDPLUGINS
ORDSYS
OUTLN
REMOTE_SCHEDULER_AGENT
SI_INFORMTN_SCHEMA
SYS
SYS$UMF
SYSBACKUP
SYSDG
SYSKM
SYSRAC
WMSYS
XDB
Total Usuarios Tablespace TS2: 39
Total Usuarios BD: 49

PL/SQL procedure successfully completed.

Commit complete.
```

6-  Realiza un procedimiento llamado MostrarDetallesIndices que reciba el nombre de una tabla y muestre los detalles sobre los índices que hay definidos sobre las columnas de la misma.

Primero, creamos y compilamos el código:

```sql
SQL> CREATE OR REPLACE PROCEDURE MostrarDetallesIndices (p_table_name IN VARCHAR2)
  2  AS
  3    l_nombre_indice   VARCHAR2(30);
  4    l_nombre_columna  VARCHAR2(30);
  5    cursor c_datos is SELECT index_name, column_name FROM user_ind_columns WHERE table_name = p_table_name;
  6  BEGIN
  7    FOR i IN c_datos LOOP
  8      l_nombre_indice := i.index_name;
  9      l_nombre_columna := i.column_name;
  10      DBMS_OUTPUT.PUT_LINE('Nombre Índice: ' || RPAD(l_nombre_indice, 20) || 'Nombre columna: ' || RPAD(l_nombre_columna, 20));
  11    END LOOP;
  12  END MostrarDetallesIndices;
  13  /

PROCEDURE created.

Commit complete.
```

Posteriormente, lo ejecutamos y comprobamos que funciona:

```sql
SQL> exec MostrarDetallesIndices('C_RG#');

Nombre Índice: I_RG#               Nombre columna: REFGROUP

PL/SQL procedure successfully completed.

Commit complete.
```

## Postgres:

7-  Averigua si existe el concepto de segmento y el de extensión en Postgres, en qué consiste y las diferencias con los conceptos correspondientes de ORACLE.

En PostgreSQL, el concepto de segmento se refiere a una porción de una tabla o índice que es almacenada en un archivo específico en el sistema de archivos. Los segmentos son utilizados para dividir una tabla o índice en varios archivos, lo que permite un mejor manejo de la memoria y mejora el rendimiento al realizar operaciones de lectura y escritura.

La extensión en PostgreSQL es un concepto relacionado con el manejo de espacio en disco. Una extensión es un conjunto de objetos de base de datos, como tablas o vistas, que son agrupados juntos para facilitar su manejo y administración.

En Oracle, el concepto de segmento se refiere a una porción de una tabla, índice o clúster que es almacenada en una extensión de una tabla. Los segmentos son utilizados para dividir una tabla en varios archivos, lo que permite un mejor manejo de la memoria y mejora el rendimiento al realizar operaciones de lectura y escritura.

En resumen, el concepto de segmento en PostgreSQL se refiere a una división de una tabla o índice en varios archivos, mientras que en Oracle se refiere a una división de una tabla, índice o clúster en varios archivos. 

En cuanto a la extensión, en PostgreSQL es utilizado para agrupar objetos de base de datos para facilitar su manejo y administración, mientras que en Oracle se refiere a una extensión de una tabla.


## MySQL:

8- Averigua si existe el concepto de espacio de tablas en MySQL y las diferencias con los tablespaces de ORACLE.

Sí, existe el concepto de espacio de tablas en MySQL. Un espacio de tablas es un conjunto de archivos de tablas que se almacenan en el sistema de archivos del servidor. 

Los esquemas de base de datos en MySQL se dividen en espacios de tablas. Cada espacio de tablas contiene un conjunto de tablas.

En Oracle, un tablespace es una estructura lógica que se utiliza para agrupar objetos relacionales de la base de datos, como tablas y índices. Los tablespaces se crean y administran utilizando el comando "CREATE TABLESPACE" en SQL.

En resumen, ambos conceptos permiten agrupar objetos relacionales de la base de datos, pero en MySQL se llama espacio de tablas y en ORACLE se llama tablespaces.

## MongoDB:

9- Averigua si existe la posibilidad en MongoDB de decidir en qué archivo se almacena una colección.

Sí, en MongoDB existe la posibilidad de decidir en qué archivo se almacena una colección mediante el uso de shard keys y shard collections.

La característica de sharding de MongoDB permite dividir una colección en varios fragmentos llamados shards, y distribuirlos en diferentes servidores o instancias. Cada shard es una instancia independiente de MongoDB que contiene un subconjunto de los datos de la colección original.

Un shard key es un índice que se utiliza para determinar en qué shard se almacenarán los documentos de una colección. 

Por ejemplo, si se elige un shard key basado en el campo "país", los documentos con "país" = "España" se almacenarán en un shard y los documentos con "país" = "Francia" se almacenarán en otro shard.

En resumen, en MongoDB, a través del uso de shard keys y shard collections se puede decidir en qué archivo se almacena una colección, permitiendo una mejor distribución de los datos y un mejor rendimiento en grandes volúmenes de datos.