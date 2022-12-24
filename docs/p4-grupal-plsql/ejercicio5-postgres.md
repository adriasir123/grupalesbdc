# Ejercicio 5 PostgreSQL

> Añade una columna a la tabla Propietarios llamada ImporteTotalPremios. Rellénala con un procedimiento o una sentencia SQL considerando los premios que han conseguido los caballos de cada uno de ellos por sus victorias en las carreras. Haz un trigger que la mantenga actualizada cada vez que se realice cualquier cambio en la tabla Participaciones que afecte a este total.

## Pasos previos

Hay que añadir la nueva columna a propietarios:

```sql
ALTER TABLE propietarios ADD COLUMN IMPORTETOTALPREMIOS NUMERIC;
```

## Código

```sql
CREATE OR REPLACE PROCEDURE pr_importe_total()
AS
$$
DECLARE
    c_propietarios CURSOR FOR
        SELECT nombre
        FROM propietarios
        WHERE dni IN (
                SELECT dnipropietario
                FROM caballos
                WHERE codigocaballo IN (
                        SELECT codigocaballo
                        FROM participaciones
                        WHERE posicionfinal=1
                    )
            );
    v_importe NUMERIC;
BEGIN
    FOR i IN c_propietarios LOOP

        SELECT SUM(importepremio) INTO v_importe
        FROM carrerasprofesionales
        WHERE codigocarrera IN (
                SELECT codigocarrera
                FROM participaciones
                WHERE posicionfinal=1 AND codigocaballo IN (
                        SELECT codigocaballo
                        FROM caballos
                        WHERE dnipropietario IN (
                                SELECT dni
                                FROM propietarios
                                WHERE nombre=i.nombre
                            )
                    )
            );

        UPDATE propietarios
        SET
            importetotalpremios=v_importe
        WHERE
            nombre=i.nombre;

    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

```sql
CREATE OR REPLACE FUNCTION fn_nueva_carrera()
RETURNS TRIGGER AS $$
BEGIN
    CALL pr_importe_total();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

```sql
CREATE TRIGGER tr_nueva_carrera
    AFTER INSERT OR UPDATE OR DELETE ON participaciones
    FOR EACH ROW
EXECUTE FUNCTION fn_nueva_carrera();
```

## Pruebas de funcionamiento

- Probamos el primer procedimiento:

    ```sql
    hipodromo=> call pr_importe_total();
    CALL
    hipodromo=> select * from propietarios;
        dni     | nombre  | apellido1 | apellido2 | cuotamensual | importetotalpremios
    ------------+---------+-----------+-----------+--------------+---------------------
    21913124n  | Horacio | Arreguy   | Fazzio    |       250.00 |            16607.00
    Z7782152S  | Robert  | Chase     |           |       300.00 |             8750.00
    83069279H  | Lucas   | Martínez  | Muñoz     |       200.70 |             7000.00
    X58056225B | Gustavo | Scarpello |           |       100.00 |             7000.00
    (4 rows)
    ```

- Instertamos nueva información en las tablas carrerasProfesionales y participaciones para comprobar si el trigger funciona correctamente:

    ```sql
    hipodromo=> insert into carrerasProfesionales values (13, TO_TIMESTAMP('20-11-2016 13:00', 'DD-MM-YYYY HH24:MI'), 8000, 650, TO_DATE('01-01-2010', 'DD-MM-YYYY'), TO_DATE('01-06-2013', 'DD-MM-YYYY'));
    INSERT 0 1
    hipodromo=> insert into participaciones values(13, 7, 'X6785562X', 1, 1);
    INSERT 0 1
    hipodromo=> insert into participaciones values(13, 4, '85108890N', 5, 5);
    INSERT 0 1
    hipodromo=> select * from propietarios;
        dni     | nombre  | apellido1 | apellido2 | cuotamensual | importetotalpremios
    ------------+---------+-----------+-----------+--------------+---------------------
    21913124n  | Horacio | Arreguy   | Fazzio    |       250.00 |            16607.00
    Z7782152S  | Robert  | Chase     |           |       300.00 |             8750.00
    83069279H  | Lucas   | Martínez  | Muñoz     |       200.70 |            15000.00
    X58056225B | Gustavo | Scarpello |           |       100.00 |             7000.00
    (4 rows)
    ```
