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

## Comprobaciones



```shell
EXEC BalanceoCargaTemp;
```





CREATE TEMPORARY TABLESPACE TEMP2 TEMPFILE 'temporary2.dbf' SIZE 5M;
CREATE TEMPORARY TABLESPACE TEMP3 TEMPFILE 'temporary3.dbf' SIZE 5M;





DROP TABLESPACE TEMP2
INCLUDING CONTENTS AND DATAFILES
CASCADE CONSTRAINTS;

DROP TABLESPACE TEMP3
INCLUDING CONTENTS AND DATAFILES
CASCADE CONSTRAINTS;


SELECT username, temporary_tablespace
FROM DBA_USERS
WHERE account_status = 'OPEN' AND username NOT IN ('SYS','SYSTEM');


DBMS_OUTPUT.PUT_LINE('ALTER USER ' || i.username || ' TEMPORARY TABLESPACE ' || v_tablespace_asignado || ';');

















































