# Alumno 4

##Instalación de un servidor Postgresql y su posterior acceso

Comenzaremos actualizando el sistema e instalando postgresql con derechos de privilegio
```
root@debian:/home/usuario# apt update && apt install postgresql -y
```

Comprobamos si el servicio está activo, si no es así lo iniciamos con `sudo systemctl start postgresql`

![postgres](/img/alumno4/postgres-instalacion-1.png)

Tras esto, comprobaremos que el usuario postrgres existe, ya que es el que nos dará el acceso a la base de datos, lo podemos comprobar con el siguiente comando:

`cat /etc/passwd | egrep "postgres"`

Si no tenemos el usuario creado, lo haremos con `sudo useradd postgres`

Con el comando groups podemos ver que ese usuario tiene certificación SSL, con lo cual podrá conectarse a la base de datos ya que su usuario tiene esos derechos:

![postgres](/img/alumno4/postgres-instalacion-2.png)

```
postgres@debian:/home/usuario$ createdb dark-souls
postgres@debian:/home/usuario$ psql
```


con \c dark-souls ingresamos en la base de datos
Ahora procedemos a ingresar las tablas, he escogido las de mi proyecto del año pasado ya que le dediqué mucho tiempo con lo cual le tengo un especial cariño.
https://github.com/Evanticks/BBDD/blob/main/Proyecto_Tercer_Trimestre/Dark_Souls_Postgres.sql



tras esto ingresamos los datos que van en las tablas, el cual se hayan en mi repositorio

Comprobamos los datos con una consulta simple:
```
dark-souls=# select * from armas;
 codarma |      nombre      | fuerza | destreza | inteligencia | rareza | nivel 
---------+------------------+--------+----------+--------------+--------+-------
 001     | Espada Corta     |      8 |       10 |            0 | D      |     5
 002     | Espada Larga     |     10 |       10 |            0 | C      |     8
 003     | Espada Artorias  |     24 |       18 |           20 | S      |    30
 004     | Hacha de Mano    |      8 |        8 |            0 | D      |     6
 005     | Hacha de Gárgola |     14 |       14 |            0 | A      |    15
 006     | Hacha de Demonio |     46 |        0 |            0 | S      |    40
(6 filas)

```

`sudo nano /etc/postgresql/13/main/postgresql.conf`

listen_addresses = '*'
Con esto podré conectarme desde un cliente a un servidor postgres

A continuación a través del fichero **pg_hba.conf** podemos decir qué máquina va a entrar a nuestro servidor, sirve de filtro de conexiones.


![postgres](/img/alumno4/postgres-instalacion-5.png)

Nos vamos a la máquina cliente y hacemos lo siguiete:

```
sudo apt update && sudo apt install postgresql-client -y
```
Ahora nos faltaría saber por qué puerto está escuchando:
![postgres](/img/alumno4/postgres-instalacion-7.png)

luego creamos el usuario en el cual pondremos como parámetro en el cliente:
createuser -s antonio -P


Con el siguiente comando estamos diciendo que se conecte al siguiente host, con el puerto 5432 con  el usuario antonio y la base de datos a la que queremos conectarnos:

psql -h 192.168.122.188 -p 5432 -U antonio -d dark-souls

![postgres](/img/alumno4/postgres-instalacion-6.png)

##Instalación de un servidor Mariadb y su posterior acceso

Procedemos con la instalación con el siguiente comando:

```
sudo apt update && sudo apt install mariadb-server -y
```

sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
procedemos a comentar la línea que nos bloquea el uso al exterior del loopback.

`#bind-address            = 192.168.122.0`

```
sudo service mariadb restart
sudo mysql -u root -p
```
Le otorgamos al siguiente usuario los privilegios sobre la base de datos:

```
create database souls;
GRANT ALL PRIVILEGES ON souls.* TO 'antonio'@'192.168.122.146' IDENTIFIED BY 'antonio' WITH GRANT OPTION;
use souls;
```

Insertamos las tablas y los insert que se hayan en mi github:

https://github.com/Evanticks/BBDD/blob/main/Proyecto_Tercer_Trimestre/Dark_Souls_Mariadb.sql

Ahora procedemos a instalar el cliente en nuestra máquina:
```
sudo apt install mariadb-client
```
![mariadb](/img/alumno4/mariadb-instalacion-2.png)

##Instalación de un servidor Mongodb y su posterior acceso


Comenzaremos ejecutando el siguiente comando:

`sudo apt-get install gnupg -y`

`wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | apt-key add -`

Estos pasos debemos realizarlo para poder tener en el repositorio la base de datos que se irá actualizando, además de poder instalar mongo por apt.

`echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main" | tee /etc/apt/sources.list.d/mongodb-org.list`


Tras hacer un update, procedemos a instalarlo:

`apt-get install mongodb-org -y`


```
systemctl start mongod
systemctl enable mongod
```

Para entrar en la base de datos de mongo:
`mongo`

Accederemos al administrador que tiene los privilegios:
`use admin`

```
db.createUser(
...   {
...     user: "AMP",
...     pwd: "AMP",
...     roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
...   }
... )
```


Ahora vamos a entrar en /etc/mongod.conf y especificaremos lo siguiente:

```
security:
authorization: enabled
```
![mongo](/img/alumno4/mongo-instalacion-2.png)


Ahora procedemos a importar nuestra base de datos guardada en JSON, ya que es el sistema de ficheros con el que trabaja Mongo, así podremos transformarlo en colecciones dentro de la base de datos SOULS que se especifica en el siguiente comando:

```
usuario@debian:~$ mongoimport --db SOULS --collection SOULSWEAPONS --file DarkSoulsWeapons.json --jsonArray --authenticationDatabase admin -u AMP -p AMP
2022-10-29T15:55:57.757+0200	connected to: mongodb://localhost/
2022-10-29T15:55:57.813+0200	140 document(s) imported successfully. 0 document(s) failed to import.
```

Creamos la base de datos SOULS, la colección SOULSWAPONS y dentro los documentos que habrá en la colección.

`mongo -u AMP -p`

Accederemos a la base de datos creada anteriormente con el siguiente comando:

`use SOULS`

![mongo](/img/alumno4/mongo-instalacion-4.png)

Ahora salimos y volvemos a entrar en el archivo de configuración de mongo /etc/mongod.conf

```
net:
  port: 27017
  bindIp: 0.0.0.0
```


  `systemctl restart mongod`


  Ahora procedemos a instalar mongo en el cliente y una vez hecho ese paso procedemos a conectarnos especificando el usuario y la ip del host:

 ` mongo --host 192.168.122.168 -u AMP`


![mongo](/img/alumno4/mongo-instalacion-5.png)


##Conexión de aplicación web a través de Oracle, Pyhton3 y Flask

En los documentos que se hayan en el siguiente repositorio:
https://github.com/Evanticks/Oracle_cx

Podremos acceder a crear la aplicación web a través de python y flask, para realizar esto creamos un entorno de trabajo llamado /env, luego dentro de este podremo instalar con pip los siguientes paquetes:
```
pip install flask
pip install cx_Oracle
```

Tras esto podemos aplicar el siguiente código que realizará lo siguiente:

```
@app.route('/login',methods=["GET","POST"])
def login():
    if request.method=="POST":
        texto=request.form.get("user")
        print ('texto=',texto)
        texto2=request.form.get("pass")
        print ('texto2=',texto2)
        if texto=='antonio' and texto2=='antonio':
            connection=cx_Oracle.connect(
	        user='antonio',
	        password='antonio',
	        dsn='192.168.122.20:1521/ORCLCDB',
	        encoding='UTF-8')
            cursor = connection.cursor()
            cursor.execute("select * from personaje")
            resultado = cursor.fetchall()
            cursor.execute("select nombre from armas")
            resultado2 = cursor.fetchall()
            cursor.execute("select nombre from tesoro")
        else:
            return redirect(url_for('login'))
    else:
        return render_template("login.html")
```

Con esto entraremos en la pantalla de login, si ingresamos nuestro usuario y contraseña de Oracle el cual será antonio, accederemos a la base de datos y mostrárá dos select, uno para listar los datos de la tabla personaje y otro para listar los nombres de las armas, en el html podremos ingresar el resultado generado de la consulta a través de Jinja2.

Ejecutaremos python3 app.py para arrancar el servidor flask que nos dará el acceso a la web

IMPORTANTE: Para realizar esto debemos previamente activar el listener y hacer un startup para levantar la base de datos, podemos comprobarlo antes accediendo desde un cliente remoto por consola.

 ![oracle](/img/alumno4/oracle-web1.png)


Aquí podemos ver que al entrar a través del login, podemos ver las tablas de la base de datos.

 ![oracle](/img/alumno4/oracle-web2.png)