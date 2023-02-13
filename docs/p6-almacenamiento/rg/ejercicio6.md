# Ejercicio 6

## Enunciado

Realizar un procedimiento llamado `BalanceoCargaTemp` que balancee la carga de usuarios entre los tablespaces temporales existentes. Para ello se averiguará cuántos existen y asignará los usuarios entre ellos de forma equilibrada. Si es necesario para comprobar su funcionamiento, crea tablespaces temporales nuevos.

## Código

```sql
CREATE OR REPLACE PROCEDURE BalanceoCargaTemp IS

    CURSOR c_usuarios IS
        SELECT username
        FROM DBA_USERS
        WHERE account_status = 'OPEN'
          AND username NOT IN ('SYS','SYSTEM');

    v_counter             NUMBER := -1;
    v_total_tablespaces   NUMBER;
    v_tablespace_asignado dba_tablespaces.tablespace_name%type;

BEGIN

    SELECT COUNT(*) INTO v_total_tablespaces
    FROM DBA_TABLESPACES
    WHERE contents = 'TEMPORARY';

    FOR i IN c_usuarios LOOP

        IF v_counter = v_total_tablespaces - 1 THEN
            v_counter := -1;
        END IF;

        v_counter := v_counter + 1;

        SELECT tablespace_name INTO v_tablespace_asignado
        FROM DBA_TABLESPACES
        WHERE contents = 'TEMPORARY'
        OFFSET v_counter ROWS
        FETCH NEXT 1 ROWS ONLY;

        EXECUTE IMMEDIATE 'ALTER USER ' || i.username || ' TEMPORARY TABLESPACE ' || v_tablespace_asignado;

    END LOOP;

END;
/
```

!!! Info

    La combinación de contadores y OFFSET la utilizo para poder hacer un bucle infinito secuencial sobre los tablespaces temporales, cuando una combinación de FOR y WHILE no tiene sentido.  
    Tampoco podía recorrer un cursor infinitamente ya que siempre acaba en algún momento y no vuelve al inicio

## Comprobaciones

Lo ejecuto una primera vez sin crear tablespaces temporales extras:

```shell
EXEC BalanceoCargaTemp;

PL/SQL procedure successfully completed.
```

Muestro el resultado:

```sql
SELECT username, temporary_tablespace
FROM DBA_USERS
WHERE account_status = 'OPEN' AND username NOT IN ('SYS','SYSTEM');

USERNAME                                                                                                                         TEMPORARY_TABLESPACE
-------------------------------------------------------------------------------------------------------------------------------- ------------------------------
BECARIO                                                                                                                          TEMP
EXAMENBD34                                                                                                                       TEMP
TESTING                                                                                                                          TEMP
SCOTT                                                                                                                            TEMP
HIPODROMO                                                                                                                        TEMP
USRPRACTICA1                                                                                                                     TEMP
```

Como vemos no ha podido balancear *(obviamente)* porque por defecto sólo tenemos 1 tablespace temporal.

Para probar el balanceo, añado un segundo tablespace:

```sql
CREATE TEMPORARY TABLESPACE TEMP2 TEMPFILE 'temporary2.dbf' SIZE 5M;

Tablespace created.
```

Vuelvo a ejecutar el procedimiento y muestro el resultado:

```sql
EXEC BalanceoCargaTemp;

PL/SQL procedure successfully completed.

SELECT username, temporary_tablespace
FROM DBA_USERS
WHERE account_status = 'OPEN' AND username NOT IN ('SYS','SYSTEM');

USERNAME                                                                                                                         TEMPORARY_TABLESPACE
-------------------------------------------------------------------------------------------------------------------------------- ------------------------------
BECARIO                                                                                                                          TEMP
TESTING                                                                                                                          TEMP
HIPODROMO                                                                                                                        TEMP
EXAMENBD34                                                                                                                       TEMP2
SCOTT                                                                                                                            TEMP2
USRPRACTICA1                                                                                                                     TEMP2
```

Como vemos ya sí ha hecho un balanceo de los tablespaces.

Podría incluso añadir un tercer tablespace y comprobar que se sigue balanceando:

```sql
CREATE TEMPORARY TABLESPACE TEMP3 TEMPFILE 'temporary3.dbf' SIZE 5M;

Tablespace created.
```

```sql
EXEC BalanceoCargaTemp;

PL/SQL procedure successfully completed.

SELECT username, temporary_tablespace
FROM DBA_USERS
WHERE account_status = 'OPEN' AND username NOT IN ('SYS','SYSTEM');

USERNAME                                                                                                                         TEMPORARY_TABLESPACE
-------------------------------------------------------------------------------------------------------------------------------- ------------------------------
BECARIO                                                                                                                          TEMP
EXAMENBD34                                                                                                                       TEMP
TESTING                                                                                                                          TEMP2
SCOTT                                                                                                                            TEMP2
HIPODROMO                                                                                                                        TEMP3
USRPRACTICA1                                                                                                                     TEMP3
```

Se ha vuelto a balancear equitativamente pero si aún así queremos estar 100% seguros de que el procedimiento balancea a la perfección, podemos añadir la siguiente línea al final del bucle `FOR` del procedimiento:

```sql
DBMS_OUTPUT.PUT_LINE('ALTER USER ' || i.username || ' TEMPORARY TABLESPACE ' || v_tablespace_asignado || ';');
```

De esta manera, cuando ejecutemos el procedimiento veremos las órdenes que está ejecutando:

```sql
EXEC BalanceoCargaTemp;
ALTER USER BECARIO TEMPORARY TABLESPACE TEMP;
ALTER USER EXAMENBD34 TEMPORARY TABLESPACE TEMP2;
ALTER USER TESTING TEMPORARY TABLESPACE TEMP3;
ALTER USER SCOTT TEMPORARY TABLESPACE TEMP;
ALTER USER HIPODROMO TEMPORARY TABLESPACE TEMP2;
ALTER USER USRPRACTICA1 TEMPORARY TABLESPACE TEMP3;

PL/SQL procedure successfully completed.
```

Este mismo procedimiento seguiría funcionando independientemente de cuántos usuarios y tablespaces temporales nuevos se creasen y borrasen.
