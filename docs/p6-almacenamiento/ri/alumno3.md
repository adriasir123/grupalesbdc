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
