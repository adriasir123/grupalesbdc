# Ejercicio 1

## 1. (ORACLE) La vida de un DBA es dura. Tras pedirlo insistentemente, en tu empresa han contratado una persona para ayudarte. Decides que se encargará de las siguientes tareas:

- Resetear los archivos de log en caso de necesidad.
- Crear funciones de complejidad de contraseña y asignárselas a  usuarios.
- Eliminar la información de rollback. (este privilegio podrá pasarlo a quien quiera)
- Modificar información existente en la tabla dept del usuario scott. (este privilegio podrá pasarlo a quien quiera)
- Realizar pruebas de todos los procedimientos existentes en la base de datos.
- Poner un tablespace fuera de línea.

Crea un usuario llamado Ayudante y, sin usar los roles predefinidos de ORACLE, dale  los privilegios mínimos para que pueda resolver dichas tareas.

Pista: Si no recuerdas el nombre de un privilegio, puedes buscarlo en el diccionario de datos.


CREATE USER ayudante identified by ayudante;

CREATE ROLE rol_ayudante;

GRANT ALTER SYSTEM TO rol_ayudante;
GRANT CREATE SESSION TO rol_ayudante;
GRANT SELECT ON v_$log TO rol_ayudante;


GRANT CREATE ANY PROCEDURE TO rol_ayudante;
GRANT ALTER PROFILE TO rol_ayudante;

GRANT DROP ROLLBACK SEGMENT TO rol_ayudante WITH ADMIN OPTION;
GRANT FLASHBACK ANY TABLE TO rol_ayudante WITH ADMIN OPTION;

GRANT SELECT, UPDATE, DELETE ON scott.dept TO rol_ayudante;

GRANT EXECUTE ANY PROCEDURE TO rol_ayudante;

GRANT ALTER TABLESPACE TO rol_ayudante;


GRANT rol_ayudante TO ayudante;


![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-1.png)

![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-1-1.png)

![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-1-2.png)



create tablespace PRUEBA2 datafile 'pruebaantonio2.dat' size 10M;

CREATE PROFILE perfil_prueba1 LIMIT SESSIONS_PER_USER 5;


