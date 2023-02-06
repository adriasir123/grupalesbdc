# Parte A

Pepe es el administrador de la base de datos.
Juan y Clara son los programadores de la base de datos, que trabajan tanto en la aplicación que usa el departamento de Ventas como en la usada por el departamento de Producción.
Ana y Eva tienen permisos para insertar, modificar y borrar registros en las tablas de la aplicación de Ventas que tienes que crear, y se llaman Productos y Ventas, siendo propiedad de Ana.
Jaime y Lidia pueden leer la información de esas tablas pero no pueden modificar la información.
Crea los usuarios y dale los roles y permisos que creas conveniente.

## Creación de usuarios

```sql
CREATE USER pepe IDENTIFIED BY pepe;
CREATE USER juan IDENTIFIED BY juan;
CREATE USER clara IDENTIFIED BY clara;
```

```sql
grant dba to pepe;
grant resource to juan;
grant resource to clara;
```

**grant dba** se utiliza para dar otorgar permisos de administrador de base de datos a un usuario, es un permiso extremadamente poderoso que solo pocos usuarios deben tener.

**grant resource** es un rol predefinido de Oracle que consiste en otorgar el uso de recursos de la base de datos, como por ejemplo la creación de tablas, vistas, ejecución de procedimientos o creación de los mismos siempre y cuando tenga derechos sobre esas tablas y no sobre los procedimientos del sistema.

create user Ana identified by Ana;
grant create session to Ana;
create user Eva identified by Eva;


```sql
CREATE TABLE Ana.ventas (
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
```

create tablespace Ana datafile /opt/oracle/oradata/ORCLCDB/ size 50MB;