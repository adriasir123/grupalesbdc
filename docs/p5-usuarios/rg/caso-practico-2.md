# Caso práctico 2

## 1. (ORACLE) La vida de un DBA es dura. Tras pedirlo insistentemente, en tu empresa han contratado una persona para ayudarte. Decides que se encargará de las siguientes tareas:

- Resetear los archivos de log en caso de necesidad.
- Crear funciones de complejidad de contraseña y asignárselas a  usuarios.
- Eliminar la información de rollback. (este privilegio podrá pasarlo a quien quiera)
- Modificar información existente en la tabla dept del usuario scott. (este privilegio podrá pasarlo a quien quiera)
- Realizar pruebas de todos los procedimientos existentes en la base de datos.
- Poner un tablespace fuera de línea.

Crea un usuario llamado Ayudante y, sin usar los roles predefinidos de ORACLE, dale  los privilegios mínimos para que pueda resolver dichas tareas.

Pista: Si no recuerdas el nombre de un privilegio, puedes buscarlo en el diccionario de datos.



## 2. (ORACLE) Muestra el texto de la última sentencia SQL que se ejecuto en el servidor, junto con el número de veces que se ha ejecutado desde que se cargó en el Shared Pool y el tiempo de CPU empleado en su ejecución.



## 3. (ORACLE, Postgres) Realiza un procedimiento que reciba dos nombres de usuario y genere un script que asigne al primero los privilegios de inserción y modificación sobre todas las tablas del segundo, así como el de ejecución de cualquier procedimiento que tenga el segundo usuario.



## 4. (ORACLE) Realiza un procedimiento que genere un script que cree un rol conteniendo todos los permisos que tenga el usuario cuyo nombre reciba como parámetro, le hayan sido asignados a aquél directamente o a traves de roles. El nuevo rol deberá llamarse BackupPrivsNombreUsuario.