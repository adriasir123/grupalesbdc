# Ejercicio 4

## Añade una columna email en la tabla Clientes y rellénala con datos consistentes. Realiza un trigger que cuando se actualice la tabla Participaciones para introducir los resultados de la carrera envíe un correo electrónico a todos los apostantes del caballo ganador informándoles del importe apostado y el beneficio obtenido.


```
ALTER TABLE clientes ADD email varchar2(50);


UPDATE clientes SET email = 'mariaromeroangulo@correo.com' WHERE dni = '28841115N';
UPDATE clientes SET email = 'johnsmith@correo.com' WHERE dni = 'Z4128090D';
UPDATE clientes SET email = 'joseltorresandrades@correo.com' WHERE dni = '41500351W';
UPDATE clientes SET email = 'desaconnif@correo.com' WHERE dni = 'X5339679E';
UPDATE clientes SET email = 'mayumiozaki@correo.com' WHERE dni = '18498310P';
UPDATE clientes SET email = 'pacojovercobos@correo.com' WHERE dni = '02411561B';
```

Para poder enviar correos desde PLSQL seguir los siguientes pasos conectándose como sysdba.

```
@$ORACLE_HOME/rdbms/admin/utlmail.sql

@$ORACLE_HOME/rdbms/admin/utlsmtp.sql

@$ORACLE_HOME/rdbms/admin/prvtmail.plb

ALTER SYSTEM SET smtp_out_server='smtp.gmail.com:587' SCOPE=BOTH;

grant execute on UTL_MAIL to ADMIN;


BEGIN
  DBMS_NETWORK_ACL_ADMIN.CREATE_ACL(
    acl => 'acl_mail.xml',
    description => 'Permissions to access mail',
    principal => 'ADMIN',
    is_grant => true,
    privilege => 'connect'
  );
  COMMIT;
END;
/

BEGIN
  DBMS_NETWORK_ACL_ADMIN.ASSIGN_ACL (
    acl => 'acl_mail.xml',
    host => 'smtp.gmail.com',
    lower_port => 587,
    upper_port => 587
  );
  COMMIT;
END;
/

```


**Creación del trigger:**

```
CREATE OR REPLACE TRIGGER enviar_correo_clientes
AFTER INSERT OR UPDATE ON participaciones
FOR EACH ROW
DECLARE
    CURSOR c_clientes IS
        SELECT dni, nombre, apellido1, email
        FROM clientes
        WHERE dni in (select dniCliente from apuestas where codigoCaballo=:NEW.codigoCaballo AND codigoCarrera = :NEW.codigoCarrera);

    v_importeApostado apuestas.importeApostado%TYPE;
    v_tantoauno apuestas.tantoAUno%TYPE;
    v_beneficios NUMBER;

BEGIN
    IF :NEW.posicionFinal = 1 THEN
        FOR cliente IN c_clientes LOOP

            SELECT importeApostado,tantoAUno
            INTO v_importeApostado,v_tantoAUno
            FROM apuestas
            WHERE codigoCaballo=:NEW.codigoCaballo AND dniCliente=cliente.dni;

            v_beneficios := (v_tantoAUno/100) * v_importeApostado;

            -- Enviar correo electrónico a cada cliente
            UTL_MAIL.SEND(
                sender => 'hipodromo@gmail.com',
                recipients => cliente.email,
                subject => 'Resultados de la carrera',
                message => 'Estimado/a ' || cliente.nombre || ' ' || cliente.apellido1 || ', ¡su caballo ha ganado la carrera!
                Importe apostado: ' || v_importeApostado || '
                Beneficio obtenido: ' || v_beneficios || '
                Gracias por su apuesta!
                Atentamente,
                El equipo del hipódromo.',
                mime_type => 'text/plain; charset=us-ascii'
            );
        END LOOP;
        CLOSE c_clientes;
        dbms_output.put_line('El correo se ha enviado con éxito');
    END IF;
END enviar_correo_clientes;
/
```










**Prueba:**

```
insert into clientes values ('49034862N', 'Arantxa', 'Fernández', '', 'Avenida Ramón y Cajal', 'Dos Hermanas', 'Sevilla', '628806858', 'ara.fer.mor@gmail.com');

insert into carrerasProfesionales  values (13, to_date('11-12-2022 12:00' , 'DD-MM-YYYY HH24:MI'), 6125, 450, '01-01-2020', '01-06-2020');


insert into apuestas values ('49034862N',13 ,3 ,300 ,50.15);

insert into participaciones values(13, 3, 'Y6857984L', 1, 1);
```








BEGIN
  UTL_MAIL.SET_SERVER (
    host => 'smtp.gmail.com',
    port => 587
  );
  

BEGIN
  UTL_MAIL.SEND (
    sender => 'ara.fer.mor@gmail.com',
    recipients => 'ara.fer.mor@gmail.com',
    subject => 'Prueba',
    message => 'Esto es un mensaje de prueba.'
  );
END;
/

