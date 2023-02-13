# Ejercicio 2

Realizad una consulta al diccionario de datos que muestre qué índices existen para objetos pertenecientes al esquema de SCOTT y sobre qué columnas están definidos. Averiguad en qué fichero o ficheros de datos se encuentran las extensiones de sus segmentos correspondientes.

Para mostrar los índices existentes para objetos pertenecientes al esquema de SCOTT y sobre qué columnas están definidos, se puede utilizar la siguiente consulta SQL:

```sql
SQL> SELECT i.index_name, i.table_name, i.uniqueness, c.column_name
FROM dba_indexes i
JOIN dba_ind_columns c ON i.index_name = c.index_name
WHERE i.owner = 'SCOTT'; 

INDEX_NAME
--------------------------------------------------------------------------------
TABLE_NAME
--------------------------------------------------------------------------------
UNIQUENES
---------
COLUMN_NAME
--------------------------------------------------------------------------------
PK_DEPTNO
DEPT
UNIQUE
DEPTNO

INDEX_NAME
--------------------------------------------------------------------------------
TABLE_NAME
--------------------------------------------------------------------------------
UNIQUENES
---------
COLUMN_NAME
--------------------------------------------------------------------------------
PK_EMPNO
EMP
UNIQUE
EMPNO
```

La consulta anterior utiliza las vistas del diccionario de datos DBA_INDEXES y DBA_IND_COLUMNS para obtener información sobre los índices y columnas de los objetos pertenecientes al esquema de SCOTT. 

La columna "index_name" contiene el nombre del índice, "table_name" contiene el nombre de la tabla a la que pertenece el índice, "uniqueness" indica si el índice es único o no, y "column_name" contiene el nombre de la columna sobre la que está definido el índice.

Para averiguar en qué fichero o ficheros de datos se encuentran las extensiones de los segmentos correspondientes a los índices, se puede utilizar la siguiente consulta SQL:

```sql
SQL> SELECT i.index_name, s.tablespace_name
FROM dba_segments s
JOIN dba_indexes i ON s.segment_name = i.index_name
WHERE i.owner = 'SCOTT';

INDEX_NAME
--------------------------------------------------------------------------------
TABLESPACE_NAME
------------------------------
PK_EMPNO
USERS

PK_DEPTNO
USERS
```

¿Cómo averiguo en que fichero o ficheros de datos se encuentran las extensiones de los segmentos correspondientes a los índices?

La consulta anterior utiliza las vistas del diccionario de datos DBA_SEGMENTS y DBA_INDEXES para obtener información sobre los segmentos y índices pertenecientes al esquema de SCOTT. 

La columna "index_name" contiene el nombre del índice, y "file_name" contiene el nombre del fichero de datos en el que se encuentra el segmento correspondiente al índice.