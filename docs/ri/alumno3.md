# Alumno 3. Arantxa Fernández Morató

Tras la instalación de cada servidor,  debe crearse una base de datos con al menos tres tablas o colecciones y poblarse de datos adecuadamente. Debe crearse un usuario y dotarlo de los privilegios necesarios para acceder remotamente a los datos.
Los clientes deben estar siempre en máquinas diferentes de los respectivos servidores a los que acceden.
Se documentará todo el proceso de configuración de los servidores.
Se aportarán pruebas del funcionamiento remoto de cada uno de los clientes.
Se aportará el código de las aplicaciones realizadas y prueba de funcionamiento de las mismas.

    • Instalación de un servidor Oracle 19c sobre Debian, otro Postgres, otro MySQL y otro de MongoDB y configuración para permitir el acceso remoto desde la red local.
    • Prueba desde un cliente remoto de SQL*Plus.
    • Realización de una aplicación web en cualquier lenguaje que conecte con el servidor Postgres tras autenticarse y muestre alguna información almacenada en el mismo.

## 1. MySQL

### 1.1 Instalación

En el servidor actualizamos los repositorios e instalamos **gnupg** y **wget**, si no estuvieran ya instalados.

`sudo apt update`

`sudo apt install -y gnupg wget`

Utilizando wget descargamos el paquete de mysql del repositorio oficial.

`wget https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb`

Procedemmos a instalar el paquete descargado.

`sudo dpkg -i mysql-apt-config_0.8.22-1_all.deb`

Nos aparecerá una ventana que nos muestra la versión de MySQL que se instalará, nos informa de que las herramientas y conectores están activados y la opción de software de prueba está desactivada. Presionamos *Aceptar* para continuar.

![instalacion-mysql](/docs/img/capturas-arantxa/1.png)

Volvemos a actualizar los repositorios.

`sudo apt update`

Ya podremos instalar MySQL server en Debian 11.

`sudo apt install -y mysql-server`

Durante la instalación nos preguntará la contraseña del usuario administrador root.

![instalacion-mysql2](/docs/img/capturas-arantxa/2.png)

Nos aparece un menje de disponibilidad de un nuevo plugin de autenticación. Presionamos *Acepta*.

![instalacion-mysql3](/docs/img/capturas-arantxa/3.png)

Dejamos seleccionado el plugin de autenticación recomendado.

![instalacion-mysql4](/docs/img/capturas-arantxa/4.png)

Terminada la instalación pasaremos a entrar con el usuario root.

`mysql -u root -p`

![entrando-mysql](/docs/img/capturas-arantxa/5.png)


### 1.2 Creación usuario

Creamos un usuario y contraseña para el administrador.

`create user admin identified by admin;`

Le damos todos los privilegios.

`grant all privileges on *.* to admin with grant option;`

`flush privileges;`




## 2. PostgreSQL

### 2.1 Instalación

Instalamos PostgreSQL en Debian 11 directamente con **apt**.

`sudo apt install postgresql`




## 3. Oracle

### 3.1 Instalación

Descargamos Oracle 19c desde su página oficial. Si se hac con wget nos dará error a la hora de pasarlo a formato deb con alien. Vamos a su página y lo descargamos.

https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html

Vemos que está en formato rpm, así que utilizaremos la herramienta **alien** para pasarlo a formato deb.

`sudo apt install alien libaio1 unixodbc`

Usamos alien para convertir el paquete a deb. Este proceso puede ser un poco lento.

`sudo alien --scripts -d oracle-database-ee-19c-1.0-1.x86_64.rpm`

Ahora vamos a crear un script en sbin que tendrá el siguiente contenido.

`sudo nano /sbin/chkconfig`

```
#!/bin/bash
# Oracle XE Installer chkconfig for Debian 11
file=/etc/init.d/oracle-xe
if [[ ! `tail -n1 $file | grep INIT` ]]; then
echo >> $file
echo '### BEGIN INIT INFO' >> $file
echo '# Provides: OracleXE' >> $file
echo '# Required-Start: $remote_fs $syslog' >> $file
echo '# Required-Stop: $remote_fs $syslog' >> $file
echo '# Default-Start: 2 3 4 5' >> $file
echo '# Default-Stop: 0 1 6' >> $file
echo '# Short-Description: Oracle 19c Express Edition' >> $file
echo '### END INIT INFO' >> $file
fi
update-rc.d oracle-xe defaults 80 01
```

![script](/docs/img/capturas-arantxa/6.png)

Damos los permisos al fichero creado.

`sudo chmod 777 /sbin/chkconfig`

Para que Oracle funcione se debe configurar el kernel por lo que añadiremos lo siguiente:

`sudo nano /etc/sysctl.d/60-oracle.conf`

```
# Oracle 19c XE kernel parameters  
fs.file-max=6815744  
net.ipv4.ip_local_port_range=9000 65000  
kernel.sem=250 32000 100 128
kernel.shmmax=536870912
```

![conf](/docs/img/capturas-arantxa/7.png)

Cargamos los parámetros sin tener que reiniciar el sistema.

`sudo systemctl start procps.service`


Y ya podremos instalar Oracle 19c.

`sudo dpkg -i oracle-database-ee-19c_1.0-2_amd64.deb`

![instalacion-oracle](/docs/img/capturas-arantxa/8.png)




### 3.2 Configuración

Creación de la base de datos y configuración de la contraseña del administrador.

`sudo /etc/init.d/oracledb_ORCLCDB-19c configure`


##### Posibles errores al hacer configure

_**ERROR 1. Fallo en la comprobación de la memoria**_

A mi me aparece el siguiente error.

![error-oracle](/docs/img/capturas-arantxa/11.png)

Para solucionarlo configuramos el fichero **/etc/init.d/oracledb_ORCLCDB-19c**. Donde pone **configure_perform** añadimos lo siguiente a la línea **$SU**, como se ve en la captura.

`-J-Doracle.assistants.dbca.validate.ConfigurationParams=false`

![conf-oracle](/docs/img/capturas-arantxa/9.png)

![conf-oracle](/docs/img/capturas-arantxa/10.png)

_**ERROR 2. Fallo en la configuración de la red**_

También me apareció el siguiente problema.

![error-oracle2](/docs/img/capturas-arantxa/12.png)

Para solucionarlo hay que instalar las net-tools.

`sudo apt install net-tools`

Y para terminar añadir a **/etc/hosts** nuestra ip, en mi caso **192.168.122.98** para *debian-oracle*. Quedaría de la siguiente forma.

`sudo nano /etc/hosts`

```
127.0.0.1 localhost
192.168.122.98 debian-oracle
```

![conf-ip](/docs/img/capturas-arantxa/13.png)

Ya deberíamos poder realizar la configuración.

![configuracion-proceso](/docs/img/capturas-arantxa/14.png)


### 3.3 Terminar configuración

Añadimos las variables de entorno bash.

`sudo nano ~/.bashrc`

Al final del fichero, en la última línea añadir lo siguiente:

```
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export ORACLE_SID=XE
export ORACLE_BASE=/opt/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
```

![variables-entorno](/docs/img/capturas-arantxa/15.png)

Iniciamos las variables de entorno.

`. ~/.profile`

Arrancamos el servicio Oracle.

`sudo systemctl start oracledb_ORCLCDB-19c`

Si no conocemos el nombre exacto del servicio buscarlo con:

`sudo systemctl list-unit-files --type service | grep oracle`

![servicio-oracle-activo](/docs/img/capturas-arantxa/16.png)

Podremos entrar a Oracle usando el siguiente comando:

`sqlplus / as sysdba`

Pero nuestro usuario local deberá estar en el grupo **dba**.

`sudo nano /etc/group`

![grupo-dba](/docs/img/capturas-arantxa/17.png)

![acceso-usuario](/docs/img/capturas-arantxa/18.png)


### 3.4 Iniciar la base de datos

Si al inicializar la base de datos aparece el siguiente error es porque no le hemos dado el nombre correcto a la base de datos.

![error-bd-init.ora](/docs/img/capturas-arantxa/20.png)

Cambiamos el nombre ORACLE_SID, al nombre de nuestra base de datos, en este caso ORCLCDB.

`export ORACLE_SID=ORCLCDB`

Volvemos a iniciar las variables de entorno y reiniciamos el proceso.

`. ~/.profile`

`sudo systemctl start oracledb_ORCLCDB-19c`

Además si nos aparece error de que no encuentra los directorios deberemos crear las siguientes carpetas en el directorio base de Oracle, en mi caso **/opt/oracle/product/19c/dbhome_1**.

`sudo mkdir fast_recovery_area`

`sudo mkdir adump`

`sudo chown -R oracle:oinstall fast_recovery_area`

`sudo chown -R oracle:oinstall adump`

Entramos de nuevo como *"sqlplus / as sysdba"* e iniciamos la base de datos con **startup**.

![bd-montada](/docs/img/capturas-arantxa/21.png)
