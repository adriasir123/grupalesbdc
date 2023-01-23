# Ejercicio 4 Oracle

## Enunciado

Realizar un procedimiento que reciba un nombre de usuario y nos muestre cuántas sesiones tiene abiertas en este momento. Además, para cada una de dichas sesiones nos mostrará la hora de comienzo y el nombre de la máquina, sistema operativo y programa desde el que fue abierta.

## Código

```sql
CREATE OR REPLACE PROCEDURE MostrarSesionUsuario (
    p_usuario VARCHAR2
) IS
    CURSOR c_datos IS
        SELECT s.sid,s.serial#,s.username,s.machine,s.osuser,s.program,s.logon_time
        FROM V$SESSION s
        WHERE s.username = upper(p_usuario)
          AND status='ACTIVE';
BEGIN
    FOR i IN c_datos LOOP
        DBMS_OUTPUT.PUT_LINE('SID: ' || i.sid);
        DBMS_OUTPUT.PUT_LINE('Serial#: ' || i.serial#);
        DBMS_OUTPUT.PUT_LINE('Username: ' || i.username);
        DBMS_OUTPUT.PUT_LINE('Machine: ' || i.machine);
        DBMS_OUTPUT.PUT_LINE('OS User: ' || i.osuser);
        DBMS_OUTPUT.PUT_LINE('Program: ' || i.program);
        DBMS_OUTPUT.PUT_LINE('Logon Time: ' || i.logon_time);
        DBMS_OUTPUT.PUT_LINE('----------------');
    END LOOP;
END;
/
```

## Comprobaciones

```sql
SQL> exec MostrarSesionUsuario('sys');

SID: 22
Serial#: 29357
Username: SYS
Machine: joseju\joseju
OS User: joseju
Program: OracleVSCodeServer60.dll
Logon Time: 17-JAN-23
----------------
SID: 401
Serial#: 17964
Username: SYS
Machine: oracle
OS User: oracle
Program: oracle@oracle (OFSD)
Logon Time: 17-JAN-23
----------------

PL/SQL procedure successfully completed.
```
