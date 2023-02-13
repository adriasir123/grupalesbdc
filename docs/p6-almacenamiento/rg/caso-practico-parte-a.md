# Parte A

Pepe es el administrador de la base de datos.
Juan y Clara son los programadores de la base de datos, que trabajan tanto en la aplicación que usa el departamento de Ventas como en la usada por el departamento de Producción.
Ana y Eva tienen permisos para insertar, modificar y borrar registros en las tablas de la aplicación de Ventas que tienes que crear, y se llaman Productos y Ventas, siendo propiedad de Ana.
Jaime y Lidia pueden leer la información de esas tablas pero no pueden modificar la información.
Crea los usuarios y dale los roles y permisos que creas conveniente.

## Creación de usuarios

Vamos a crear los usuarios y a continuación vamos a darles los permisos de conexión a la base de datos:

```sql
CREATE USER pepe IDENTIFIED BY pepe;
grant create session to pepe;
CREATE USER juan IDENTIFIED BY juan;
grant create session to juan;
CREATE USER clara IDENTIFIED BY clara;
grant create session to clara;
```

Ahora le vamos a dar los derechos del sistema a pepe y los recursos de creación de plsql a juan y clara:

```sql
grant dba to pepe;
grant resource to juan;
grant resource to clara;
```

**grant dba** se utiliza para dar otorgar permisos de administrador de base de datos a un usuario, es un permiso extremadamente poderoso que solo pocos usuarios deben tener.

**grant resource** es un rol predefinido de Oracle que consiste en otorgar el uso de recursos de la base de datos, como por ejemplo la creación de tablas, vistas, ejecución de procedimientos o creación de los mismos siempre y cuando tenga derechos sobre esas tablas y no sobre los procedimientos del sistema.



A continuación craemos los usuarios de Ana y Eva:
  
```sql

create user Ana identified by Ana;
grant create session to Ana;
create user Eva identified by Eva;
grant create session to Eva;

```

Añadimos las tablas junto con los siguientes datos:

```sql
CREATE TABLE Ana.ventas (
  id NUMBER PRIMARY KEY,
  fecha DATE,
  producto VARCHAR2(50),
  cantidad NUMBER,
  precio NUMBER
);

CREATE TABLE Ana.productos (
  id NUMBER PRIMARY KEY,
  fecha DATE,
  producto VARCHAR2(50),
  cantidad NUMBER,
  precio NUMBER
);
```

```sql
INSERT INTO Ana.ventas (id, fecha, producto, cantidad, precio)
VALUES (1, '01-FEB-2023', 'Computadora', 2, 800);

INSERT INTO Ana.ventas (id, fecha, producto, cantidad, precio)
VALUES (2, '01-FEB-2023', 'Impresora', 3, 200);

INSERT INTO Ana.ventas (id, fecha, producto, cantidad, precio)
VALUES (3, '02-FEB-2023', 'Teclado', 5, 30);

INSERT INTO Ana.productos (id, fecha, producto, cantidad, precio)
VALUES (1, '01-FEB-2023', 'Computadora', 2, 800);

INSERT INTO Ana.productos (id, fecha, producto, cantidad, precio)
VALUES (2, '01-FEB-2023', 'Impresora', 3, 200);

INSERT INTO Ana.productos (id, fecha, producto, cantidad, precio)
VALUES (3, '02-FEB-2023', 'Teclado', 5, 30);
```

Creamos los tablespaces de ambas tablas, porque si no tenemos espacios no pueden haber registros en las tablas:

```sql
CREATE TABLESPACE ventas
DATAFILE 'ventas.dbf'
SIZE 50M;

CREATE TABLESPACE productos
DATAFILE 'productos.dbf'
SIZE 50M;
```

Ahora vamos a darle los permisos de ventas y productos a Ana:

```sql
ALTER USER ana DEFAULT TABLESPACE ventas;
ALTER USER ana quota 2M on ventas;
ALTER USER ana quota 2M on productos;
```

Ahora crearemos al usuario eva, la cual tendrá cuotas en ambas tablas para hacer select, insert, update y delete:

```sql
CREATE USER Eva identified by eva;
ALTER USER Eva quota 2M on ventas;
ALTER USER Eva quota 2M on productos;
grant create session to eva;
```

Vamos a crear el rol de ventasyproductos, que tendrá permisos de select, insert, update y delete en ambas tablas:

```sql
CREATE ROLE ventasyproductos;
GRANT select, insert, update, delete on Ana.Ventas to ventasyproductos;
GRANT select, insert, update, delete on Ana.Ventas to ventasyproductos;
GRANT select, insert, update, delete on Ana.productos to ventasyproductos;
GRANT select, insert, update, delete on Ana.productos to ventasyproductos;
```

```sql
GRANT ventasyproductos to eva;
GRANT ventasyproductos to juan;
GRANT ventasyproductos to clara;
```

Ahora vamos a crear los usuarios de Jaime y Lidia, que solo podrán hacer select:

```sql
CREATE USER Jaime identified by Jaime;
ALTER USER Jaime quota 2M on ventas;
ALTER USER Jaime quota 2M on productos;
grant create session to Jaime;

CREATE USER Lidia identified by Lidia;
ALTER USER Lidia quota 2M on ventas;
ALTER USER Lidia quota 2M on productos;
grant create session to Lidia;
```

```sql
CREATE ROLE verventasyproductos;
GRANT select on Ana.Ventas to ventasyproductos;
GRANT select on Ana.Ventas to ventasyproductos;
GRANT select on Ana.productos to ventasyproductos;
GRANT select on Ana.productos to ventasyproductos;
```

Por último vamos a dar los permisos del rol a Jaime y Lidia:

```sql
GRANT verventasyproductos to Jaime;
GRANT verventasyproductos to Lidia;
```

Y con esto doy pro finalizado el ejercicio.