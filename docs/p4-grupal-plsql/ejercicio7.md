# Ejercicio 7

> Realiza los módulos de programación necesarios para evitar que un mismo jockey corra más de tres carreras en el mismo día.

Voy a crear un trigger que antes de insertar los datos de participación en una carrera compruebe si el jockey que se quiere insertar ha participado en más carreras ese mismo día.

```
CREATE OR REPLACE TRIGGER evitar_tres_carreras_jockey
BEFORE INSERT OR UPDATE ON participaciones
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





delete from participaciones where codigocarrera=10 and codigocaballo=2 and dnijockey='Y6857984L';

delete from participaciones where codigocarrera=11 and codigocaballo=2 and dnijockey='Y6857984L';

delete from participaciones where codigocarrera=12 and codigocaballo=2 and dnijockey='Y6857984L';

delete from participaciones where codigocarrera=14 and codigocaballo=2 and dnijockey='Y6857984L';






## Mutante

```
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
END RELLENARTABLACONTROLJOCKEYS;
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



CREATE OR REPLACE PROCEDURE actualizartablajockeys (P_DNIJOCKEY PARTICIPACIONES.DNIJOCKEY%TYPE, P_FECHA_CARRERA CARRERASPROFESIONALES.FECHAHORA%TYPE)
IS
  INDICE NUMBER := CONTROLJOCKEYS.CONTAR.LAST + 1;
BEGIN
  CONTROLJOCKEYS.CONTAR(INDICE).DNIJOCKEY := P_DNIJOCKEY;
  CONTROLJOCKEYS.CONTAR(INDICE).FECHAHORA := P_FECHA_CARRERA;
END actualizartablajockeys;
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
    actualizartablajockeys(V_DNIJOCKEY, V_FECHA_CARRERA);
  END IF;
END MaxTresCarreras;
/
```
