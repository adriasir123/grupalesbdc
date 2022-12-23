# Ejercicio 3 PostgreSQL

> Realiza un trigger que controle que solo participan en una carrera caballos en el rango de edades permitido. Si es necesario, cambia el modelo de datos

```sql
CREATE OR REPLACE FUNCTION fn_comprobar_edad(p_codigocarrera participaciones.codigocarrera%TYPE, p_codigocaballo participaciones.codigocaballo%TYPE)
RETURNS boolean
AS $$
DECLARE
  v_fechanac_caballo caballos.fechanac%TYPE;
  v_fechanacmin_carreras carrerasprofesionales.fechanacmin%TYPE;
  v_fechanacmax_carreras carrerasprofesionales.fechanacmax%TYPE;
BEGIN
  SELECT fechanac
  INTO v_fechanac_caballo
  FROM caballos
  WHERE codigocaballo = p_codigocaballo;

  SELECT fechanacmin, fechanacmax
  INTO v_fechanacmin_carreras, v_fechanacmax_carreras
  FROM carrerasprofesionales
  WHERE codigocarrera = p_codigocarrera;

  IF v_fechanac_caballo <= v_fechanacmax_carreras AND v_fechanac_caballo >= v_fechanacmin_carreras THEN
    RETURN TRUE;
  ELSE
    RETURN FALSE;
  END IF;
END;
$$ LANGUAGE plpgsql;
-- (1)!
```

1. Esta es la función que comprueba que la edad de un caballo sea permitida según la carrera. Si la edad del caballo está en el rango permitido se devuelve `TRUE`, sino `FALSE`.

```sql
CREATE OR REPLACE FUNCTION fn_monitorizar_participaciones()
RETURNS TRIGGER AS $$
BEGIN
  IF fn_comprobar_edad(NEW.codigocarrera, NEW.codigocaballo) = FALSE THEN
    RAISE EXCEPTION 'El caballo con codigo % no esta en el rango de edades permitido para la carrera %', NEW.codigocaballo, NEW.codigocarrera;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
-- (1)!
```

1. Esta es la función que saltará cuando se ejecute el trigger. Desde aquí se llama a la función anterior.

```sql
CREATE TRIGGER tr_monitorizar_participaciones
  BEFORE INSERT OR UPDATE ON participaciones
  FOR EACH ROW
EXECUTE FUNCTION fn_monitorizar_participaciones();
-- (1)!
```

1. Este es el bloque de código principal, el trigger que monitoriza las participaciones. Desde aquí se llama a la función anterior.

Muestro la tabla `carrerasprofesionales`:

```sql
hipodromo=> select * from carrerasprofesionales;
 codigocarrera |      fechahora      | importepremio | limiteapuesta | fechanacmin | fechanacmax
---------------+---------------------+---------------+---------------+-------------+-------------
             1 | 2017-01-08 09:30:00 |       8750.00 |        650.00 | 2010-01-01  | 2012-12-31
             2 | 2017-01-08 10:00:00 |      11156.00 |        800.00 | 2010-01-01  | 2012-12-31
             3 | 2017-01-15 11:00:00 |       6125.00 |        450.00 | 2010-01-01  | 2012-12-31
             4 | 2017-01-15 12:00:00 |       7857.00 |        520.00 | 2010-01-01  | 2012-12-31
             5 | 2017-01-22 13:00:00 |       7000.00 |        470.00 | 2009-06-01  | 2012-12-31
             6 | 2017-01-22 14:00:00 |       6125.00 |        450.00 | 2009-06-01  | 2012-12-31
             7 | 2016-11-06 11:00:00 |       8750.00 |        650.00 | 2009-06-01  | 2012-12-31
             8 | 2016-11-06 12:00:00 |      11167.00 |        800.00 | 2009-06-01  | 2012-12-31
             9 | 2016-11-06 13:00:00 |       7000.00 |        470.00 | 2010-01-01  | 2013-06-01
            10 | 2016-11-13 10:00:00 |       8750.00 |        650.00 | 2010-01-01  | 2013-06-01
            11 | 2016-11-13 10:30:00 |       7000.00 |        470.00 | 2010-01-01  | 2013-06-01
            12 | 2016-11-13 12:00:00 |       6125.00 |        450.00 | 2010-01-01  | 2013-06-01
(12 rows)
```

Muestro la tabla `caballos`:

```sql
hipodromo=> select * from caballos;
 codigocaballo | dnipropietario |    nombre     |  fechanac  |        raza
---------------+----------------+---------------+------------+--------------------
             1 | 21913124n      | Wad Vison     | 2011-11-11 | Quarter Horse
             2 | Z7782152S      | Dagoberto     | 2010-10-10 | Purasangre español
             3 | 83069279H      | Atreus        | 2011-12-24 | Shagya Árabe
             4 | X58056225B     | Mayo          | 2010-03-15 | Purasangre español
             5 | 21913124n      | Argos         | 2012-04-04 | Purasangre inglés
             6 | Z7782152S      | Daniela       | 2011-06-06 | Shagya Árabe
             7 | 83069279H      | Kayak         | 2010-02-14 | Purasangre español
             8 | X58056225B     | Charming Star | 2011-09-23 | Quarter Horse
             9 | 21913124n      | Perdigon      | 2012-06-24 | Purasangre español
            10 | Z7782152S      | Innuendo      | 2011-01-31 | Darley Arabian
(10 rows)
```

Como podemos ver, con el estado actual de la base de datos no podríamos probar el trigger ya que todos los caballos están dentro del rango de fechas de nacimiento permitido por las carreras.

Tengo que insertar un caballo que se salga del rango:

```sql
hipodromo=> INSERT INTO caballos values (11,'21913124n', 'test', to_date('11-11-2015', 'DD-MM-YYYY'), 'test' );
INSERT 0 1
```

El trigger evitará que inserte este caballo en `participaciones`, en cualquier carrera, ya que se sale de cualquier rango de edades permitido:

```sql
hipodromo=> INSERT INTO participaciones values(1, 11, 'Y6857984L', 1, 1);
ERROR:  El caballo con codigo 11 no esta en el rango de edades permitido para la carrera 1
CONTEXT:  PL/pgSQL function fn_monitorizar_participaciones() line 4 at RAISE
```

En cambio, si intento insertar un caballo que esté dentro del rango de edades permitido, dejará insertar:

```sql
hipodromo=> INSERT INTO participaciones values(1, 10, 'Y6857984L', 1, 1);
INSERT 0 1
```
