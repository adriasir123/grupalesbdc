# Ejercicio 5

## Meted las tablas EMP y DEPT de SCOTT en un cluster

Primero creo un tablespace que nos permitirá controlar el espacio de almacenamiento que se asigna a las tablas agrupadas en el cluster.

```sql
CREATE TABLESPACE ts_empdept
  DATAFILE '/opt/oracle/oradata/ORCLCDB/ts_empdept.dbf' SIZE 64M
  AUTOEXTEND ON NEXT 64M MAXSIZE 512M;
```

!!! note "Cláusulas del Tablespace:"
    En la cláusula **DATAFILE** especificamos la ubicación del archivo de datos, ya que esto permite controlar la ubicación física de los datos y, por tanto, mejorar el rendimiento y la disponibilidad de las aplicaciones. Si no se especifica la ubicación del archivo de datos, Oracle utilizará la ubicación predeterminada.  
    La cláusula **SIZE** especifica el tamaño inicial del archivo de datos en 64 MB.  
    La cláusula **'AUTOEXTEND ON NEXT 64M MAXSIZE 512M'** indica que el archivo de datos se ampliará automáticamente en bloques de 64 MB cuando sea necesario, hasta un máximo de 512 MB.

Ahora creamos el cluster para las tablas emp y dept. Como la columna que tienen en común es DEPTNO, añadimos esta al cluster, así se alamacenarán los datos de ambas tablas en la misma página y cada valor clave del cluster solo se almacenará una vez.

```sql
CREATE CLUSTER cluster_empdept (deptno number(2))
TABLESPACE ts_empdept;
```

Creamos un índice para el cluster. Esto es obligatorio ya que si no se hiciera no se podría ejecutar ninguna sentencia DML de las tablas del cluster. El índice se utiliza para localizar una fila en el cluster. Apunta a la página asociada con cada valor de clave del cluster.

```sql
CREATE INDEX indice_cluster_empdept ON CLUSTER cluster_empdept;
```

A continuación habría que crear las tablas asociadas al cluster, pero en nustro caso ya tenemos creadas las tablas emp y dept con datos añadidos, por lo que vamos a realizar los siguientes pasos:

- Crear una nueva tabla en el cluster con la misma estructura que la tabla DEPT:

```sql
CREATE TABLE dept_cluster (
  deptno NUMBER (2),
  dname VARCHAR2 (14),
  loc VARCHAR2 (13),
  constraint pk_deptno PRIMARY KEY(deptno)
)
CLUSTER cluster_empdept (deptno);
```

- Copiar los datos de la tabla DEPT a la tabla DEPT_CLUSTER:

```sql
INSERT INTO dept_cluster SELECT * FROM dept;
```

- Hacer lo mismo con la tabla EMP.

```sql
CREATE TABLE cluster_emp (
  empno NUMBER (4),
  ename VARCHAR2 (10),
  job VARCHAR2 (9),
  mgr NUMBER (4),
  hiredate DATE,
  sal NUMBER (7,2),
  comm NUMBER (7,2),
  deptno NUMBER (2),
  constraint pk_cluster_emp PRIMARY KEY (empno),
  constraint fk_cluster_deptno FOREIGN KEY (deptno) REFERENCES dept_cluster (deptno)
)
CLUSTER cluster_empdept (deptno);

INSERT INTO cluster_emp SELECT * FROM emp;
```

- Borrar la tabla EMP y DEPT originales:

```sql
DROP TABLE emp;
DROP TABLE dept;
```

- Renombrar la tabla CLUSTER_EMP y DEPT_CLUSTER a EMP y DEPT:

```sql
ALTER TABLE dept_cluster RENAME TO dept;
ALTER TABLE cluster_emp RENAME TO emp;
```

### Comprobaciones generales

**Comprobamos que el cluster se ha creado:**

```sql
SELECT cluster_name FROM DBA_CLUSTERS;
```

![cluster](/img/capturas-arantxa/96.png)

**Comprobamos que las tablas pertenecen al cluster creado:**

```sql
SELECT table_name, cluster_name FROM USER_TABLES where table_name='DEPT' or table_name='EMP';
```

![cluster2](/img/capturas-arantxa/97.png)

**Comprobamos que podemos consultar las tablas del cluster.**

```sql
select empno,ename,emp.deptno,dname from emp,dept where emp.deptno=dept.deptno;
```

![cluster3](/img/capturas-arantxa/98.png)
