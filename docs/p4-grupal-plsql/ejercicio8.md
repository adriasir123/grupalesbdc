# Ejercicio 8

> Realiza los módulos de programación necesarios para evitar que un propietario apueste por un caballo que no sea de su propiedad si en esa misma carrera corre algún caballo suyo

## Código

```sql
CREATE OR REPLACE PROCEDURE comprobar_si_propietario(p_dnicliente apuestas.dnicliente%TYPE, p_control_propietario OUT BOOLEAN)
IS
  v_propietario_si_no NUMBER;
BEGIN
  SELECT COUNT(*)
  INTO v_propietario_si_no
  FROM propietarios
  WHERE dni = p_dnicliente;

  IF v_propietario_si_no = 1 THEN
    p_control_propietario := TRUE;
  ELSE
    p_control_propietario := FALSE;
  END IF;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE comprobar_si_caballo_suyo(p_codigocaballo participaciones.codigocaballo%TYPE, p_dnicliente apuestas.dnicliente%TYPE, p_control_caballo_suyo OUT BOOLEAN)
IS
  v_caballo_suyo_si_no NUMBER;
BEGIN
  SELECT COUNT(*)
  INTO v_caballo_suyo_si_no
  FROM caballos
  WHERE dnipropietario = p_dnicliente AND codigocaballo = p_codigocaballo;

  IF v_caballo_suyo_si_no = 1 THEN
    p_control_caballo_suyo := TRUE;
  ELSE
    p_control_caballo_suyo := FALSE;
  END IF;
END;
/
```

```sql
CREATE OR REPLACE PROCEDURE comprobar_carrera(p_codigocarrera apuestas.codigocarrera%TYPE, p_dnicliente apuestas.dnicliente%TYPE, p_control_carrera OUT BOOLEAN)
IS
  CURSOR c_caballos IS
    SELECT codigocaballo
    FROM participaciones
    WHERE codigocarrera = p_codigocarrera;
  v_control_suyo BOOLEAN;
  v_contador NUMBER := 0;
BEGIN
  FOR i IN c_caballos LOOP
    comprobar_si_caballo_suyo(i.codigocaballo,p_dnicliente,v_control_suyo);
    IF v_control_suyo = TRUE THEN
      v_contador := v_contador+1;
    END IF;
  END LOOP;
  IF v_contador > 0 THEN
    p_control_carrera := TRUE;
  ELSE
    p_control_carrera := FALSE;
  END IF;
END;
/
```

```sql
CREATE OR REPLACE TRIGGER monitorizar_apuestas
  BEFORE INSERT OR UPDATE ON apuestas
  FOR EACH ROW
DECLARE
  v_control_propietario BOOLEAN;
  v_control_carrera BOOLEAN;
  v_control_caballo_suyo BOOLEAN;
BEGIN
  comprobar_si_propietario(:NEW.dnicliente,v_control_propietario);
  IF v_control_propietario = TRUE THEN
    comprobar_carrera(:NEW.codigocarrera,:NEW.dnicliente,v_control_carrera);
    IF v_control_carrera = TRUE THEN
      comprobar_si_caballo_suyo(:NEW.codigocaballo,:NEW.dnicliente,v_control_caballo_suyo);
      IF v_control_caballo_suyo = FALSE THEN
        RAISE_APPLICATION_ERROR(-20001,'El propietario con DNI ' || :NEW.dnicliente || ' no puede apostar por el caballo con codigo ' || :NEW.codigocaballo || ' porque no es suyo');
      END IF;
    END IF;
  END IF;
END;
/
```

## Requerimientos

Tengo que añadir un cliente que sea un propietario, porque en principio no hay ningún propietario que sea a su vez cliente para poder apostar:

```sql
INSERT INTO clientes values ('21913124n', 'Horacio', 'Arreguy', 'Fazzio', 'Calle Real, 12', 'Cordoba', 'Cordoba','678123467');
```

## Comprobaciones

Inserto una apuesta por un cliente normal, no propietario (dejará insertar):

```sql
INSERT INTO apuestas values ('28841115N',1 ,2 ,300 ,3.10);

1 row created.
```

Para las comprobaciones que vienen a continuación será necesario tener presente cuáles son los caballos que pertenecen al propietario anteriormente añadido como cliente:

```sql
SELECT *
FROM caballos
WHERE dnipropietario = '21913124n';
```

```sql
CODIGOCABALLO DNIPROPIET NOMBRE 		                FECHANAC  RAZA
------------- ---------- ------------------------------ --------- ------------------------------
	    1 21913124n  Wad Vison		                11-NOV-11 Quarter Horse
	    5 21913124n  Argos		                        04-APR-12 Purasangre ingles
	    9 21913124n  Perdigon		                24-JUN-12 Purasangre espanol
```

Inserto una apuesta de un propietario en una carrera en la que no corren caballos suyos.  
Por ejemplo, la carrera 1 nos sirve. Muestro los caballos que hay en esa carrera:

```sql
SELECT codigocaballo
FROM participaciones
WHERE codigocarrera = 1;
```

```sql
CODIGOCABALLO
-------------
	    2
	    4
	    6
```

Como no hay ningún caballo suyo, le dejará apostar tanto en uno suyo como en uno ajeno:

```sql
INSERT INTO apuestas values ('21913124n',1 ,1 ,300 ,3.10);

1 row created.

INSERT INTO apuestas values ('21913124n',1 ,10 ,300 ,3.10);

1 row created.
```

Ahora inserto una apuesta de un propietario en una carrera en la corre algún caballo suyo.  
Por ejemplo, la carrera 3 nos sirve. Muestro los caballos que hay en esa carrera:

```sql
SELECT codigocaballo
FROM participaciones
WHERE codigocarrera = 3;
```

```sql
CODIGOCABALLO
-------------
	    2
	    5
	    7
	    8
```

Como hay un caballo suyo, el 5, le dejará apostar en alguno de los demás suyos, pero no en ajenos:

```sql
INSERT INTO apuestas values ('21913124n',3 ,9 ,300 ,3.10);

1 row created.

INSERT INTO apuestas values ('21913124n',3 ,10 ,300 ,3.10);
INSERT INTO apuestas values ('21913124n',3 ,10 ,300 ,3.10)
            *
ERROR at line 1:
ORA-20001: El propietario con DNI 21913124n no puede apostar por el caballo con codigo 10 porque no es suyo
ORA-06512: at "HIPODROMO.MONITORIZAR_APUESTAS", line 12
ORA-04088: error during execution of trigger 'HIPODROMO.MONITORIZAR_APUESTAS'
```
