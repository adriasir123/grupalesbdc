# Ejercicio 4 PostgreSQL

## Pasos previos

Modifico el siguiente parámetro en `/etc/postgresql/13/main/postgresql.conf`:

```sql
log_hostname = on
```

Reinicio:

```sql
sudo systemctl restart postgresql
```

!!! Info

    Es necesario activar ese parámetro para que luego aparezca el hostname

## Código

```sql
CREATE OR REPLACE PROCEDURE MostrarSesionesUsuario (p_usuario VARCHAR)
AS $$
DECLARE
    c_sesiones CURSOR FOR 
        SELECT client_hostname,backend_start,application_name
        FROM pg_stat_activity
        WHERE usename = p_usuario;
    v_hora VARCHAR;
BEGIN
    FOR i IN c_sesiones LOOP
        v_hora := i.backend_start;
        SELECT SUBSTRING(v_hora from 12 for 8) INTO v_hora;
        RAISE NOTICE 'Hora de comienzo: %', v_hora;
        RAISE NOTICE 'Hostname: %', i.client_hostname;
        RAISE NOTICE 'Programa: %', i.application_name;
        RAISE NOTICE '----------------';
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

## Comprobaciones

Muestro las sesiones del usuario `postgres`:

```sql
postgres=# CALL MostrarSesionesUsuario('postgres');
NOTICE:  Hora de comienzo: 09:48:14
NOTICE:  Hostname: <NULL>
NOTICE:  Programa: 
NOTICE:  ----------------
NOTICE:  Hora de comienzo: 09:48:57
NOTICE:  Hostname: <NULL>
NOTICE:  Programa: psql
NOTICE:  ----------------
NOTICE:  Hora de comienzo: 09:58:55
NOTICE:  Hostname: servidorpostgresql1
NOTICE:  Programa: psql
NOTICE:  ----------------
```

La última sesión es la nuestra.
