# Ejercicio 5

## Realiza un procedimiento que muestre los usuarios que pueden conceder privilegios de sistema a otros usuarios y cuales son dichos privilegios.

### Creamos Procedimiento:

```sql
CREATE OR REPLACE PROCEDURE MostrarUsuariosPrivilegios
IS
cursor c_usuarios is select grantee FROM dba_sys_privs;
BEGIN
  for i in c_usuarios loop
    DBMS_OUTPUT.PUT_LINE(i.grantee || ' puede conceder los siguientes privilegios: ');
    DBMS_OUTPUT.PUT_LINE('  ');
    MostrarPrivilegios(i.grantee);
  end loop;
END;
/

CREATE OR REPLACE PROCEDURE MostrarPrivilegios(p_usuario dba_sys_privs.GRANTEE%type)
IS
cursor c_privilegios is select privilege FROM dba_sys_privs;
BEGIN
  for i in c_privilegios loop
    DBMS_OUTPUT.PUT_LINE( i.privilege );
  end loop;
END;
/
```
