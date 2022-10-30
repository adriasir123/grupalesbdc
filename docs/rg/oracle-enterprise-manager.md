# Oracle Enterprise Manager

Primero comenzaremos mirando el puerto por el cual podremos acceder a Oracle Enterprise, para ello tenemos que conectarnos a través de sys:

![oracle-ee](/img/alumno4/oracle-express-1.png)

Puede ocurrir que no tengamos el puerto activo, lo cual se mostraría con un 0, ingresaremos el siguiente comando:
`exec dbms_xdb_config.sethttpsport(5500)`

Ahora vamos a sqlplus y vamos a especificar una contraseña al usuario sys, de lo contrario no podremos conectarnos:

`alter user sys identified by sys;`

![oracle-ee](/img/alumno4/oracle-express-2.png)

Ahora especificando la ip y el puerto https://192.168.122.20:5500/em, podemos acceder al login.


![oracle-ee](/img/alumno4/oracle-express-3.png)

Tenemos acceso a Oracle Enterprise y podremos ver todos los recursos que se están utilizando en el sistema.

El enlace es en la red local con lo cual podemos acceder a Oracle enterprise desde cualquier web de nuestr área local.