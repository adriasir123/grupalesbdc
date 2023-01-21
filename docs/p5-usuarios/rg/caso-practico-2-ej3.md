# Ejercicio 3

## 3. (ORACLE, Postgres) Realiza un procedimiento que reciba dos nombres de usuario y genere un script que asigne al primero los privilegios de inserción y modificación sobre todas las tablas del segundo, así como el de ejecución de cualquier procedimiento que tenga el segundo usuario.


Primero vamos a crear el procedimiento que cederá los privilegios sobre objetos sobre el segundo usuario:

```
CREATE OR REPLACE PROCEDURE conceder_privilegios (p_privilegiado VARCHAR2, p_cede VARCHAR2)
AS
  v_sql VARCHAR2(2000);
  v_tab_name VARCHAR2(40);
  v_proc_name VARCHAR2(40);
BEGIN
  COMPROBARUSUARIO(p_privilegiado,p_cede);
  FOR v_tablas IN (SELECT table_name FROM all_tables WHERE owner = p_cede) LOOP
    v_tab_name := v_tablas.table_name;
    v_sql := 'GRANT INSERT, UPDATE ON ' || p_cede || '.' || v_tab_name || ' TO ' || p_privilegiado;
    EXECUTE IMMEDIATE v_sql;
  END LOOP;

  FOR v_objeto IN (SELECT object_name FROM all_procedures WHERE owner = p_cede) LOOP
    v_proc_name := v_objeto.object_name;
    v_sql := 'GRANT EXECUTE ON ' || p_cede || '.' || v_proc_name || ' TO ' || p_privilegiado;
    EXECUTE IMMEDIATE v_sql;
  END LOOP;
END;
/
```

Ahora vamos a hacer el código de comprobación de usuario:

```
CREATE OR REPLACE PROCEDURE COMPROBARUSUARIO (p_privilegiado varchar2,p_cede varchar2)
IS
v_resultado number;
BEGIN
select count (USERNAME) into v_resultado from DBA_USERS WHERE USERNAME = p_privilegiado OR USERNAME = p_cede;
IF v_resultado < 2 THEN
  raise_application_error(-20110,'Hay uno o varios usuarios que no existen');
END IF;
END;
/
```


Ahora procedemos a hacer la prueba de ingresar un usuario que no exista:

![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-2.png)


Ahora la prueba de funcionamiento, entramos en el usuario antonio para ver si tenemos acceso a las tablas:

![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-2-2.png)