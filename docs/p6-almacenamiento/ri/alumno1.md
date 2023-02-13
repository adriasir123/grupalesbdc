# Alumno 1


1. Muestra los espacios de tablas existentes en tu base de datos y la ruta de los ficheros que los componen. ¿Están las extensiones gestionadas localmente o por diccionario?



select FILE_NAME, TABLESPACE_NAME, BLOCKS from dba_data_files UNION select FILE_NAME, TABLESPACE_NAME, BLOCKS from dba_temp_files;


2. Usa la vista del diccionario de datos v$datafile para mirar cuando fue la última vez que se ejecutó el proceso CKPT en tu base de datos.



3. Intenta crear el tablespace TS1 con un fichero de 2M en tu disco que crezca automáticamente cuando sea necesario. ¿Puedes hacer que la gestión de extensiones sea por diccionario? Averigua la razón.


4. Averigua el tamaño de un bloque de datos en tu base de datos. Cámbialo al doble del valor que tenga.


5. Realiza un procedimiento MostrarObjetosdeUsuarioenTS que reciba el nombre de un tablespace y el de un usuario y muestre qué objetos tiene el usuario en dicho tablespace y qué tamaño tiene cada uno de ellos.


6. Realiza un procedimiento llamado MostrarUsrsCuotaIlimitada que muestre los usuarios que puedan escribir de forma ilimitada en más de uno de los tablespaces que cuentan con ficheros en la unidad C: