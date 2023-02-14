# Parte D

Para permitir que los programadores tengan acceso a la tabla "Prueba" y puedan ceder ese derecho y el de conectarse a la base de datos a los usuarios que ellos quieran, se pueden seguir varios pasos:
    
1- Crear un rol de programadores con los privilegios necesarios para acceder a la tabla "Prueba" y conectarse a la base de datos. Por ejemplo:

```sql
SQL> CREATE ROLE programadores;

SQL> GRANT CONNECT, SELECT, INSERT, UPDATE, DELETE ON Prueba TO programadores;
Rol creado.
```

2- Asignar el rol de programadores a los programadores deseados. Por ejemplo:

```sql
GRANT programadores TO programador1, programador2, programador3;
```

3- Permitir que los programadores asignen el rol de programadores y el privilegio de conectarse a la base de datos a otros usuarios. Por ejemplo:
GRANT CREATE SESSION, GRANT ANY ROLE TO programadores;

Con estos pasos se permite que los programadores tengan acceso a la tabla "Prueba" y puedan ceder ese derecho y el de conectarse a la base de datos a los usuarios que ellos quieran. 

Sin embargo, es importante tener en cuenta que esta configuración puede representar un riesgo de seguridad si no se administra adecuadamente, ya que los programadores tendrán acceso a la tabla "Prueba" y podrán otorgar acceso a otros usuarios sin supervisión.