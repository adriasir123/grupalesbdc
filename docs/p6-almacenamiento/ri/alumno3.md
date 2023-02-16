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

Podemos comprobar que TS1 está utilizando el fichero ts1.dbf (con id 20) y que su tamaño en K es de 64.

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

Podemos comprobar que en TS2 el datafile utilizado para la tabla employees es ts2_1.dbf y su tamaño de 64K.

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

También podemos verlo de la siguiente forma:

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

## MySQL:

### 8. Averigua si existe el concepto de extensión en MySQL y si coincide con el existente en ORACLE.

## MongoDB:

### 9. Averigua si en MongoDB puede saberse el espacio disponible para almacenar nuevos documentos.
