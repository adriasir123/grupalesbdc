# Ejercicio 5

## Enunciado

Realiza un procedimiento que muestre los usuarios que pueden conceder privilegios de sistema a otros usuarios y cuales son dichos privilegios.

## Código

```sql
SQL> CREATE OR REPLACE PROCEDURE mostrar_privilegios_sistema
AS
  CURSOR c_info IS
    SELECT USERNAME, PRIVILEGE
    FROM USER_SYS_PRIVS
    WHERE PRIVILEGE IN ('CREATE USER', 'ALTER USER', 'DROP USER');
  v_usuario VARCHAR2(30);
  v_privilegio VARCHAR2(30);
BEGIN
  DBMS_OUTPUT.PUT_LINE('Usuarios que tienen privilegios de sistema y sus privilegios:');
  DBMS_OUTPUT.PUT_LINE('--------------------------------------------------------------');
  FOR i IN c_info LOOP
    v_usuario := i.USERNAME;
    v_privilegio := i.PRIVILEGE;
    DBMS_OUTPUT.PUT_LINE(v_usuario || ' tiene el privilegio de ' || v_privilegio);
  END LOOP;
END mostrar_privilegios_sistema;
/

Procedimiento creado.
```

## Comprobación

```sql
SQL> exec mostrar_privilegios_sistema;

Usuarios que tienen privilegios de sistema y sus privilegios:
--------------------------------------------------------------
SYS tiene el privilegio de ALTER USER
SYS tiene el privilegio de DROP USER
SYS tiene el privilegio de CREATE USER

PL/SQL procedure successfully completed.

Commit complete.
```