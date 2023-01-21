# Alumno 4


## Mongodb

### Ejercicio 1: Averigua si existe la posibilidad en MongoDB de limitar el acceso de un usuario a los datos de
una colección determinada.



Primero entramos en la base de datos como administrador, luego entramos en la base de datos que queremos crear:

```
mongo -u admin -p

use ejemplo

db.createCollection("coleccion")

db.coleccion.InsertOne({nombre : "Antonio"})


db.createUser({user: "antonio", pwd: "antonio", roles: ["readWrite"]})

db.revokeRolesFromUser("antonio",[ { "role" : "readWrite", db : "ejemplo" } ])


```


Con esto al haber creado el usuario, se le da con derechos de lectura y escritura por defecto en la base de datos en la que ha sido creada, lo que ocurre es que al hacer un RevoqueRolesFromUser pues ha perdido su rol de readWrite, por tanto no podrá ver las colecciones:

![prueba1](/img/capturas-antonio/prueba-funcionamiento-individual-ejercicio-1-1.png)


Si volvemos a entrar como administrador en el sistema y volvemos a entrar en la base de datos, podemos volver a conceder los permisos:

```
use ejemplo

db.grantRolesToUser("antonio",[ { "role" : "readWrite", db : "ejemplo" } ])
```

Y de esta manera al listar las colecciones podemos ver que estará disponible la que hemos otorgado a través del rol:


![prueba1](/img/capturas-antonio/prueba-funcionamiento-individual-ejercicio-1-2.png)



### Ejercicio 2: Averigua si en MongoDB existe el concepto de privilegio del sistema y muestra las diferencias más importantes con ORACLE.

MongoDB tiene un sistema de autorización basado en roles que se utiliza para controlar el acceso a los recursos de la base de datos, por tanto sí, tiene sistema de privilegios de usuario similar al de Oracle. Los roles se asignan a los usuarios y definen los privilegios de acceso que tienen sobre los recursos de la base de datos.

En comparación con Oracle, MongoDB tiene algunas diferencias importantes en cuanto a los privilegios del sistema:

1. MongoDB no tiene una jerarquía de privilegios como Oracle, donde los usuarios pueden heredar privilegios de otros usuarios o roles.

2. MongoDB no tiene un sistema de vistas de seguridad como Oracle, donde los datos pueden ser filtrados para que solo se muestren los datos a los que un usuario tiene acceso.

3. MongoDB tiene un sistema de autorización basado en roles en vez de a objetos, lo que significa que los privilegios se asignan a los roles en lugar de asignarlos directamente a los objetos de la base de datos como hemos podido comprobar en anteriores ejercicios con Oracle.

4. MongoDB tiene un sistema de autorización en el nivel de campo, lo que significa que se pueden asignar privilegios específicos para el acceso a los campos individuales de un documento como hemos visto en el anterior ejercicio.

5. MongoDB tiene un enfoque más simple en la gestión de privilegios. Los roles se definen en un nivel de base de datos, en lugar de en un nivel de sistema como en ORACLE.

6. MongoDB no tiene una funcionalidad de encriptación de datos nativa como ORACLE, por lo que la encriptación debe ser manejada por el cliente o a través de un complemento.

### Ejercicio 3: Explica los roles por defecto que incorpora MongoDB y como se asignan a los usuarios.

Los roles por defecto que incorpora mongodb son los siguientes:

read:  permite al usuario leer los documentos de una colección.

readWrite: Permite al usuario tanto leer como escribir en documentos de una colección.

dbAdmin:  permite al usuario administrar la base de datos.

userAdmin:  permite al usuario administrar usuarios y roles en una base de datos.

clusterAdmin: permite al usuario realizar tareas administrativas en todo el cluster de MongoDB

backup: permite al usuario realizar copias de seguridad y restaurar datos.

restore: Otorga acceso para restaurar la base de datos.

dbOwner: Este rol otorga acceso total a todas las operaciones y colecciones en la base de datos, incluyendo la capacidad de crear y eliminar colecciones.

readAnyDatabase: Otorga Acceso de lectura a cualquier base de datos.

readWriteAnyDatabase: Otorga acceso de lectura y escritura a cualquier base de datos.

Se le puede asignar un rol a un usuario cuando este es creado, si no se especifica este tendrá por defecto un readWrite.

```
db.createUser({
  user: "antonio",
  pwd: "antonio",
  roles: [{ role: "read", db: "ejemplo" }]
});
```

Ahora bien, si queremos cambiar el rol que queremos que tenga un usuario podemos emplear los comandos de quitar o añadir roles:

```
db.grantRolesToUser("antonio",[ { "role" : "readWrite", db : "ejemplo" } ])

db.revokeRolesFromUser("antonio",[ { "role" : "readWrite", db : "ejemplo" } ])
```

O de una manera alternativa podemos especificar qué rol queremos que tenga el usuario:
```
db.updateUser("antonio", { roles: [{ role: "readWrite", db: "ejemplo" }] });
```

### Ejercicio 4: Explica como puede consultarse el diccionario de datos de MongoDB para saber que roles han sido concedidos a un usuario y qué privilegios incluyen.

Para obtener los roles de un usuario en Mongodb utilizaremos el comando `db.getUser("antonio")` para ver qué roles tiene disponibles y en la base de datos en la que las tiene.

el comando `db.runCommand({usersInfo:"antonio", showPrivileges:true})` mostrará la base de datos que tiene un determinado usuario y luego los roles que posee sobre ella.

```
ejemplo> db.runCommand({usersInfo:"antonio", showPrivileges:true})
{
  users: [
    {
      _id: 'ejemplo.antonio',
      userId: new UUID("722d3e71-2c31-4371-803d-21c4ba8ba622"),
      user: 'antonio',
      db: 'ejemplo',
      mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ],
      roles: [],
      inheritedRoles: [],
      inheritedPrivileges: [],
      inheritedAuthenticationRestrictions: []
    }
  ],
  ok: 1
}

```

Si queremos ver los privilegios que alberga un rol, podemos utilizar el getRole, especificando el parámetro de los privilegios en verdadero:

```
ejemplo> db.getRole("readWrite", { showPrivileges: true })
{
  db: 'ejemplo',
  role: 'readWrite',
  roles: [],
  privileges: [
    {
      resource: { db: 'ejemplo', collection: '' },
      actions: [
        'changeStream',
        'collStats',
        'convertToCapped',
        'createCollection',
        'createIndex',
        'dbHash',
        'dbStats',
        'dropCollection',
        'dropIndex',
        'emptycapped',
        'find',
        'insert',
        'killCursors',
        'listCollections',
        'listIndexes',
        'planCacheRead',
        'remove',
        'renameCollectionSameDB',
        'update'
      ]
    },

```



## Oracle

### Ejercicio 1:

Realiza un procedimiento llamado MostrarObjetosAccesibles que reciba un nombre de usuario y
muestre todos los objetos a los que tiene acceso.

create or replace procedure MostrarObjetosAccesibles (p_usuario varchar2)
IS
  Cursor c_objeto is
  select table_name, privilege, owner from dba_tab_privs where grantee = p_usuario;
  v_nombretabla varchar2(50);
  v_nombretabla_old varchar2(50):=' ';
BEGIN
  dbms_output.put_line('objetos accesibles:');
  for v_objeto in c_objeto loop
    v_nombretabla:=v_objeto.table_name;
    if v_nombretabla_old != v_nombretabla then
      dbms_output.put_line('Tabla: ' || v_objeto.table_name);
      dbms_output.put_line(CHR(9)|| 'Propietario: '|| v_objeto.owner);
      dbms_output.put_line(CHR(9)|| 'privilegios:');
    end if;
    dbms_output.put_line(CHR(9)||CHR(9)|| '- '|| v_objeto.privilege);
    v_nombretabla_old:=v_nombretabla;
end loop;
END;
/


![prueba1](/img/capturas-antonio/prueba-funcionamiento-individual-ejercicio-3-1.png)

### Ejercicio 2:

Realiza un procedimiento que reciba un nombre de usuario, un privilegio y un objeto y nos
muestre el mensaje 'SI, DIRECTO' si el usuario tiene ese privilegio sobre objeto concedido
directamente, 'SI, POR ROL' si el usuario lo tiene en alguno de los roles que tiene concedidos y un 'NO' si el usuario no tiene dicho privilegio.

```
create or replace procedure mostrarobjetoprivilegio (p_usuario varchar2, p_privilegio varchar2, p_objeto varchar2)
IS
v_contador number:=0;
BEGIN
COMPROBARUSUARIO (p_usuario);
SELECT COUNT(*) INTO v_contador FROM DBA_TAB_PRIVS WHERE GRANTEE = p_usuario AND PRIVILEGE = p_privilegio AND TABLE_NAME = p_objeto;
if v_contador = 1 then
    SELECT COUNT(*) INTO v_contador FROM DBA_ROLE_PRIVS WHERE grantee = p_usuario;
    if v_contador = 1 then
        dbms_output.put_line('SI, POR ROL');
    else
        dbms_output.put_line('SI, DIRECTO');
    end if;
else
    dbms_output.put_line('NO');
end if;
END;
/


create or replace procedure COMPROBARUSUARIO (p_usuario varchar2)
IS
v_existe number:=0;
BEGIN
select count(*) into v_existe from DBA_USERS where USERNAME = p_usuario;
if v_existe = 0 then
    raise_application_error(-20112,'No existe el usuario');
end if;
END;
/
```


![prueba1](/img/capturas-antonio/prueba-funcionamiento-individual-ejercicio-4-1.png)