# Ejercicio 1 (Oracle)

## Enunciado

La vida de un DBA es dura. Tras pedirlo insistentemente, en tu empresa han contratado una persona para ayudarte. Decides que se encargará de las siguientes tareas:

- Resetear los archivos de log en caso de necesidad.
- Crear funciones de complejidad de contraseña y asignárselas a  usuarios.
- Eliminar la información de rollback. (este privilegio podrá pasarlo a quien quiera)
- Modificar información existente en la tabla dept del usuario scott. (este privilegio podrá pasarlo a quien quiera)
- Realizar pruebas de todos los procedimientos existentes en la base de datos.
- Poner un tablespace fuera de línea.

Crea un usuario llamado Ayudante y, sin usar los roles predefinidos de ORACLE, dale  los privilegios mínimos para que pueda resolver dichas tareas.

Pista: Si no recuerdas el nombre de un privilegio, puedes buscarlo en el diccionario de datos.

## Realización

Vamos a crear un usuario llamado ayudante:

```sql
CREATE USER ayudante identified by ayudante;
```

Ahora vamos a crear el rol:

```sql
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
```

## Comprobaciones

Aquí vamos a ver que con el usuario ayudante gracias al rol que ahora tenemos podemos ver las tablas de SCOTT.DEPT:

![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-1.png)

Ahora vamos a hacer una prueba de perfil en el cual haré que incremente el límite de sesiones por usuario:

![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-1-1.png)

Aquí incrementaré el tiempo de conexión del usuario a 10 minutos:

![prueba1](/img/capturas-antonio/prueba-funcionamiento-caso2-ejercicio-1-2.png)
