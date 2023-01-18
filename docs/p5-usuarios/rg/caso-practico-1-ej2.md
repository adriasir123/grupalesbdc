# Ejercicio 2

## Oracle

### Enunciado

Escribe una consulta que genere un script para quitar el privilegio de borrar registros en alguna tabla de SCOTT a los usuarios que lo tengan.

### Código

```sql
CREATE OR REPLACE FUNCTION generador_script_quitar_privilegios(
    p_tabla VARCHAR2
) RETURN VARCHAR2 IS
    CURSOR c_usuarios IS
        SELECT grantee
        FROM dba_tab_privs
        WHERE privilege = 'DELETE'
            AND table_name = p_tabla
            AND owner = 'SCOTT';
    v_output VARCHAR2(2000);
BEGIN
    FOR i IN c_usuarios LOOP
        v_output := v_output || 'REVOKE DELETE ON scott.' || p_tabla || ' FROM ' || i.grantee || ';' || chr(10);
    END LOOP;
    RETURN v_output;
END;
/
```

### Comprobaciones

Para que la prueba funcione añado los siguientes privilegios de borrado sobre la tabla a probar de SCOTT en algunos usuarios:

```sql
GRANT DELETE ON scott.dept TO hipodromo;
GRANT DELETE ON scott.dept TO examenbd34;
```

Lanzo la consulta que genera el script y muestro el output:

```sql
SELECT generador_script_quitar_privilegios('DEPT') AS "Copia y pega este script" FROM dual;
```

```sql
Copia y pega este script
--------------------------------------------------
REVOKE DELETE ON scott.DEPT FROM HIPODROMO;
REVOKE DELETE ON scott.DEPT FROM EXAMENBD34;
```

Ya sólo tendríamos que copiar y pegar el resultado, y quitaríamos los privilegios.

## PostgreSQL















## MariaDB




