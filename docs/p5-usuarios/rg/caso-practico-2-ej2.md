# Ejercicio 2 (Oracle)

## Enunciado

Muestra el texto de la última sentencia SQL que se ejecutó en el servidor, junto con el número de veces que se ha ejecutado desde que se cargó en el Shared Pool y el tiempo de CPU empleado en su ejecución

## Código

```sql
SELECT DISTINCT sql_text,executions,cpu_time
FROM v$sqlarea
ORDER BY first_load_time DESC
FETCH FIRST 1 ROWS ONLY;
```

## Comprobación

Podemos comprobar que la última sentencia que se ejecutó fue "select * from pelicula", se ha ejecutado 1 vez desde que se cargó en el Shared Pool y el tiempo usado ha sido 21710.

![prueba](/img/capturas-arantxa/88.png)
