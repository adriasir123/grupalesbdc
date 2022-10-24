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

