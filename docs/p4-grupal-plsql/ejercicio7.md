# Ejercicio 7

> Realiza los módulos de programación necesarios para evitar que un mismo jockey corra más de tres carreras en el mismo día.

En un primer momento hice este trigger, pero éste no contempla el error de mutación de la tabla al hacer un insert all. Si se hacen los inserts uno a uno sí funciona.

```sql
CREATE OR REPLACE TRIGGER evitar_tres_carreras_jockey
    BEFORE INSERT OR UPDATE ON participaciones
    FOR EACH ROW
DECLARE
    v_num_carreras NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_num_carreras
    FROM participaciones p, carrerasprofesionales c
    WHERE p.codigocarrera=c.codigocarrera
        AND p.dnijockey = :new.dnijockey
        AND to_char(c.fechahora, 'DD-MM-YYYY') = (
            SELECT to_char(fechahora, 'DD-MM-YYYY')
            FROM carrerasprofesionales
            WHERE codigocarrera = :new.codigocarrera
        );
        
    IF v_num_carreras >= 3 THEN
        raise_application_error(-20001, 'El jockey ya ha corrido tres carreras ese día.');
    END IF;
END;
/
```

**Prueba:**

Las carreras con código 10, 11 y 12 son en la misma fecha (13-11-2016), voy a añadir otra carrera que sea en esa fecha y voy a añadir las participaciones de un mismo jockey en esas carreras.

```sql
insert into carrerasProfesionales  values (14, to_date('13-11-2016 13:00', 'DD-MM-YYYY HH24:MI'), 6125, 450, '01-01-2016', '01-06-2016');
```

```sql
insert into participaciones values(10, 2, 'Y6857984L', 2, 5);
insert into participaciones values(11, 2, 'Y6857984L', 7, 4);
insert into participaciones values(12, 2, 'Y6857984L', 1, 3);
insert into participaciones values(14, 2, 'Y6857984L', 1, 1);
```

![prueba1](/img/capturas-arantxa/80.png)

Con insert all vemos que nos aparece el problema de la tabla mutando.

```sql
INSERT ALL
    INTO participaciones VALUES(10, 5, '00015258D', 3, 4)
    INTO participaciones VALUES(11, 5, '00015258D', 5, 1)
    INTO participaciones VALUES(12, 5, '00015258D', 2, 5)
    INTO participaciones VALUES(14, 2, '00015258D', 1, 1)
SELECT * FROM dual;
```

![prueba2](/img/capturas-arantxa/81.png)

Borro los datos que he insertado y el trigger y vamos a intentar hacerlo de forma que no de error por mutación de la tabla.

```sql
delete from participaciones where codigocarrera=10 and codigocaballo=2 and dnijockey='Y6857984L';

delete from participaciones where codigocarrera=11 and codigocaballo=2 and dnijockey='Y6857984L';

delete from participaciones where codigocarrera=12 and codigocaballo=2 and dnijockey='Y6857984L';

delete from participaciones where codigocarrera=14 and codigocaballo=2 and dnijockey='Y6857984L';

drop trigger evitar_tres_carreras_jockey;
```

## Mutante

Mi código lo muestro a continuación pero no hemos conseguido hacer que funcione, no vemos dónde está el error (ningún compañero del grupo). El error es que al hacer inserts individuales no detecta que haya jockeys que ya hayan participado tres veces ese día, y al hacer insert all aparece un error como el siguiente:

```sql
ORA-04091: la tabla ADMIN.PARTICIPACIONES esta mutando, puede que el
disparador/la funcion no puedan verla
ORA-06512: en "ADMIN.RELLENARTABLACONTROLJOCKEYS", linea 3
ORA-06512: en "ADMIN.RELLENARTABLACONTROLJOCKEYS", linea 8
ORA-04088: error durante la ejecucion del disparador
'ADMIN.RELLENARTABLACONTROLJOCKEYS'
```

**Módulos de programación:**

```sql
CREATE OR REPLACE PACKAGE CONTROLJOCKEYS
AS
TYPE tREGISTROJOCKEYFECHA IS RECORD
(
  DNIJOCKEY PARTICIPACIONES.DNIJOCKEY%TYPE,
  FECHAHORA DATE
);
TYPE tTABLACONTROLJOCKEYS IS TABLE OF tREGISTROJOCKEYFECHA
INDEX BY BINARY_INTEGER;
contar tTABLACONTROLJOCKEYS;
END CONTROLJOCKEYS;
/



CREATE OR REPLACE TRIGGER RELLENARTABLACONTROLJOCKEYS
BEFORE INSERT OR UPDATE ON PARTICIPACIONES
DECLARE
  CURSOR C_DNIFECHA IS
    SELECT P.DNIJOCKEY, C.FECHAHORA
    FROM PARTICIPACIONES P, CARRERASPROFESIONALES C
    WHERE P.CODIGOCARRERA = C.CODIGOCARRERA;
  INDICE NUMBER := 0;
BEGIN
  FOR v IN C_DNIFECHA LOOP
    CONTROLJOCKEYS.CONTAR(INDICE).DNIJOCKEY := v.DNIJOCKEY;
    CONTROLJOCKEYS.CONTAR(INDICE).FECHAHORA := TRUNC(v.FECHAHORA);
    INDICE := INDICE + 1;
  END LOOP;
END;
/



CREATE OR REPLACE FUNCTION superacarreras (p_dni PARTICIPACIONES.DNIJOCKEY%TYPE, p_fecha DATE)
RETURN NUMBER
AS
  v_exist NUMBER := 0;
BEGIN
  FOR i IN CONTROLJOCKEYS.CONTAR.FIRST .. CONTROLJOCKEYS.CONTAR.LAST LOOP
    IF CONTROLJOCKEYS.CONTAR(i).DNIJOCKEY = P_DNI 
    AND CONTROLJOCKEYS.CONTAR(i).FECHAHORA = P_FECHA THEN
      v_exist := v_exist + 1;
    END IF;
  END LOOP;
  -- Si el jockey ha participado en tres o más carreras en la misma fecha,
  -- se devuelve 1 (verdadero), en caso contrario se devuelve 0 (falso).
  IF v_exist >= 3 THEN
    RETURN 1;
  ELSE
    RETURN 0;
  END IF;
END;
/



CREATE OR REPLACE PROCEDURE actualizartablacontroljockeys (P_DNIJOCKEY PARTICIPACIONES.DNIJOCKEY%TYPE, P_FECHA_CARRERA CARRERASPROFESIONALES.FECHAHORA%TYPE)
IS
  INDICE NUMBER := CONTROLJOCKEYS.CONTAR.LAST + 1;
BEGIN
  CONTROLJOCKEYS.CONTAR(INDICE).DNIJOCKEY := P_DNIJOCKEY;
  CONTROLJOCKEYS.CONTAR(INDICE).FECHAHORA := P_FECHA_CARRERA;
END actualizartablacontroljockeys;
/



CREATE OR REPLACE TRIGGER MaxTresCarreras
BEFORE INSERT OR UPDATE ON PARTICIPACIONES
FOR EACH ROW
DECLARE
  V_DNIJOCKEY PARTICIPACIONES.DNIJOCKEY%TYPE; 
  V_FECHA_CARRERA CARRERASPROFESIONALES.FECHAHORA%TYPE;
BEGIN
  V_DNIJOCKEY := :NEW.DNIJOCKEY;
  select c.fechahora into V_FECHA_CARRERA from carrerasprofesionales c where c.codigocarrera=:new.codigocarrera;

  IF SUPERACARRERAS(V_DNIJOCKEY, V_FECHA_CARRERA) = 1 THEN
    RAISE_APPLICATION_ERROR(-20002, 'No puedes participar en más de tres carreras en un mismo día');
  ELSE
    actualizartablacontroljockeys(V_DNIJOCKEY, V_FECHA_CARRERA);
  END IF;
END MaxTresCarreras;
/
```

## Prueba de errores

Creo que el error puede estar en RELLENARTABLACONTROLJOCKEYS. A continuación comento algunas cosas que he probado:

- He probado si la función superacarreras funcionaba y he comprobado que efectivamente funciona:

```sql
select superacarreras('Y6857984L', '13-11-2016') from dual;

SUPERACARRERAS('Y6857984L','13-11-2016')
----------------------------------------
			    1

select superacarreras('00015258D', '13-11-2016') from dual;

SUPERACARRERAS('00015258D','13-11-2016')
----------------------------------------
				0

select superacarreras('09849927Q', '13-11-2016') from dual;

SUPERACARRERAS('09849927Q','13-11-2016')
----------------------------------------
				1
```

- He probado a meter "prints" en el código para encontrar dónde falla, por ejemplo en RELLENARTABLACONTROLJOCKEYS:

```sql
CREATE OR REPLACE TRIGGER RELLENARTABLACONTROLJOCKEYS
BEFORE INSERT OR UPDATE ON PARTICIPACIONES
DECLARE
  CURSOR C_DNIFECHA IS
    SELECT P.DNIJOCKEY, C.FECHAHORA
    FROM PARTICIPACIONES P, CARRERASPROFESIONALES C
    WHERE P.CODIGOCARRERA = C.CODIGOCARRERA;
  INDICE NUMBER := 0;
BEGIN
  dbms_output.put_line('Empezando cursor');
  FOR v IN C_DNIFECHA LOOP
    CONTROLJOCKEYS.CONTAR(INDICE).DNIJOCKEY := v.DNIJOCKEY;
    CONTROLJOCKEYS.CONTAR(INDICE).FECHAHORA := TRUNC(v.FECHAHORA);
    INDICE := INDICE + 1;
    dbms_output.put_line('Indice:'||INDICE);
    dbms_output.put_line('IndiceDNI:'||CONTROLJOCKEYS.CONTAR(INDICE).DNIJOCKEY);
    dbms_output.put_line('IndiceFECHA:'||CONTROLJOCKEYS.CONTAR(INDICE).FECHAHORA);
  END LOOP;
  dbms_output.put_line('Terminado cursor');
END;
/
```

- También he probado lo anterior en la función superacarreras.

```sql
CREATE OR REPLACE FUNCTION superacarreras (p_dni PARTICIPACIONES.DNIJOCKEY%TYPE, p_fecha DATE)
RETURN NUMBER
AS
  v_exist NUMBER := 0;
BEGIN
  FOR i IN CONTROLJOCKEYS.CONTAR.FIRST .. CONTROLJOCKEYS.CONTAR.LAST LOOP
    dbms_output.put_line('superacarreras i '||i);
    dbms_output.put_line('superacarreras dni '||P_DNI || 'vs DNI ' || CONTROLJOCKEYS.CONTAR(i).DNIJOCKEY);
    dbms_output.put_line('superacarreras fechahora '||P_FECHA || 'vs fechahora ' || CONTROLJOCKEYS.CONTAR(i).FECHAHORA);
    IF CONTROLJOCKEYS.CONTAR(i).DNIJOCKEY = P_DNI 
    AND CONTROLJOCKEYS.CONTAR(i).FECHAHORA = P_FECHA THEN
      v_exist := v_exist + 1;
      dbms_output.put_line('dentro if');
    END IF;
    ....
```

- He hecho lo mismo con el último trigger MaxTresCarreras.

```sql
CREATE OR REPLACE TRIGGER MaxTresCarreras
...
BEGIN
  dbms_output.put_line('Empieza la ejecución');
  V_DNIJOCKEY := :NEW.DNIJOCKEY;
  select c.fechahora into V_FECHA_CARRERA from carrerasprofesionales c where c.codigocarrera=:new.codigocarrera;
  dbms_output.put_line('Se ha hecho el select y se han guardado los datos');
  .....
```

La primera vez que metí un dbms_output de comprobación en RELLENARTABLACONTROLJOCKEYS me apareció lo siguiente:

![error](/img/capturas-arantxa/85.png)

Lo cual me pareció raro, ya que una vez terminado el loop no debería haberme aparecido de nuevo "Empezando cursor".

Al meter los prints de dentro del loop en el mismo trigger, el error cambió.

![error](/img/capturas-arantxa/86.png)

Al hacer un insert individual me aparece el mensaje de que ya ha participado más de tres veces por lo que he intentado borrar los datos de la colección pero me sigue dando el mismo error a pesar de no haber más datos que tengan al mismo jockey participando el mismo día.

![error](/img/capturas-arantxa/87.png)

Para borrar los datos de la colección he hecho lo siguiente:

```sql
CREATE OR REPLACE PROCEDURE borrar_controljockeys
AS
  -- Declare a variable to hold the number of elements in the collection
  v_count NUMBER;
BEGIN
  -- Get the number of elements in the collection
  v_count := CONTROLJOCKEYS.CONTAR.COUNT;

  -- Loop through the collection and delete each element
  FOR i IN 1 .. v_count LOOP
    CONTROLJOCKEYS.CONTAR.DELETE(1);
  END LOOP;
END borrar_controljockeys;
/

exec borrar_controljockeys;
```
