# Ejercicio 2

## 1. Oracle

### 1. Enunciado

Escribe una consulta que genere un script para quitar el privilegio de borrar registros en alguna tabla de SCOTT a los usuarios que lo tengan.

### 1. Código

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

### 1. Comprobaciones

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

## 2. PostgreSQL

### 2. Pasos previos

Para que el output de la función sea "copiable y pegable", tenemos que acceder a postgres así:

```sql
psql -At
```

!!! Warning

    Si no se hace esto, osea si se accede de forma normal, el output de la función saldrá con signos `+` al final de las líneas y no será "copiable y pegable"

Las funciones en PostgreSQL se almacenan según la base de datos a la que estemos conectados, así que nos conectamos a la base de datos sobre la que queramos usar la función:

```sql
\c scott
```

### 2. Código

```sql
CREATE OR REPLACE FUNCTION generador_script_quitar_privilegios(p_tabla text)
RETURNS text AS $$
DECLARE
    c_usuarios CURSOR FOR
        SELECT grantee 
        FROM information_schema.role_table_grants 
        WHERE privilege_type = 'DELETE' 
            AND table_name = p_tabla;
    v_output text := '';
BEGIN
    FOR i IN c_usuarios LOOP
        v_output := v_output || 'REVOKE DELETE ON ' || p_tabla || ' FROM ' || i.grantee || ';' || E'\n';
    END LOOP;
    RETURN v_output;
END;
$$ LANGUAGE plpgsql;
```

### 2. Comprobaciones

Lanzo la consulta que genera el script y muestro el output:

```sql
scott=# SELECT generador_script_quitar_privilegios('dept');
REVOKE DELETE ON dept FROM postgres;
REVOKE DELETE ON dept FROM scott;
```

Ya sólo tendríamos que copiar y pegar el resultado, y quitaríamos los privilegios.

## 3. MariaDB

### 3. Pasos previos

Es necesario acceder así:

```sql
sudo mariadb -Nsr
```

También que entre en la base de datos a usar:

```sql
USE scott;
```

Para que la prueba luego funcione, añado el siguiente privilegio de borrado sobre la tabla a probar de SCOTT:

```sql
GRANT DELETE ON dept TO 'scott'@'%' IDENTIFIED BY '1234';
flush privileges;
```

### 3. Código

```sql
DELIMITER //
CREATE OR REPLACE FUNCTION generador_script_quitar_privilegios(p_tabla text)
RETURNS text
BEGIN
    DECLARE v_output text default '';
    DECLARE done INT DEFAULT FALSE;
    DECLARE usuario TEXT;

    DECLARE c_usuarios CURSOR FOR 
        SELECT user
        FROM mysql.tables_priv
        WHERE table_priv = 'delete'
            AND table_name = p_tabla
            AND db = 'scott';

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN c_usuarios;

    REPEAT
        FETCH c_usuarios INTO usuario;
        IF NOT done THEN
            SET v_output = CONCAT(v_output, 'REVOKE DELETE ON ', p_tabla, ' FROM ', usuario, ';' , '\n');
        END IF;
    UNTIL done
    END REPEAT;

    CLOSE c_usuarios;

    RETURN v_output;
END //
DELIMITER ;
```

### 3. Comprobaciones

Lanzo la consulta que genera el script y muestro el output:

```sql
MariaDB [scott]> SELECT generador_script_quitar_privilegios('dept');
REVOKE DELETE ON dept FROM scott;
```

Ya sólo tendríamos que copiar y pegar el resultado, y quitaríamos los privilegios.
