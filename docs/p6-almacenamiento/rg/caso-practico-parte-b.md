# Parte B

b) Los espacios de tablas son System, Producción (ficheros prod1.dbf y prod2.dbf) y Ventas (fichero vent.dbf). 
Los programadores del departamento de Informática pueden crear objetos en cualquier tablespace de la base de datos, excepto en System.
Los demás usuarios solo podrán crear objetos en su tablespace correspondiente teniendo un límite de espacio de 30 M los del departamento de Ventas y 100K los del de Producción.
Pepe tiene cuota ilimitada en todos los espacios, aunque el suyo por defecto es System.

Primero vamos a crear los tablespaces:

```sql
CREATE TABLESPACE ts_produccion
DATAFILE 'prod1.dbf' SIZE 100M,
'prod2.dbf' SIZE 100M;

CREATE TABLESPACE ts_venta
DATAFILE 'vent.dbf'
SIZE 100M;
```

```sql
ALTER USER ANA QUOTA 30M ON ts_venta;
ALTER USER EVA QUOTA 30M ON ts_venta;
ALTER USER JAIME QUOTA 100K ON ts_produccion;
ALTER USER LIDIA QUOTA 100K ON ts_produccion;
```

```sql
ALTER USER Pepe DEFAULT TABLESPACE SYSTEM;
GRANT UNLIMITED TABLESPACE TO Pepe;
```

```sql
GRANT UNLIMITED TABLESPACE TO JUAN;
GRANT UNLIMITED TABLESPACE TO CLARA;
ALTER USER JUAN QUOTA 0 ON SYSTEM;
ALTER USER CLARA QUOTA 0 ON SYSTEM;
```
