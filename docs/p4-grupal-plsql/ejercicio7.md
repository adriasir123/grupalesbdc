# Ejercicio 7

## Realiza los módulos de programación necesarios para evitar que un mismo jockey corra más de tres carreras en el mismo día.

Voy a crear un trigger que antes de insertar los datos de participación en una carrera compruebe si el jockey que se quiere insertar ha participado en más carreras ese mismo día.

```
CREATE OR REPLACE TRIGGER evitar_tres_carreras_jockey
BEFORE INSERT ON participaciones
FOR EACH ROW
DECLARE
    v_num_carreras NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_num_carreras
    FROM participaciones p, carrerasProfesionales c
    WHERE p.codigoCarrera=c.codigoCarrera
    AND p.dniJockey = :new.dniJockey
    AND TO_CHAR(c.fechaHora, 'DD-MM-YYYY') = (SELECT TO_CHAR(fechaHora, 'DD-MM-YYYY') FROM carrerasProfesionales WHERE codigoCarrera = :new.codigoCarrera);
    
    IF v_num_carreras >= 3 THEN
        RAISE_APPLICATION_ERROR(-20001, 'El jockey ya ha corrido tres carreras ese día.');
    END IF;
END;
/
```




**PRUEBA:**

Las carreras con código 10, 11 y 12 son en la misma fecha (13-11-2016), voy a añadir otra carrera que sea en esa fecha y voy a añadir las participaciones de un mismo jockey en esas carreras.

```
insert into carrerasProfesionales  values (14, to_date('13-11-2016 13:00', 'DD-MM-YYYY HH24:MI'), 6125, 450, '01-01-2016', '01-06-2016');
```

```
insert into participaciones values(10, 2, 'Y6857984L', 2, 5);
insert into participaciones values(11, 2, 'Y6857984L', 7, 4);
insert into participaciones values(12, 2, 'Y6857984L', 1, 3);
insert into participaciones values(14, 2, 'Y6857984L', 1, 1);
```

![prueba1](/img/capturas-arantxa/80.png)

```
insert all into participaciones values(10, 5, '00015258D', 3, 4) into participaciones values(11, 5, '00015258D', 5, 1) into participaciones values(12, 5, '00015258D', 2, 5) into participaciones values(14, 2, '00015258D', 1, 1) select * from dual;
```

![prueba2](/img/capturas-arantxa/81.png)

