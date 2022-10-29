# SQL Developer

## 1. Instalación

En mi máquina Windows 10 descargo SQL Developer de la página oficial de Oracle.

https://www.oracle.com/database/sqldeveloper/technologies/download/

Una vez descargado, se descomprime el .zip y en la carpeta creada hacemos click en **sqldeveloper**.

![sqldeveloper](/img/capturas-arantxa/53.png)

Nos aparecerá el siguiente recuadro, en el que se nos preguntará si queremos importar las preferencias de una instalación anterior. Como en nuestro caso no teníamos instalación hacemos click en **"No"**.

![sqldeveloper2](/img/capturas-arantxa/54.png)

> Podemos crear un acceso directo en el escritorio para que nos sea más fácil acceder a sqldeveloper la próxima vez que encendamos el ordenador.

Ya nos aparecerá la ventana del programa.

![sqldeveloper3](/img/capturas-arantxa/55.png)


## 2. Conexión a base de datos

La conexión a la base de datos se puede hacer de dos formas, de forma manual o cargando un archivo TNS Names.

Con TNS Names habría que dejar el contenido de tnsnames.ora de la siguiente manera, para que se conecte a nuestro servidor Oracle:

```
ORCLCDB=
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.122.98)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = ORCLCDB)
    )
  )
```

Nosotros lo vamos a hacer de forma manual. En SQL Developer presionamos en **"Crear una conexión manualmente"** o le damos al símbolo de **+**.

![sqldeveloper4](/img/capturas-arantxa/56.png)

Nos aparecerá una ventana para crear nuevas conexiones. En esta ventana habrán diferentes valores que tendremos que rellenar. 

- Name: escribiremos el nombre que le queremos dar a la conexión creada. Nosotros la hemos llamado **"Conexion-Oracle19c-Debian"**.
- Usuario: escribimos el nombre de usuario de la base de datos con el que nos conectaremos, en mi caso **"admin"**.
- Contraseña: la contraseña del usuario con el que accederemos. 
- Rol: esta opción la dejamos por defecto si vamos a entrar con un usuario normal.
- Tipo de conexión: dejamos la opción básica.
- Nombre del Host: escribimos la ip del servidor de base de datos. En nuestro caso es **"192.168.122.98"**.
- Puerto: ponemos el puerto del servidor. Por defecto suele ser **1521**, si en el servidor se ha cambiado el puerto habrá que tenerlo en cuenta. 
- SID: escribimos nuestro SID, que en nuestro caso es **"ORCLCDB"**.

Quedaría de la siguiente manera.

![sqldeveloper5](/img/capturas-arantxa/57.png)

Clickamos en la opción **Probar** y en *Estado* debría poner "Correcto". Si aparece error será porque no tenemos bien configurado algo en el servidor o nos hemos equivocado en algún dato.

![sqldeveloper6](/img/capturas-arantxa/58.png)

Le damos a **Conectar** y ya nos aparecerá a la izquierda la base de datos con sus tablas. Si seleccionamos una tabla podremos ver sus columnas.

![sqldeveloper7](/img/capturas-arantxa/59.png)

Con SQL Developer podremos modificar todos los valores de la base de datos, crear nuevas tablas, modificar columnas, añadir o borrar datos, etc. de una forma gráfica.

![sqldeveloper8](/img/capturas-arantxa/60.png)
