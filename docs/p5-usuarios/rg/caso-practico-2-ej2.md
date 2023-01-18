# Ejercicio 2

## 2. (ORACLE) Muestra el texto de la última sentencia SQL que se ejecutó en el servidor, junto con el número de veces que se ha ejecutado desde que se cargó en el Shared Pool y el tiempo de CPU empleado en su ejecución

```sql
select distinct sql_text, executions, CPU_TIME
from v$sqlarea
order by first_load_TIME desc
fetch first 1 rows only;
```

### Comprobación

Podemos comprobar que la última sentencia que se ejecutó fue "select * from pelicula", se ha ejecutado 1 vez desde que se cargó en el Shared Pool y el tiempo usado ha sido 21710.

![prueba](/img/capturas-arantxa/88.png)