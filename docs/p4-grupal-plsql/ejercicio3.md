# Ejercicio 3

> Realiza un trigger que controle que solo participan en una carrera caballos en el rango de edades permitido. Si es necesario, cambia el modelo de datos

```sql
-- Este procedimiento comprueba que la edad de un caballo sea permitida
-- según la carrera. Si la edad del caballo está en el rango permitido o no,
-- lo controla el parámetro OUT BOOLEAN

CREATE OR REPLACE PROCEDURE comprobar_edad(p_codigocarrera participaciones.codigocarrera%TYPE, p_codigocaballo participaciones.codigocaballo%TYPE, p_control OUT BOOLEAN)
IS
  v_fechanac_caballo caballos.fechanac%TYPE;
  v_fechanacmin_carreras carrerasprofesionales.fechanacmin%TYPE;
  v_fechanacmac_carreras carrerasprofesionales.fechanacmac%TYPE;
BEGIN
  SELECT fechanac
  INTO v_fechanac_caballo
  FROM caballos
  WHERE codigocaballo = p_codigocaballo;

  SELECT fechanacmin, fechanacmac
  INTO v_fechanacmin_carreras, v_fechanacmac_carreras
  FROM carrerasprofesionales
  WHERE codigocarrera = p_codigocarrera;

  IF v_fechanac_caballo <= v_fechanacmac_carreras AND v_fechanac_caballo >= v_fechanacmin_carreras THEN
    p_control := TRUE;
  ELSE
    p_control := FALSE;
  END IF;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RAISE_APPLICATION_ERROR(-20002, 'La carrera o caballo, o ambos, que estas intentando insertar no existen');
END;
/
```

```sql
-- Este es el bloque de código principal, el trigger que
-- monitoriza las participaciones. Desde aquí se llama al 
-- procedimiento hijo anterior

CREATE OR REPLACE TRIGGER monitorizar_participaciones
  BEFORE INSERT OR UPDATE ON participaciones
  FOR EACH ROW
DECLARE
  v_control BOOLEAN;
BEGIN
  comprobar_edad(:NEW.codigocarrera,:NEW.codigocaballo,v_control);
  IF v_control = FALSE THEN
    RAISE_APPLICATION_ERROR(-20001, 'El caballo con codigo ' || :NEW.codigocaballo || ' no esta en el rango de edades permitido para la carrera ' || :NEW.codigocarrera);
  END IF;
END;
/
```

!!! danger

    No se pueden llamar a funciones dentro de un trigger, sólo a procedimientos

Muestro la tabla `carrerasprofesionales`:

![carrerasprofesionales](https://i.imgur.com/S7S8wL8.png)

Muestro la tabla `caballos`:

![caballos](https://i.imgur.com/ii3xDhq.png)

Como podemos ver, con el estado actual de la base de datos no podríamos probar el trigger ya que todos los caballos están dentro del rango de fechas de nacimiento permitido por las carreras.

Tengo que insertar un caballo que se salga del rango:

```sql
INSERT INTO caballos values (11,'21913124n', 'test', to_date('11-11-2015', 'DD-MM-YYYY'), 'test' );
```

El trigger evitará que inserte este caballo en `participaciones`, en cualquier carrera, ya que se sale de cualquier rango de edades permitido:

```sql
INSERT INTO participaciones values(1, 11, 'Y6857984L', 1, 1);
INSERT INTO participaciones values(1, 11, 'Y6857984L', 1, 1)
            *
ERROR at line 1:
ORA-20001: El caballo con codigo 11 no esta en el rango de edades permitido para la carrera 1
ORA-06512: at "HIPODROMO.MONITORIZAR_PARTICIPACIONES", line 6
ORA-04088: error during execution of trigger 'HIPODROMO.MONITORIZAR_PARTICIPACIONES'
```

En cambio, si intento insertar un caballo que esté dentro del rango de edades permitido, dejará insertar:

```sql
INSERT INTO participaciones values(1, 10, 'Y6857984L', 1, 1);

1 row created.
```

Como funcionalidad adicional y fuera de enunciado, he añadido una excepción para manejar el error que ocurre en caso de que intentemos insertar una carrera o un caballo que no existen:

```sql
INSERT INTO participaciones values(1, 100, 'Y6857984L', 1, 1);
INSERT INTO participaciones values(1, 100, 'Y6857984L', 1, 1)
            *
ERROR at line 1:
ORA-20002: La carrera o caballo, o ambos, que estas intentando insertar no existen
ORA-06512: at "HIPODROMO.COMPROBAR_EDAD", line 24
ORA-06512: at "HIPODROMO.MONITORIZAR_PARTICIPACIONES", line 4
ORA-04088: error during execution of trigger 'HIPODROMO.MONITORIZAR_PARTICIPACIONES'
```
