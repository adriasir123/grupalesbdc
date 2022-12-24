# Ejercicio 6

> Realiza los módulos de programación necesarios para evitar que un mismo propietario envíe dos caballos a una misma carrera.

## PASO 1

Primero voy a crear el paquete que albergue la tabla vacía con el código del dni de la tabla caballos, y el código de la carrera de la tabla participaciones, le damos los nombres que consideremos necesarios y al final al realizar el llamamiento del paquete debemos utilizar CONTROLPROPIETARIOS.CONTAR y el nombre del registro de la tabla, ya sea codigodni o codigocarrera.

```sql
CREATE OR REPLACE PACKAGE controlpropietarios AS
    v_codcaballo NUMBER;
    TYPE tregistrodni IS
        RECORD ( codigodni caballos.dnipropietario%type, codigocarrera participaciones.codigocarrera%type );
    TYPE ttablacontroldni IS
        TABLE OF tregistrodni INDEX BY BINARY_INTEGER;
    contar       ttablacontroldni;
END controlpropietarios;
/
```

## PASO 2

Creamos un trigger que realice la función de rellenar las tablas que hemos creado en el paquete, esto afectará a la tabla que hemos creado en memoria entonces en ningún momento toca la tabla participaciones, por lo tanto no mutará, así que se le puede hacer consultas y cursores. El índice es el identificador que se le proporciona a la fila de la tabla, como vemos en el bucle for, tenemos un índice que al ser recorrido va insertando los datos de la tabla participaciones en el índice, y generará una nueva fila (+1) hasta que se hayan insertado todos los datos.

```sql
CREATE OR REPLACE TRIGGER rellenarvariablesdni
    BEFORE INSERT OR UPDATE ON participaciones
DECLARE
    CURSOR c_buscar IS
        SELECT codigocarrera, dnipropietario
        FROM participaciones p, caballos c
        WHERE c.codigocaballo=p.codigocaballo;
    indice NUMBER:=0;
BEGIN
    FOR v_buscar IN c_buscar LOOP
        controlpropietarios.contar(indice).codigodni := v_buscar.dnipropietario;
        controlpropietarios.contar(indice).codigocarrera := v_buscar.codigocarrera;
        indice:=indice+1;
    END LOOP;
END rellenarvariablesdni;
/
```

## PASO 3

Vamos a generar una función que realice la comprobación de que esa carrera existe, entonces en el bucle for vamos a recorrer el paquete de principio a fin buscando las coincidencias de que los nuevos parámetros que se inserten van a ser los mismos que los ya albergados en el array (tabla en memoria), así que si el nuevo dni introducido coincide con uno que esté dentro de la tabla y el nuevo código carrera coincide tabién, la función devolverá un 1, que sería similar a un booleano.

```sql
CREATE OR REPLACE FUNCTION existecarreradni (
    p_dni caballos.dnipropietario%type,
    p_carrera participaciones.codigocarrera%type
) RETURN NUMBER IS
    v_existe NUMBER:=0;
BEGIN
    FOR i IN controlpropietarios.contar.first .. controlpropietarios.contar.last LOOP
        IF controlpropietarios.contar(i).codigodni = p_dni AND controlpropietarios.contar(i).codigocarrera = p_carrera THEN
            v_existe:=1;
        END IF;
    END LOOP;
    RETURN v_existe;
END;
/
```

Dentro de este procedimiento vamos a declarar una variable que va a ser comparada con el nuevo código caballo introducido en la tabla participaciones, de manera que podemos ingresarlo como parámetro de entrada a ACTUALIZARPARTICIPACIONES junto con :NEW.CODIGOCARRERA que obtendremos de la propia tabla del trigger, luego, procedemos a ver que si la función anterior ha resultado ser 1 entonces salta un RAISE_APPLICATION_ERROR con su respectivo mensaje, que es desde un principio el objetivo del ejercicio.

Si la función no devuelve 1 procederíamos a ACTUALIZARPARTICIPACIONES, esto debemos hacerlo porque cuando ingresamos datos, la tabla en memoria debemos actualizarlas ya que si no lo hacemos el trigger sólo contemplaría los antiguos registros.

Este trigger es en esencia el que nos permite levantar la excepción del ejercicio.

```sql
CREATE OR REPLACE TRIGGER controlarcaballos
    BEFORE INSERT OR UPDATE ON participaciones
    FOR EACH ROW
DECLARE
    v_dniprop caballos.dnipropietario%type;
BEGIN
    SELECT dnipropietario INTO v_dniprop
    FROM caballos
    WHERE codigocaballo = :new.codigocaballo;
    
    IF existecarreradni(v_dniprop, :new.codigocarrera) = 1 THEN
        raise_application_error(-20003, 'No puedes meter más de un caballo por propietario');
    ELSE
        actualizarparticipaciones(v_dniprop, :new.codigocarrera);
    END IF;
END controlarcaballos;
/
```

## PASO 4

Por último podemos ver que en ACTUALIZARPARTICIPACIONES declaramos el índice, el cual cuando se llame procederá a ingresar una fila nueva, por tanto en los valores CODIGODNI y CODIGOCARRERA vamos a actualizar los nuevos valores con los introducidos por los parámetros, para así cuando se vuelva a ingresar, se vuelve a actualizar por cada insert que yo realice.

```sql
CREATE OR REPLACE PROCEDURE actualizarparticipaciones (
    p_dni caballos.dnipropietario%type,
    p_carrera participaciones.codigocarrera%type
) IS
    indice NUMBER;
BEGIN
    indice:=controlpropietarios.contar.last+1;
    controlpropietarios.contar(indice).codigodni:=p_dni;
    controlpropietarios.contar(indice).codigocarrera:=p_carrera;
END;
/
```

## PRUEBA DE FUNCIONAMIENTO

insert into participaciones values(90, 10, '09849927Q', 3, 4);
insert into participaciones values(90, 11, '09849927Q', 7, 6);

![prueba1](/img/capturas-antonio/comprobar-mutante.png)
