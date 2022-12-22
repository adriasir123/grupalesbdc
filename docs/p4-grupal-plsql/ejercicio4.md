# Ejercicio 4

> Añade una columna email en la tabla Clientes y rellénala con datos consistentes. Realiza un trigger que cuando se actualice la tabla Participaciones para introducir los resultados de la carrera envíe un correo electrónico a todos los apostantes del caballo ganador informándoles del importe apostado y el beneficio obtenido

## Modificación de la tabla y los datos

```
ALTER TABLE clientes ADD email varchar2(50);


UPDATE clientes SET email = 'mariaromeroangulo@correo.com' WHERE dni = '28841115N';
UPDATE clientes SET email = 'johnsmith@correo.com' WHERE dni = 'Z4128090D';
UPDATE clientes SET email = 'joseltorresandrades@correo.com' WHERE dni = '41500351W';
UPDATE clientes SET email = 'desaconnif@correo.com' WHERE dni = 'X5339679E';
UPDATE clientes SET email = 'mayumiozaki@correo.com' WHERE dni = '18498310P';
UPDATE clientes SET email = 'pacojovercobos@correo.com' WHERE dni = '02411561B';
```

## Envío de correos mediante UTL_MAIL

Para poder enviar correos desde PLSQL seguir los siguientes pasos conectándose como sysdba.

```
@$ORACLE_HOME/rdbms/admin/utlmail.sql

@$ORACLE_HOME/rdbms/admin/prvtmail.plb

ALTER SYSTEM SET smtp_out_server='localhost' SCOPE=BOTH;

BEGIN
  DBMS_NETWORK_ACL_ADMIN.CREATE_ACL(
    acl => 'aclcorreo.xml',
    description => 'Enviar correos',
    principal => 'ADMIN',
    is_grant => true,
    privilege => 'connect',
    start_date => SYSTIMESTAMP,
    end_date => NULL
  );
  COMMIT;
END;
/

BEGIN
  DBMS_NETWORK_ACL_ADMIN.ASSIGN_ACL (
    acl => 'aclcorreo.xml',
    host => '*',
    lower_port => NULL,
    upper_port => NULL
  );
  COMMIT;
END;
/

grant execute on UTL_MAIL to ADMIN;
```

A continuación ya podremos conectarnos con ADMIN y enviar un correo de prueba:

```
BEGIN
  UTL_MAIL.SEND (
    sender => 'ara.fer.mor@gmail.com',
    recipients => 'ara.fer.mor@gmail.com',
    subject => 'Prueba',
    message => 'Esto es un mensaje de prueba.'
  );
END;
/
```

![correo-prueba](/img/capturas-arantxa/82)

## Creación del trigger

```
CREATE OR REPLACE TRIGGER enviar_correo_clientes
AFTER INSERT OR UPDATE ON participaciones
FOR EACH ROW
DECLARE
    -- Creo un cursor para recorrer todos los clientes que han apostado por el caballo insertado y la carrera
    CURSOR c_clientes IS
        SELECT dni, nombre, apellido1, email
        FROM clientes
        WHERE dni in (select dniCliente from apuestas where codigoCaballo=:NEW.codigoCaballo AND codigoCarrera = :NEW.codigoCarrera);

    -- Declaración de variables
    v_importeApostado apuestas.importeApostado%TYPE;
    v_tantoauno apuestas.tantoAUno%TYPE;
    v_beneficios NUMBER;

BEGIN
    -- Si el caballo ha ganado la carrera, enviar correo a todos los clientes que hayan apostado por él
    IF :NEW.posicionFinal = 1 THEN
        FOR cliente IN c_clientes LOOP

            -- Obtener el importe apostado y el tanto a uno de cada cliente 
            SELECT importeApostado,tantoAUno
            INTO v_importeApostado,v_tantoAUno
            FROM apuestas
            WHERE codigoCaballo=:NEW.codigoCaballo AND dniCliente=cliente.dni;

            -- Calcular el beneficio obtenido por cada cliente
            v_beneficios := (v_tantoAUno/100) * v_importeApostado;

            -- Enviar correo electrónico a cada cliente
            UTL_MAIL.SEND(
                sender => 'ara.fer.mor@gmail.com@gmail.com',
                recipients => cliente.email,
                subject => 'Resultados de la carrera',
                message => 'Estimado/a ' || cliente.nombre || ' ' || cliente.apellido1||',' ||CHR(10)|| 'su caballo ha ganado la carrera!!!'||CHR(10)||CHR(10)||'El importe apostado ha sido: ' || v_importeApostado || CHR(10)||'Y el beneficio obtenido: ' || v_beneficios || CHR(10)||CHR(10)|| 'Gracias por su apuesta!' ||CHR(10)||CHR(10)||'Atentamente,'||CHR(10)||'El equipo del hipodromo.',
                mime_type => 'text/plain; charset=us-ascii'
            );
        END LOOP;
    END IF;
END enviar_correo_clientes;
/
```

## Prueba

```
insert into clientes values ('49034862N', 'Arantxa', 'Fernandez', '', 'Avenida Ramón y Cajal', 'Dos Hermanas', 'Sevilla', '628806858', 'ara.fer.mor@gmail.com');

insert into carrerasProfesionales  values (13, to_date('11-12-2022 12:00' , 'DD-MM-YYYY HH24:MI'), 6125, 450, '01-01-2020', '01-06-2020');


insert into apuestas values ('49034862N',13 ,3 ,300 ,50.15);

insert into participaciones values(13, 3, 'Y6857984L', 1, 1);
```

![inserts](/img/capturas-arantxa/83.png)
![correo-enviado](/img/capturas-arantxa/84.png)

!!! info "**Comandos que me han ayudado a solucionar algunos problemas con UTL_MAIL:**"

  Para ver las ACL creadas y los pivilegios asignados a esas ACLs:
   ```
   select * from dba_network_acls;
   select * from dba_network_acl_privileges;
   ```
   Procedimientos para borrar un privilegio de una ACL y para borrar una ACL:
  ```
  BEGIN
    DBMS_NETWORK_ACL_ADMIN.DELETE_PRIVILEGE(
          acl         => 'aclmail.xml',
          principal   => 'ADMIN');
  END;
  /

  BEGIN
     DBMS_NETWORK_ACL_ADMIN.DROP_ACL(
        acl => 'aclmail.xml');
  END;
  /
  ```
