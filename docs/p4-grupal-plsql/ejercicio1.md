# Ejercicio 1


## Realiza una función que reciba como parámetros un código de carrera y un código de caballo y devuelva el importe total que tendrá que pagar el hipódromo a los apostantes suponiendo que dicha carrera sea ganada por el caballo recibido como parámetro. Se deben contemplar las siguientes excepciones: Carrera inexistente, Caballo inexistente, Caballo no participante en esa carrera.


```sql
create or replace function importetotal (p_codcarrera apuestas.codigocarrera%type, p_codcaballo apuestas.codigocaballo%type)
return number
IS
v_mult number;
BEGIN
COMPROBARVACIAS(p_codcarrera,p_codcaballo);
select importeapostado*tantoauno into v_mult from apuestas where codigocarrera = p_codcarrera and codigocaballo = p_codcaballo;
dbms_output.put_line('Hay que pagar al ganador un total de ' || v_mult || 'euros');
return v_mult;
END;
/

CREATE OR REPLACE PROCEDURE COMPROBARVACIAS (p_codcarrera apuestas.codigocarrera%type, p_codcaballo apuestas.codigocaballo%type)
IS
v_carrera number;
v_caballo number;
v_caballocarrera number;
BEGIN
    select count (*) into v_carrera from apuestas where codigocarrera = p_codcarrera;
    if v_carrera = 0 then
        raise_application_error(-20112, 'No existe la carrera');
    end if;
    select count (*) into v_caballo from apuestas where codigocaballo = p_codcaballo;
    if v_caballo = 0 then
        raise_application_error(-20110, 'No se ha encontrado el caballo');
    end if;
        select count (*) into v_caballocarrera from participaciones where codigocarrera = p_codcarrera and codigocaballo = p_codcaballo;
    if v_caballocarrera = 0 then
        raise_application_error(-20103, 'No existen caballos en esas en esas carreras');
    end if;
END;
/
```

LO SIGUIENTE ES PARA PROBAR LA PRIMERA FUNCIÓN, COMPROBANDO EL PROCEDIMIENTO:



```sql
create or replace procedure importetotal2 (p_codcarrera apuestas.codigocarrera%type, p_codcaballo apuestas.codigocaballo%type)
IS
v_mult number;
BEGIN
COMPROBARVACIAS(p_codcarrera,p_codcaballo);
select importeapostado*tantoauno into v_mult from apuestas where codigocarrera = p_codcarrera and codigocaballo = p_codcaballo;
dbms_output.put_line('Hay que pagar al ganador un total de ' || v_mult || 'euros');
END;
/


CREATE OR REPLACE PROCEDURE COMPROBARVACIAS (p_codcarrera apuestas.codigocarrera%type, p_codcaballo apuestas.codigocaballo%type)
IS
v_carrera number;
v_caballo number;
v_caballocarrera number;
BEGIN
    select count (*) into v_carrera from apuestas where codigocarrera = p_codcarrera;
    if v_carrera = 0 then
        raise_application_error(-20112, 'No existe la carrera');
    end if;
    select count (*) into v_caballo from apuestas where codigocaballo = p_codcaballo;
    if v_caballo = 0 then
        raise_application_error(-20110, 'No se ha encontrado el caballo');
    end if;
        select count (*) into v_caballocarrera from participaciones where codigocarrera = p_codcarrera and codigocaballo = p_codcaballo;
    if v_caballocarrera = 0 then
        raise_application_error(-20103, 'No existen caballos en esas en esas carreras');
    end if;
END;
/
```