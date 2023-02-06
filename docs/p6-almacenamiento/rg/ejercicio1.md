# Ejercicio 1

## Cread un índice para la tabla EMP de SCOTT que agilice las consultas por nombre de empleado en un tablespace creado específicamente para índices. ¿Dónde deberiáis ubicar el fichero de datos asociado? ¿Cómo se os ocurre que podriáis probar si el índice resulta de utilidad?

Vamos a emplear un procedimiento que genere nombres en el usuario scott y que nos permita crear 100000 empleados, tras esto haremos que la fila ename sea char (1000)


CREATE OR REPLACE PROCEDURE add_100000_emp
AS
v_nombre emp.ename%TYPE;
BEGIN
    FOR i IN 10001..100000 LOOP
        v_nombre := 'e' || i;
        INSERT INTO scott.emp VALUES (i, v_nombre, 'ANTONIO', 7902, TO_DATE('01/01/1980', 'DD/MM/YYYY'), 1000, 1000 , 20);
    END LOOP;
END;
/

explain plan for select ename from emp where ename = 'e92153';

select id,operation,cpu_cost,cost,time,object_name from PLAN_TABLE;


![wireguard-site-1.png](/img/capturas-antonio/grupal1-1.png)


Una vez hecho esto vamos a montar un disco rápido en el sistema donde albergará el tablespace del índice para que funcione lo más rápido posible y la búsqueda del nombre sea mucho más eficiente.

![wireguard-site-1.png](/img/capturas-antonio/grupal1-2.png)

Vamos a montar una unidad en systemd en el cual haremos persistente el montaje del nuevo disco.

nano /etc/systemd/system/opt-oracle-oradata-ORCLCDB-vdb.mount

```bash
[Unit]
Description=Disco Oracle     

[Mount]
What=/dev/vdb
Where=/opt/oracle/oradata/ORCLCDB/vdb
Type=ext4
Options=defaults

[Install]
WantedBy=multi-user.target
```

systemctl daemon-reload
systemctl enable opt-oracle-oradata-ORCLCDB-vdb.mount
systemctl start opt-oracle-oradata-ORCLCDB-vdb.mount

![wireguard-site-1.png](/img/capturas-antonio/grupal1-3.png)

Ahora entramos como administrador a sqlplus y creamos el tablespace en el disco que hemos montado.

```sql
create tablespace indices
datafile 'ename.dbf'
size 10M
autoextend on;
```

```sql
set autotrace on;
     create tablespace indices
  2  datafile 'ename.dbf'
  3  size 10M
  4  autoextend on;

Tablespace creado.

```

```
CREATE INDEX nombres_empleados ON emp(ename)
      TABLESPACE indices;
```


![wireguard-site-1.png](/img/capturas-antonio/grupal1-4.png)



