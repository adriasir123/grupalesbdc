# Ejercicio 5

> Añade una columna a la tabla Propietarios llamada ImporteTotalPremios. Rellénala con un procedimiento o una sentencia SQL considerando los premios que han conseguido los caballos de cada uno de ellos por sus victorias en las carreras. Haz un trigger que la mantenga actualizada cada vez que se realice cualquier cambio en la tabla Participaciones que afecte a este total.

## Código

```sql
create or replace procedure importe_total
is
    cursor c_propietarios is select NOMBRE from propietarios where DNI in (select dniPropietario from caballos where codigoCaballo in (select codigocaballo from participaciones where POSICIONFINAL=1));  
    v_importe number;
begin
    for i in c_propietarios loop
        select sum(IMPORTEPREMIO) into v_importe from carrerasprofesionales where codigoCarrera in (select codigoCarrera from participaciones where POSICIONFINAL=1 and codigocaballo in (select codigocaballo from caballos where dnipropietario in (select dni from propietarios where nombre=i.nombre)));
        update propietarios set IMPORTETOTALPREMIOS=v_importe where nombre=i.nombre;
    end loop;
end importe_total;
/
 
CREATE OR REPLACE TRIGGER nueva_carrera
before insert or update on PARTICIPACIONES
BEGIN
    importe_total;
end;
/
```

## Prueba de funcionamiento

- Probamos el primer procedimiento:

    ```sql
    SQL> exec importe_total;

    Procedimiento PL/SQL terminado correctamente.

    SQL> select *
    2  from propietarios;

    DNI	       NOMBRE          APELLIDO1       APELLIDO2	   CUOTAMENSUAL IMPORTETOTALPREMIOS
    ---------- --------------- --------------- --------------- ------------ -------------------

    21913124n  Horacio	       Arreguy	       Fazzio		   250          16607
            
    Z7782152S  Robert	       Chase				           300          8750
            
    83069279H  Lucas	       Martinez	       Munoz		   200,7        7000
            
    X58056225B Gustavo	       Scarpello				       100          7000
    ```

- Instertamos nueva información en las tablas carrerasProfesionales y participaciones para comprobar si el trigger funciona correctamente:

```sql

SQL> insert into carrerasProfesionales values (13, to_date('20-11-2016 13:00' , 'DD-MM-YYYY HH24:MI'), 8000, 650, to_date('01-01-2010', 'DD-MM-YYYY'), to_date('01-06-2013', 'DD-MM-YYYY'));

1 fila creada.

SQL> insert into participaciones values(13, 7, 'X6785562X', 1, 1);

1 fila creada.

SQL> insert into participaciones values(13, 4, '85108890N', 5, 5);

1 fila creada.

DNI	       NOMBRE          APELLIDO1       APELLIDO2	   CUOTAMENSUAL IMPORTETOTALPREMIOS
---------- --------------- --------------- --------------- ------------ -------------------

21913124n  Horacio	       Arreguy	       Fazzio		   250          16607
	      
Z7782152S  Robert	       Chase				           300          8750
	       
83069279H  Lucas	       Martinez	       Munoz		   200,7        15000
	       
X58056225B Gustavo	       Scarpello				       100          7000

```
