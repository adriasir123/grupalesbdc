# Ejercicio 5

> Añade una columna a la tabla Propietarios llamada ImporteTotalPremios. Rellénala con un procedimiento o una
sentencia SQL considerando los premios que han conseguido los caballos de cada uno de ellos por sus victorias
en las carreras. Haz un trigger que la mantenga actualizada cada vez que se realice cualquier cambio en la tabla
Participaciones que afecte a este total.

## Código

```sql
CREATE OR REPLACE PROCEDURE importe_total
AS
    cursor c_propietarios is select NOMBRE from propietarios where DNI in (select dniPropietario from caballos where codigoCaballo in (select codigocaballo from participaciones where POSICIONFINAL=1));  
    v_importe NUMERIC;
BEGIN
    FOR i IN SELECT * FROM c_propietarios LOOP
        SELECT sum(IMPORTEPREMIO) INTO v_importe FROM carrerasprofesionales WHERE codigoCarrera IN (SELECT codigoCarrera FROM participaciones WHERE POSICIONFINAL=1 AND codigocaballo IN (SELECT codigocaballo FROM caballos WHERE dnipropietario IN (SELECT dni FROM propietarios WHERE nombre=i.nombre)));
        UPDATE propietarios SET IMPORTETOTALPREMIOS=v_importe WHERE nombre=i.nombre;
    END LOOP;
END importe_total;

CREATE TRIGGER nueva_carrera
AFTER INSERT OR UPDATE OR DELETE ON PARTICIPACIONES
FOR EACH ROW
BEGIN
    importe_total;
END;
```

## Prueba Funcionamiento

- Probamos el primer procedimiento:

```sql
hipodromo=# call importe_total;

Query returned successfully in 85 msec .

hipodromo=# select *
hipodromo-# from propietarios;
    dni     | nombre  | apellido1 | apellido2 | cuotamensual | importetotalpremios 
------------+---------+-----------+-----------+--------------+---------------------
 21913124n  | Horacio | Arreguy   | Fazzio    |       250.00 |              16607                    
 Z7782152S  | Robert  | Chase     |           |       300.00 |              8750   
 83069279H  | Lucas   | Martínez  | Muñoz     |       200.70 |              7000     
 X58056225B | Gustavo | Scarpello |           |       100.00 |              7000     
(4 filas)
```

- Instertamos nueva información en las tablas carrerasProfesionales y participaciones para comprobar si el trigger funciona correctamente:

```sql

hipodromo=# insert into carrerasProfesionales values (13, TO_TIMESTAMP('20-11-2016 13:00', 'DD-MM-YYYY HH24:MI'), 8000, 650, TO_DATE('01-01-2010', 'DD-MM-YYYY'), TO_DATE('01-06-2013', 'DD-MM-YYYY'));
INSERT 0 1

hipodromo=# insert into participaciones values(13, 7, 'X6785562X', 1, 1);
INSERT 0 1

hipodromo=# insert into participaciones values(13, 4, '85108890N', 5, 5);
INSERT 0 1

hipodromo=# select *
hipodromo-# from propietarios;
    dni     | nombre  | apellido1 | apellido2 | cuotamensual | importetotalpremios 
------------+---------+-----------+-----------+--------------+---------------------
 21913124n  | Horacio | Arreguy   | Fazzio    |       250.00 |              16607                    
 Z7782152S  | Robert  | Chase     |           |       300.00 |              8750   
 83069279H  | Lucas   | Martínez  | Muñoz     |       200.70 |              15000    
 X58056225B | Gustavo | Scarpello |           |       100.00 |              7000     
(4 filas)
```