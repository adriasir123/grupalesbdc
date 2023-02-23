# Alumno 3

## ORACLE:

### 1. Muestra los objetos a los que pertenecen las extensiones del tablespace TS2 (creado por Alumno 2) y el tamaño de cada una de ellas.

```sql
SELECT segment_name, segment_type, extent_id, bytes
FROM dba_extents
WHERE tablespace_name = 'TS2';
```

![ts2_objetos](/img/capturas-arantxa/103.png)

### 2. Borra la tabla que está llenando TS2 consiguiendo que vuelvan a existir extensiones libres. Añade después otro fichero de datos a TS2.

Utilizamos TRUNCATE para borrar los datos de la tabla employee pero mantener su estructura.

```sql
truncate table employees;
```

Añadimos otro dichero de datos a TS2.

```sql
alter tablespace ts2 add datafile '/home/usuario/ts23/ts2_3.dbf' size 1M;
```

![ts2_datafile](/img/capturas-arantxa/104.png)

### 3. Crea el tablespace TS3 gestionado localmente con un tamaño de extension uniforme de 128K y un fichero de datos asociado. Cambia la ubicación del fichero de datos y modifica la base de datos para que pueda acceder al mismo. Crea en TS3 dos tablas e inserta registros en las mismas. Comprueba que segmentos tiene TS3, qué extensiones tiene cada uno de ellos y en qué ficheros se encuentran.

En el home de mi usuario:

```bash
mkdir ts23
sudo chown oracle: ts23/
```

Creamos el tablespace TS3.

```sql
create tablespace ts3 datafile '/home/usuario/ts23/ts3.dbf' size 1M extent management local uniform size 128K;
```

Ponemos el tablespaces offline:

```sql
ALTER TABLESPACE ts3 OFFLINE NORMAL;
```

Cambiamos la ubicación del fichero de datos desde la terminal a la carpeta ts21.

```bash
sudo mv ts23/ts3.dbf ts22/
```

Usamos ALTER TABLESPACE con RENAME DATAFILE para cambiar la ubicación en la que se encuentra el fichero de datos.

```sql
alter tablespace ts3
rename datafile '/home/usuario/ts23/ts3.dbf'
to '/home/usuario/ts22/ts3.dbf';
```

Activamos el tablespace.

```sql
ALTER TABLESPACE ts3 ONLINE;
```

Creamos dos tablas e insertamos registros.

```sql
create table tabla1
  (codigo varchar2(15),
  nombre varchar2(15))
  tablespace ts3;

create table tabla2
  (codigo varchar2(15),
  nombre varchar2(15))
  tablespace ts3;

insert into tabla1 values ('A1234', 'Pepito');
insert into tabla2 values ('B5678', 'Camionero');
```

Comprobamos los segmentos que tiene TS3 y las extensiones que tiene cada uno de ellos y en qué ficheros se encuentran.

```sql
--Comprobar segmentos
select owner, segment_name, segment_type, bytes from dba_segments
where tablespace_name = 'TS3';
```

![segmentos](/img/capturas-arantxa/105.png)

```sql
--Comprobar extensiones de esos segmentos
select segment_name, extent_id, bytes, file_id from dba_extents
where segment_name = 'TABLA1' or segment_name = 'TABLA2';
```

![extensiones](/img/capturas-arantxa/106.png)

```sql
--Comprobar fichero donde se encuentra 
select file_name from dba_data_files where file_id=19;
```

![fichero](/img/capturas-arantxa/107.png)

### 4. Redimensiona los ficheros asociados a los tres tablespaces que has creado de forma que ocupen el mínimo espacio posible para alojar sus objetos.

Comprobar el espacio utilizado por los datafile de los tablespace.

```sql
--TS1
select segment_name, bytes from dba_segments
where tablespace_name = 'TS1';

SEGMENT_NAME
--------------------------------------------------------------------------------
     BYTES
----------
LIBROS
     65536

select segment_name, extent_id, bytes/1024 as tamano_K, file_id from dba_extents
where segment_name = 'LIBROS';

SEGMENT_NAME
--------------------------------------------------------------------------------
 EXTENT_ID   TAMANO_K	 FILE_ID
---------- ---------- ----------
LIBROS
	 0	   64	      20


select file_name from dba_data_files where file_id=20;

FILE_NAME
--------------------------------------------------------------------------------
/home/usuario/ts11/ts1.dbf
```

Podemos comprobar que TS1 está utilizando el fichero ts1.dbf (con id 20) y que su tamaño en K es de 64. Vamos ahora con TS2.

```sql
--TS2
select segment_name, bytes from dba_segments
where tablespace_name = 'TS2';

SEGMENT_NAME
--------------------------------------------------------------------------------
     BYTES
----------
EMPLOYEES
     65536


select segment_name, extent_id, bytes/1024 as tamano_K, file_id from dba_extents
where segment_name = 'EMPLOYEES';

SEGMENT_NAME
--------------------------------------------------------------------------------
 EXTENT_ID	TAMANO_K	 FILE_ID
---------- ---------- ----------
EMPLOYEES
	 0			64	      16


select file_name from dba_data_files where file_id=16;
FILE_NAME
--------------------------------------------------------------------------------
/home/usuario/ts21/ts2_1.dbf
```

Podemos comprobar que en TS2 el datafile utilizado para la tabla employees es ts2_1.dbf y su tamaño de 64K. Por último, vemos TS3.

```sql
--TS3
--En el ejercicio anterior pudimos comprobar que el datafile es ts3.dbf y que tiene un tamaño de 128K cada segmento
select segment_name, bytes from dba_segments
where tablespace_name = 'TS3';

SEGMENT_NAME
--------------------------------------------------------------------------------
     BYTES
----------
TABLA1
    131072

TABLA2
    131072


select segment_name, extent_id, bytes/1024 as tamano_K, file_id from dba_extents
where segment_name = 'TABLA1' or segment_name = 'TABLA2';

SEGMENT_NAME
--------------------------------------------------------------------------------
 EXTENT_ID   TAMANO_K	 FILE_ID
---------- ---------- ----------
TABLA1
	 0	  128	      19

TABLA2
	 0	  128	      19


select file_name from dba_data_files where file_id=19;

FILE_NAME
--------------------------------------------------------------------------------
/home/usuario/ts22/ts3.dbf
```

Otra forma de calcularlo más rápidamente es como se muestra a continuación:

```sql
SELECT tablespace_name, SUM(bytes)/1024 AS size_mb
FROM dba_segments where tablespace_name='TS1' or tablespace_name='TS2' or tablespace_name='TS3'
GROUP BY tablespace_name;

TABLESPACE_NAME      	  SIZE_MB
----------------------- ----------
TS1				             64
TS3				            256
TS2				             64
```

Por tanto el resultado sería:

- TS1 (/home/usuario/ts11/ts1.dbf): 64K

- TS2 (/home/usuario/ts21/ts2_1.dbf): 64K

- TS3 (/home/usuario/ts22/ts3.dbf): 256K

Así pues redimensionamos los datafile a esos tamaños para tener el mínimo espacio posible para alojar sus objetos.

```sql
--Quitar solo lectura del fichero de TS1
ALTER TABLESPACE TS1 READ WRITE;
--Redimensionar fichero de TS1
alter database datafile '/home/usuario/ts11/ts1.dbf' resize 64K;
--Redimensionar fichero de TS2
alter database datafile '/home/usuario/ts21/ts2_1.dbf' resize 64K;
--Redimensionar fichero de TS3
alter database datafile '/home/usuario/ts22/ts3.dbf' resize 256K;
```

### 5. Realiza un procedimiento llamado InformeRestricciones que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna o columnas afectan y en qué consisten exactamente.

```sql
CREATE OR REPLACE PROCEDURE InformeRestricciones (p_tabla user_constraints.table_name%TYPE) IS
BEGIN
  FOR i IN (SELECT c.constraint_name, c.constraint_type, c.table_name, cc.column_name
            FROM user_constraints c, user_cons_columns cc
            WHERE c.constraint_name = cc.constraint_name
            AND c.table_name = p_tabla)
  LOOP
      DBMS_OUTPUT.PUT_LINE('------------------------------------------------------------');
      DBMS_OUTPUT.PUT_LINE('RESTRICCION: ' || i.constraint_name||CHR(10)||'El tipo de restriccion es "' || i.constraint_type||'".'||CHR(10)||'- Pertenece a la tabla "'|| i.table_name||'" y afecta a la columna "' || i.column_name||'".');
  END LOOP;
  DBMS_OUTPUT.PUT_LINE('------------------------------------------------------------');
  DBMS_OUTPUT.PUT_LINE('Los tipos de restricciones son los siguientes:'||CHR(10)||CHR(9)||'C: Check constraint'||CHR(10)||CHR(9)||'F: Foreign key constraint'||CHR(10)||CHR(9)||'P: Primary key constraint'||CHR(10)||CHR(9)||'R: Referential integrity constraint (equivalente a una restriccion foreign_key)'||CHR(10)||CHR(9)||'U: Unique key constraint');
END;
/
```

Comprobación:

```sql
exec InformeRestricciones('EMP');

------------------------------------------------------------
RESTRICCION: FK_CLUSTER_DEPTNO
El tipo de restriccion es "R".
- Pertenece a la tabla "EMP" y afecta a la columna "DEPTNO".
------------------------------------------------------------
RESTRICCION: PK_CLUSTER_EMP
El tipo de restriccion es "P".
- Pertenece a la tabla "EMP" y afecta a la columna "EMPNO".
------------------------------------------------------------
Los tipos de restricciones son los siguientes:
	C: Check constraint
	F: Foreign key constraint
	P: Primary key constraint
	R: Referential integrity constraint (equivalente a una restriccion foreign_key)
	U: Unique key constraint

Procedimiento PL/SQL terminado correctamente.
```

### 6. Realiza un procedimiento llamado MostrarAlmacenamientoUsuario que reciba el nombre de un usuario y devuelva el espacio que ocupan sus objetos agrupando por dispositivos y archivos:

```txt
				Usuario: NombreUsuario

					Dispositivo:xxxx

						Archivo: xxxxxxx.xxx

								Tabla1......nnn K
								…
								TablaN......nnn K
								Indice1.....nnn K
								…
								IndiceN.....nnn K

						Total Espacio en Archivo xxxxxxx.xxx: nnnnn K

						Archivo:...
						…

				
					
					Total Espacio en Dispositivo xxxx: nnnnnn K

					Dispositivo: yyyy
					…

				Total Espacio Usuario en la BD: nnnnnnn K
```


## Postgres:

### 7. Averigua si es posible establecer cuotas de uso sobre los tablespaces en Postgres.

PostgreSQL permite crear tablespaces pero de una forma más sencilla que en Oracle o MySQL. Muchas de las opciones que encontramos en Oracle a la hora de crear los tablespaces no existen en PostgreSQL. Por ejemplo en PostgreSQL no podemos establecer un tamaño máximo al datafile. No tendremos ninguna opción para asignar cuotas de almacenamiento. La única forma de controlar el espacio es utilizando algunas funciones de PostgreSQL para comprobar el tamaño ocupado por objetos de la base de datos. Por ejemplo, la función pg_total_relation_size() nos dirá el tamaño que ocupa una tabla de la base de datos.

```sql
--Ver el tamaño de la tabla película con la función pg_total_relation_size() y con la función pg_size_pretty() para que se vea el resultado mejorado en KB

maravilla=# SELECT pg_size_pretty(pg_total_relation_size('pelicula'));
 pg_size_pretty 
----------------
 24 kB
(1 fila)
```

## MySQL:

### 8. Averigua si existe el concepto de extensión en MySQL y si coincide con el existente en ORACLE.

Sí existe el concepto de extensión en MySQL. En MySQL el término "extent" se utiliza en el contexto de InnoDB para referirse a un grupo de páginas dentro de un tablespace. Cada extent en InnoDB tiene un tamaño fijo y contiene un número fijo de páginas, según la configuración de tamaño de página de InnoDB y el tamaño de la extensión.

En InnoDB, el tamaño de la extensión se define como 1 MB para los tamaños de página de 4 KB, 8 KB y 16 KB, y aumenta a medida que aumenta el tamaño de la página. Por ejemplo, para un tamaño de página de 32 KB, el tamaño de la extensión es de 2 MB, y para un tamaño de página de 64 KB, el tamaño de la extensión es de 4 MB.

Las operaciones de E/S en InnoDB están diseñadas para trabajar con extensiones, lo que significa que se leen, escriben, asignan o liberan datos una extensión a la vez. Esto mejora la eficiencia de E/S y reduce la fragmentación del tablespace.

MySQL no utiliza el término "extent" en el mismo contexto que Oracle. En Oracle, el término "extent" se refiere a una porción de un tablespace que se utiliza para almacenar segmentos de la base de datos, como tablas, índices y otros objetos. En Oracle, cada extent tiene un tamaño fijo y se utiliza para almacenar un segmento específico. Cuando un segmento de la base de datos crece y no hay más espacio disponible en su extent actual, se asigna un nuevo extent para el segmento.

Por tanto, aunque ambos conceptos se refieren a porciones de almacenamiento utilizadas para almacenar objetos de la base de datos, la implementación de los términos "extent" en MySQL y Oracle son diferentes.

## MongoDB:

### 9. Averigua si en MongoDB puede saberse el espacio disponible para almacenar nuevos documentos.

Para obtener información sobre el espacio disponible en un servidor MongoDB podemos utilizar el comando db.stats(). Este comando devuelve una serie de estadísticas sobre una base de datos, incluyendo el tamaño total de la base de datos, el espacio utilizado por la base de datos, y el espacio disponible para la base de datos.

Se pueden utilizar los parámetros **scale** y **freeStorage** para ver, respectivamente, los resultados en KB (poniendo el valor en 1024) y el espacio libre para las colecciones de la base de datos (poniendo el valor en 1).

```sql
maravilla> db.stats( { freeStorage: 1, scale: 1024 } )

{
  db: 'maravilla',
  collections: 5,
  views: 0,
  objects: 19,
  avgObjSize: 73.21052631578948,
  dataSize: 1.3583984375,
  storageSize: 68,
  freeStorageSize: 0,
  indexes: 5,
  indexSize: 68,
  indexFreeStorageSize: 0,
  totalSize: 136,
  totalFreeStorageSize: 0,
  scaleFactor: 1024,
  fsUsedSize: 34825072,
  fsTotalSize: 39987708,
  ok: 1
}
```
