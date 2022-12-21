# Ejercicio 6

> Realiza los módulos de programación necesarios para evitar que un mismo propietario envíe dos caballos a una misma carrera.


## PASO 1
Primero voy a crear el paquete que albergue la tabla vacía con el código del dni de la tabla caballos, y el código de la carrera de la tabla participaciones, le damos los nombres que consideremos necesarios y al final al realizar el llamamiento del paquete debemos utilizar CONTROLPROPIETARIOS.CONTAR y el nombre del registro de la tabla, ya sea codigodni o codigocarrera.


```sql
CREATE OR REPLACE PACKAGE CONTROLPROPIETARIOS
AS
V_CODCABALLO NUMBER;
TYPE tREGISTRODNI IS RECORD
(
CODIGODNI CABALLOS.DNIPROPIETARIO%type,
CODIGOCARRERA PARTICIPACIONES.CODIGOCARRERA%type
);
TYPE tTABLACONTROLDNI IS TABLE OF tREGISTRODNI
INDEX BY BINARY_INTEGER;
CONTAR tTABLACONTROLDNI;
END CONTROLPROPIETARIOS;
/
```
## PASO 2
Creamos un trigger que realice la función de rellenar las tablas que hemos creado en el paquete, esto afectará a la tabla que hemos creado en memoria, entonces en ningún momento toca la tabla participaciones, por lo tanto no mutará, así que se le puede hacer consultas y cursores. El índice es el identificador que s ele proporciona a la fila de la tabla, como vemos en el bucle for, tenemos un índice que al ser recorrido va insertando los datos de la tabla participaciones en el índice, y generará una nueva fila (+1) hasta que se hayan insertado todos los datos.

```sql
CREATE OR REPLACE TRIGGER RELLENARVARIABLESDNI
BEFORE INSERT OR UPDATE ON PARTICIPACIONES
DECLARE
CURSOR C_BUSCAR IS select CODIGOCARRERA,DNIPROPIETARIO from PARTICIPACIONES P,CABALLOS C WHERE C.CODIGOCABALLO=P.CODIGOCABALLO;
INDICE NUMBER:=0;
BEGIN
FOR V_BUSCAR IN C_BUSCAR LOOP
    CONTROLPROPIETARIOS.CONTAR(INDICE).CODIGODNI := V_BUSCAR.DNIPROPIETARIO;
    CONTROLPROPIETARIOS.CONTAR(INDICE).CODIGOCARRERA := V_BUSCAR.CODIGOCARRERA;
    INDICE:=INDICE+1;
END LOOP;
END RELLENARVARIABLESDNI;
/
```

# PASO 3
Vamos a generar una función que realice la comprobación de que esa carrera existe, entonces en el bucle for vamos a recorrer el paquete de principio a fin buscando las coincidencias de que los nuevos parámetros que se inserten van a ser los mismos, que los que ya se albergan en el array (tabla en memoria), así que si el nuevo dni que introduzca coincide con uno que esté dentro de la tabla y el nuevo código carrera coincide tabién, la función devolverá un 1, que sería similar a un booleano.


```sql
CREATE OR REPLACE FUNCTION EXISTECARRERADNI (p_dni CABALLOS.DNIPROPIETARIO%type, p_carrera PARTICIPACIONES.CODIGOCARRERA%type)
return number
IS
v_existe number:=0;
BEGIN
for i in CONTROLPROPIETARIOS.CONTAR.FIRST .. CONTROLPROPIETARIOS.CONTAR.LAST LOOP
        if CONTROLPROPIETARIOS.CONTAR(i).CODIGODNI = p_dni and CONTROLPROPIETARIOS.CONTAR(i).CODIGOCARRERA = p_carrera THEN
            v_existe:=1;
        end if;
END LOOP;
return v_existe;
END;
/
```

Dentro de este procedimiento vamos a declarar una variable que va a ser comparada con el nuevo código caballo introducido en la tabla participaciones, de manera que podemos ingresarlo como parámetro de entrada a ACTUALIZARPARTICIPACIONES junto con :NEW.CODIGOCARRERA que obtendremos de la propia tabla del trigger, luego, procedemos a ver que si la función anterior ha resultado ser 1 entonces salta un RAISE_APPLICATION_ERROR con su respectivo mensaje, que es desde un principio el objetivo del ejercicio.

Si la función no devuelve 1 procederíamos a ACTUALIZARPARTICIPACIONES, esto debemos hacerlo porque cuando ingresamos datos, la tabla en memoria debemos actualizarlas ya que si no lo hacemos el trigger sólo contemplaría los antiguos registros.

Este trigger es en esencia el que nos permite levantar la excepción del ejercicio.

```sql
CREATE OR REPLACE TRIGGER CONTROLARCABALLOS
BEFORE INSERT OR UPDATE ON PARTICIPACIONES
FOR EACH ROW
DECLARE
V_DNIPROP CABALLOS.DNIPROPIETARIO%TYPE;
BEGIN
select DNIPROPIETARIO into V_DNIPROP from CABALLOS where CODIGOCABALLO = :new.CODIGOCABALLO;
IF EXISTECARRERADNI(V_DNIPROP,:NEW.CODIGOCARRERA) = 1 THEN
    RAISE_APPLICATION_ERROR(-20002,'No puedes meter más de un caballo por propietario');
ELSE
    ACTUALIZARPARTICIPACIONES(V_DNIPROP,:NEW.CODIGOCARRERA);
END IF;
END CONTROLARCABALLOS;
/
```
## PASO 4

Por último podemos ver que en ACTUALIZARPARTICIPACIONES declaramos el índice, el cual cuando se llame procederá a ingresar una fila nueva, por tanto en los valores CODIGODNI y CODIGOCARRERA vamos a actualizar los nuevos valores con los introducidos por los parámetros, para así cuando se vuelva a ingresar, se vuelve a actualizar por cada insert que yo realice.

```sql
CREATE OR REPLACE PROCEDURE ACTUALIZARPARTICIPACIONES (p_dni CABALLOS.DNIPROPIETARIO%type, p_carrera PARTICIPACIONES.CODIGOCARRERA%type)
IS
indice number;
BEGIN
indice:=CONTROLPROPIETARIOS.CONTAR.LAST+1;
CONTROLPROPIETARIOS.CONTAR(indice).CODIGODNI:=p_dni;
CONTROLPROPIETARIOS.CONTAR(indice).CODIGOCARRERA:=p_carrera;
END;
/
```


