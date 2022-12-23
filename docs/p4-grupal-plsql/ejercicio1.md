# Ejercicio 1

> Realiza una función que reciba como parámetros un código de carrera y un código de caballo y devuelva el importe total que tendrá que pagar el hipódromo a los apostantes suponiendo que dicha carrera sea ganada por el caballo recibido como parámetro. Se deben contemplar las siguientes excepciones: Carrera inexistente, Caballo inexistente, Caballo no participante en esa carrera.

```sql
CREATE OR REPLACE FUNCTION importetotal (
    p_codcarrera apuestas.codigocarrera%type,
    p_codcaballo apuestas.codigocaballo%type
) RETURN NUMBER IS
    v_mult NUMBER;
BEGIN
    comprobarvacias(p_codcarrera, p_codcaballo);

    SELECT importeapostado * tantoauno INTO v_mult
    FROM apuestas
    WHERE codigocarrera = p_codcarrera AND codigocaballo = p_codcaballo;

    dbms_output.put_line('Hay que pagar al ganador un total de ' || v_mult || 'euros');
    
    RETURN v_mult;
END;
/

CREATE OR REPLACE PROCEDURE comprobarvacias (
    p_codcarrera apuestas.codigocarrera%type,
    p_codcaballo apuestas.codigocaballo%type
) IS
    v_carrera        NUMBER;
    v_caballo        NUMBER;
    v_caballocarrera NUMBER;
BEGIN
    SELECT COUNT (*) INTO v_carrera
    FROM apuestas
    WHERE codigocarrera = p_codcarrera;

    IF v_carrera = 0 THEN
        raise_application_error(-20112, 'No existe la carrera');
    END IF;

    SELECT COUNT (*) INTO v_caballo
    FROM apuestas
    WHERE codigocaballo = p_codcaballo;

    IF v_caballo = 0 THEN
        raise_application_error(-20110, 'No se ha encontrado el caballo');
    END IF;

    SELECT COUNT (*) INTO v_caballocarrera
    FROM participaciones
    WHERE codigocarrera = p_codcarrera AND codigocaballo = p_codcaballo;
    
    IF v_caballocarrera = 0 THEN
        raise_application_error(-20103, 'No existen caballos en esas en esas carreras');
    END IF;
END;
/
```

!!! note "Ejercicio de prueba"

    Este ejercicio está hecho para ser probado antes de utilizar la función.

```sql
CREATE OR REPLACE PROCEDURE importetotal2 (
    p_codcarrera apuestas.codigocarrera%type,
    p_codcaballo apuestas.codigocaballo%type
) IS
    v_mult NUMBER;
BEGIN
    comprobarvacias(p_codcarrera, p_codcaballo);

    SELECT importeapostado*tantoauno INTO v_mult
    FROM apuestas
    WHERE codigocarrera = p_codcarrera AND codigocaballo = p_codcaballo;
    
    dbms_output.put_line('Hay que pagar al ganador un total de ' || v_mult || 'euros');
END;
/


CREATE OR REPLACE PROCEDURE comprobarvacias (
    p_codcarrera apuestas.codigocarrera%type,
    p_codcaballo apuestas.codigocaballo%type
) IS
    v_carrera        NUMBER;
    v_caballo        NUMBER;
    v_caballocarrera NUMBER;
BEGIN
    SELECT COUNT (*) INTO v_carrera
    FROM apuestas
    WHERE codigocarrera = p_codcarrera;

    IF v_carrera = 0 THEN
        raise_application_error(-20112, 'No existe la carrera');
    END IF;

    SELECT COUNT (*) INTO v_caballo
    FROM apuestas
    WHERE codigocaballo = p_codcaballo;

    IF v_caballo = 0 THEN
        raise_application_error(-20110, 'No se ha encontrado el caballo');
    END IF;

    SELECT COUNT (*) INTO v_caballocarrera
    FROM participaciones
    WHERE codigocarrera = p_codcarrera AND codigocaballo = p_codcaballo;
    
    IF v_caballocarrera = 0 THEN
        raise_application_error(-20103, 'No existen caballos en esas en esas carreras');
    END IF;
END;
/
```
