# Ejercicio 3

> Crear una secuencia para rellenar el campo deptno de la tabla dept de forma coherente con los datos ya existentes

```sql
CREATE SEQUENCE deptno_seq
    INCREMENT BY 10
    START WITH 50;
```

> Insertar al menos dos registros haciendo uso de la secuencia

```sql
INSERT INTO SCOTT.DEPT VALUES (deptno_seq.NEXTVAL,'MARKETING','DOS HERMANAS');

1 row created.

INSERT INTO SCOTT.DEPT VALUES (deptno_seq.NEXTVAL,'FINANCE','UTRERA');

1 row created.
```

> Comprobar que los valores en `deptno` son correctos

```sql
select * from scott.dept;

    DEPTNO DNAME          LOC
---------- -------------- -------------
        10 ACCOUNTING     NEW YORK
        20 RESEARCH       DALLAS
        30 SALES          CHICAGO
        40 OPERATIONS     BOSTON
        50 MARKETING      DOS HERMANAS
        60 FINANCE        UTRERA
```
