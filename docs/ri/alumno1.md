# Alumno 1

## Escenario

```shell
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 2
    libvirt.memory = 1024
  end

  config.vm.define :servidororacle do |servidororacle|
    servidororacle.vm.box = "debian/bullseye64"
    servidororacle.vm.provider :libvirt do |servidororacle|
      servidororacle.memory = 4096
      servidororacle.cpus = 6
    end
    servidororacle.vm.hostname = "servidororacle"
    servidororacle.vm.network :private_network,
      :libvirt__network_name => "red-oracle",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clienteoracle do |clienteoracle|
    clienteoracle.vm.box = "debian/bullseye64"
    clienteoracle.vm.hostname = "clienteoracle"
    clienteoracle.vm.network :private_network,
      :libvirt__network_name => "red-oracle",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.0.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servidorpostgresql do |servidorpostgresql|
    servidorpostgresql.vm.box = "debian/bullseye64"
    servidorpostgresql.vm.hostname = "servidorpostgresql"
    servidorpostgresql.vm.network :private_network,
      :libvirt__network_name => "red-postgresql",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.1.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientepostgresql do |clientepostgresql|
    clientepostgresql.vm.box = "debian/bullseye64"
    clientepostgresql.vm.hostname = "clientepostgresql"
    clientepostgresql.vm.network :private_network,
      :libvirt__network_name => "red-postgresql",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.1.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servidormariadb do |servidormariadb|
    servidormariadb.vm.box = "debian/bullseye64"
    servidormariadb.vm.hostname = "servidormariadb"
    servidormariadb.vm.network :private_network,
      :libvirt__network_name => "red-mariadb",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.2.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientemariadb do |clientemariadb|
    clientemariadb.vm.box = "debian/bullseye64"
    clientemariadb.vm.hostname = "clientemariadb"
    clientemariadb.vm.network :private_network,
      :libvirt__network_name => "red-mariadb",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.2.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servidormongodb do |servidormongodb|
    servidormongodb.vm.box = "debian/bullseye64"
    servidormongodb.vm.hostname = "servidormongodb"
    servidormongodb.vm.network :private_network,
      :libvirt__network_name => "red-mongodb",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.3.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientemongodb do |clientemongodb|
    clientemongodb.vm.box = "debian/bullseye64"
    clientemongodb.vm.hostname = "clientemongodb"
    clientemongodb.vm.network :private_network,
      :libvirt__network_name => "red-mongodb",
      :libvirt__dhcp_enabled => false,
      :ip => "10.0.3.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

## 1. Oracle 19c

### 1.1 Redimensión de disco

> Tenemos que tener suficiente espacio para instalar Oracle

Paro la VM:

```shell
vagrant halt servidororacle
```

Redimensiono el fichero de disco:

```shell
virsh -c qemu:///system vol-resize p1-bd-alumno1_servidororacle.img 50G --pool default
```

Dentro de la VM redimensiono la partición y el sistema de ficheros:

```shell
echo ", +" | sudo sfdisk -N 1 /dev/vda --no-reread
sudo apt update
sudo apt install parted
sudo partprobe
sudo resize2fs /dev/vda1
```

### 1.2 Instalación del servidor

#### 1.2.1

Descargo el rpm en mi host desde la página oficial de Oracle:

![descargarpm](https://i.imgur.com/b7LjTs5.png)

#### 1.2.2

Lo paso a la VM `servidororacle`:

```shell
scp Downloads/oracle-database-ee-19c-1.0-1.x86_64.rpm vagrant@192.168.121.159:/home/vagrant
```

#### 1.2.3

Instalo los paquetes requeridos:

```shell
sudo apt install alien libaio1 unixodbc net-tools
```

#### 1.2.4

Convierto el rpm a deb:

```shell
sudo alien --scripts -d oracle-database-ee-19c-1.0-1.x86_64.rpm
```

Como resultado tendremos el paquete `oracle-database-ee-19c_1.0-2_amd64.deb`

#### 1.2.5

Creo el siguiente script:

```shell
sudo pico /sbin/chkconfig
```

Con el siguiente contenido:

```shell
#!/bin/bash
# Oracle 19c installer chkconfig hack
file=/etc/init.d/oracle-19c
if [[ ! `tail -n1 $file | grep INIT` ]]; then
echo >> $file
echo '### BEGIN INIT INFO' >> $file
echo '# Provides: Oracle 19c' >> $file
echo '# Required-Start: $remote_fs $syslog' >> $file
echo '# Required-Stop: $remote_fs $syslog' >> $file
echo '# Default-Start: 2 3 4 5' >> $file
echo '# Default-Stop: 0 1 6' >> $file
echo '# Short-Description: Oracle 19c' >> $file
echo '### END INIT INFO' >> $file
fi
update-rc.d oracle-19c defaults 80 01
```

Le doy permisos:

```shell
sudo chmod 777 /sbin/chkconfig  
```

#### 1.2.6

Creo el siguiente fichero para los parámetros de Kernel de Oracle:

```shell
sudo touch /etc/sysctl.d/60-oracle.conf
```

Con el siguiente contenido:

```shell
# Oracle 19c kernel parameters
fs.file-max=6815744
net.ipv4.ip_local_port_range=9000 65000
kernel.sem=250 32000 100 128
kernel.shmmax=536870912
```

#### 1.2.7

Arranco el siguiente servicio:

```shell
sudo systemctl start procps
```

#### 1.2.8

Creo el siguiente fichero para configurar el punto de montaje `/dev/shm` de Oracle:

```shell
sudo touch /etc/rc2.d/S01shm_load
```

Con el siguiente contenido:

```shell
#!/bin/sh
case "$1" in
start) mkdir /var/lock/subsys 2>/dev/null
       touch /var/lock/subsys/listener
       rm /dev/shm 2>/dev/null
       mkdir /dev/shm 2>/dev/null
       mount -t tmpfs shmfs -o size=2048m /dev/shm ;;
*) echo error
   exit 1 ;;
esac
```

Le doy permisos:

```shell
sudo chmod 777 /etc/rc2.d/S01shm_load
```

Hago un reinicio:

```shell
sudo reboot
```

#### 1.2.9

Instalo el paquete:

```shell
sudo dpkg --install oracle-database-ee-19c_1.0-2_amd64.deb
```

#### 1.2.10

Añado el parámetro `-J-Doracle.assistants.dbca.validate.ConfigurationParams=false` al final de la siguiente línea en `/etc/init.d/oracledb_ORCLCDB-19c`:

![parametroañadido](https://i.postimg.cc/GpqQmxHm/parametroracle.png)

Quedaría de la siguiente manera la línea al modificarla:

```shell
$SU -s /bin/bash  $ORACLE_OWNER -c "$DBCA -silent -createDatabase -gdbName $ORACLE_SID -templateName $TEMPLATE_NAME -characterSet $CHARSET -createAsContainerDatabase $CREATE_AS_CDB -numberOfPDBs $NUMBER_OF_PDBS -pdbName $PDB_NAME -createListener $LISTENER_NAME:$LISTENER_PORT -datafileDestination $ORACLE_DATA_LOCATION -sid $ORACLE_SID -autoGeneratePasswords -emConfiguration DBEXPRESS -emExpressPort $EM_EXPRESS_PORT -J-Doracle.assistants.dbca.validate.ConfigurationParams=false"
```

#### 1.2.11

Añado la siguiente línea a `/etc/hosts`:

```shell
192.168.121.159 servidororacle servidororacle
```

#### 1.2.12

Ejecuto el script de configuración de Oracle:

```shell
sudo /etc/init.d/oracledb_ORCLCDB-19c configure
```

#### 1.2.13

Edito el siguiente fichero:

```shell
pico ~/.bashrc
```

Añado las siguientes variables de entorno al final del fichero:

```shell
# Oracle environment variables
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export ORACLE_SID=ORCLCDB
export NLS_LANG=`$ORACLE_HOME/bin/nls_lang.sh`
export ORACLE_BASE=/opt/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
```

Aplico los cambios:

```shell
. ~/.profile
```

Hago un reinicio:

```shell
sudo reboot
```

#### 1.2.14

Inicio el servicio de Oracle:

```shell
sudo systemctl start oracledb_ORCLCDB-19c
```

#### 1.2.15

Añado la contraseña `oracle` al usuario `oracle`:

```shell
sudo passwd oracle
```

Cambio su shell:

```shell
sudo usermod --shell /bin/bash oracle
```

Creo su home:

```shell
sudo mkdir /home/oracle
```

Cambio los propietarios:

```shell
sudo chown -R oracle:oinstall /home/oracle
```

Añado el siguiente fichero:

```shell
touch ~/.profile
```

Con el siguiente contenido:

```shell
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.

# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
#umask 022

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
	. "$HOME/.bashrc"
    fi
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
```

Le añado a este usuario también las variables de entorno:

```shell
pico ~/.bashrc
```

```shell
# Oracle environment variables
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export ORACLE_SID=ORCLCDB
export NLS_LANG=`$ORACLE_HOME/bin/nls_lang.sh`
export ORACLE_BASE=/opt/oracle
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
```

Aplico los cambios:

```shell
. ~/.profile
```

Le hago admin:

```shell
sudo usermod -aG sudo oracle
```

A partir de ahora, para controlar Oracle correctamente, tendremos que estar logeados siempre con este usuario.

#### 1.2.16

Añado el usuario `vagrant` al grupo `dba` por si en algún momento quiero acceder a sqlplus de manera rápida:

```shell
sudo usermod -a -G dba vagrant
```

### 1.3 Acceso local privilegiado

```shell
oracle@servidororacle:~$ sqlplus / as sysdba

SQL*Plus: Release 19.0.0.0.0 - Production on Fri Oct 28 14:13:53 2022
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SELECT banner FROM v$version;

BANNER
--------------------------------------------------------------------------------
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
```

Cuando en futuras ocasiones al conectarnos a Oracle nos aparezca el mensaje `Connected to an idle instance`, tenemos que ejecutar en `sqlplus` el comando `startup`.

### 1.4 Creación de usuario

```shell
alter session set "_ORACLE_SCRIPT"=true;
create user bibliofilos_admin identified by 1234;
grant all privileges to bibliofilos_admin;
```

Pruebo que funciona el acceso:

```shell
oracle@servidororacle:~$ sqlplus bibliofilos_admin/1234

SQL*Plus: Release 19.0.0.0.0 - Production on Fri Oct 28 14:18:26 2022
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Last Successful login time: Fri Oct 28 2022 13:52:49 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL>
```

Pruebo que los privilegios se han aplicado correctamente con la siguiente consulta:

```shell
select * from dba_sys_privs where grantee = 'BIBLIOFILOS_ADMIN';
```

El output de esta consulta es demasiado largo, así que [entra en este gist](https://gist.github.com/adriasir123/5cd2ec8d56974b6ae0d283243f6d5f41) para verlo.

### 1.5 Creación de tablas

```sql
CREATE TABLE bibliotecas (
  id NUMBER(10) NOT NULL,
  ciudad VARCHAR2(50) NOT NULL,
  calle VARCHAR2(50) NOT NULL,
  CONSTRAINT bibliotecas_pk PRIMARY KEY (id)
);

CREATE TABLE libros (
  id NUMBER(10) NOT NULL,
  id_biblioteca NUMBER(10) NOT NULL,
  n_paginas NUMBER(10) NOT NULL,
  autor VARCHAR2(50) NOT NULL,
  CONSTRAINT libros_pk PRIMARY KEY (id),
  CONSTRAINT id_biblioteca_fk_libros FOREIGN KEY (id_biblioteca)
    REFERENCES bibliotecas (id)
);

CREATE TABLE trabajadores (
  id NUMBER(10) NOT NULL,
  id_biblioteca NUMBER(10) NOT NULL,
  nombre VARCHAR2(50) NOT NULL,
  apellido VARCHAR2(50) NOT NULL,
  CONSTRAINT trabajadores_pk PRIMARY KEY (id),
  CONSTRAINT id_biblioteca_fk_trabajadores FOREIGN KEY (id_biblioteca)
    REFERENCES bibliotecas (id)
);
```

Compruebo que se han creado con la siguiente consulta:

```sql
SELECT
  table_name, owner
FROM
  all_tables
WHERE
  owner='BIBLIOFILOS_ADMIN'
ORDER BY
  owner, table_name;
```

Devuelve lo siguiente:

```shell
TABLE_NAME								                                                                                                                 OWNER
-------------------------------------------------------------------------------------------------------------------------------- --------------------------------------------------------------------------------------------------------------------------------
BIBLIOTECAS								                                                                                                                 BIBLIOFILOS_ADMIN
LIBROS								                                                                                                                         BIBLIOFILOS_ADMIN
TRABAJADORES								                                                                                                                 BIBLIOFILOS_ADMIN
```

### 1.6 Inserción de registros

```sql
INSERT INTO bibliotecas (id, ciudad, calle) VALUES (1, 'Utrera', 'Alvarez Quintero');
INSERT INTO bibliotecas (id, ciudad, calle) VALUES (2, 'Dos Hermanas', 'Plaza Huerta Palacios');

INSERT INTO libros (id, id_biblioteca, n_paginas, autor) VALUES (1, 1, 320, 'Dale Carnegie');
INSERT INTO libros (id, id_biblioteca, n_paginas, autor) VALUES (2, 2, 714, 'Anne Frank');

INSERT INTO trabajadores (id, id_biblioteca, nombre, apellido) VALUES (1, 1, 'Pepe', 'Pepito');
INSERT INTO trabajadores (id, id_biblioteca, nombre, apellido) VALUES (2, 2, 'Jose', 'Joselito');
```

Compruebo que se han añadido:

```sql
select * from bibliotecas;

	ID CIUDAD			                                      CALLE
---------- -------------------------------------------------- --------------------------------------------------
	 1 Utrera			                                      Alvarez Quintero
	 2 Dos Hermanas 		                              Plaza Huerta Palacios

select * from libros;

	ID ID_BIBLIOTECA  N_PAGINAS AUTOR
---------- ------------- ---------- --------------------------------------------------
	 1	       1	320 Dale Carnegie
	 2	       2	714 Anne Frank

select * from trabajadores;

	ID ID_BIBLIOTECA NOMBRE 			                                    APELLIDO
---------- ------------- -------------------------------------------------- --------------------------------------------------
	 1	       1 Pepe			                                            Pepito
	 2	       2 Jose			                                            Joselito
```

### 1.7 Configuración acceso remoto

Dejo `/opt/oracle/product/19c/dbhome_1/network/admin/listener.ora` de la siguiente manera:

```shell
# listener.ora Network Configuration File: /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = ORCLCDB)
      (ORACLE_HOME = /opt/oracle/product/19c/dbhome_1)
      (SID_NAME = ORCLCDB)
    )
  )
```

Dejo `/opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora` de la siguiente manera:

```shell
# tnsnames.ora Network Configuration File: /opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora
# Generated by Oracle configuration tools.

ORCLCDB=
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = servidororacle)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = ORCLCDB)
    )
  )

LISTENER_ORCLCDB =
  (ADDRESS = (PROTOCOL = TCP)(HOST = servidororacle)(PORT = 1521))
```

Muestro el estado del listener:

```shell
oracle@servidororacle:~$ lsnrctl status

LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 28-OCT-2022 13:43:38

Copyright (c) 1991, 2019, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=1521)))
TNS-12541: TNS:no listener
 TNS-12560: TNS:protocol adapter error
  TNS-00511: No listener
   Linux Error: 111: Connection refused
Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
TNS-12541: TNS:no listener
 TNS-12560: TNS:protocol adapter error
  TNS-00511: No listener
   Linux Error: 2: No such file or directory
```

Estos errores significan que está parado, y además podemos comprobar que efectivamente no tenemos procesos esperando peticiones en el puerto 1521:

```shell
oracle@servidororacle:~$ sudo ss -tulpn
Netid      State        Recv-Q       Send-Q             Local Address:Port              Peer Address:Port      Process
udp        UNCONN       0            0                        0.0.0.0:68                     0.0.0.0:*          users:(("dhclient",pid=314,fd=9))
udp        UNCONN       0            0                      127.0.0.1:323                    0.0.0.0:*          users:(("chronyd",pid=413,fd=5))
udp        UNCONN       0            0                          [::1]:21455                     [::]:*          users:(("ora_lreg_orclcd",pid=774,fd=10))
udp        UNCONN       0            0                          [::1]:48386                     [::]:*          users:(("ora_d000_orclcd",pid=786,fd=7))
udp        UNCONN       0            0                          [::1]:323                       [::]:*          users:(("chronyd",pid=413,fd=6))
udp        UNCONN       0            0                          [::1]:51821                     [::]:*          users:(("ora_s000_orclcd",pid=788,fd=7))
tcp        LISTEN       0            128                      0.0.0.0:22                     0.0.0.0:*          users:(("sshd",pid=411,fd=3))
tcp        LISTEN       0            128                            *:19393                        *:*          users:(("ora_d000_orclcd",pid=786,fd=8))
tcp        LISTEN       0            128                         [::]:22                        [::]:*          users:(("sshd",pid=411,fd=4))
```

Inicio el listener:

```shell
oracle@servidororacle:~$ lsnrctl start

LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 28-OCT-2022 13:48:46

Copyright (c) 1991, 2019, Oracle.  All rights reserved.

Starting /opt/oracle/product/19c/dbhome_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 19.0.0.0.0 - Production
System parameter file is /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
Log messages written to /opt/oracle/diag/tnslsnr/servidororacle/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
Start Date                28-OCT-2022 13:48:46
Uptime                    0 days 0 hr. 0 min. 0 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
Listener Log File         /opt/oracle/diag/tnslsnr/servidororacle/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
Services Summary...
Service "ORCLCDB" has 1 instance(s).
  Instance "ORCLCDB", status UNKNOWN, has 1 handler(s) for this service...
The command completed successfully
```

Muestro que ahora sí tenemos un proceso esperando peticiones en el puerto 1521:

![listenerfunciona](https://i.imgur.com/bhzCQGe.png)

Pruebo a hacer un acceso usando tnsnames localmente para comprobar que funciona:

![tnsnameslocal](https://i.imgur.com/7CJIL08.png)

Cuando usamos el @ estamos forzando las conexiones por tnsnames aunque sean locales.

### 1.8 Instalación del cliente

Se hará sobre `clienteoracle`.

Descargo estos 2 archivos:

```shell
wget https://download.oracle.com/otn_software/linux/instantclient/218000/instantclient-basic-linux.x64-21.8.0.0.0dbru.zip
wget https://download.oracle.com/otn_software/linux/instantclient/218000/instantclient-sqlplus-linux.x64-21.8.0.0.0dbru.zip
```

Creo el siguiente directorio:

```shell
sudo mkdir /opt/oracle
```

Descargo `unzip`:

```shell
sudo apt update
sudo apt install unzip
```

Descomprimo:

```shell
sudo unzip -d /opt/oracle instantclient-basic-linux.x64-21.8.0.0.0dbru.zip
sudo unzip -d /opt/oracle instantclient-sqlplus-linux.x64-21.8.0.0.0dbru.zip
```

Añado las siguientes variables de entorno a `.bashrc`:

```shell
export LD_LIBRARY_PATH=/opt/oracle/instantclient_21_8:$LD_LIBRARY_PATH
export PATH=$LD_LIBRARY_PATH:$PATH
```

Aplico los cambios:

```shell
source ~/.bashrc
```

Compruebo que ya funciona `sqlplus`:

```shell
vagrant@clienteoracle:~$ sqlplus -V

SQL*Plus: Release 21.0.0.0.0 - Production
Version 21.8.0.0.0
```

Creo el fichero `/opt/oracle/instantclient_21_8/network/admin/tnsnames.ora`:

```shell
sudo touch /opt/oracle/instantclient_21_8/network/admin/tnsnames.ora
```

Con el siguiente contenido:

```shell
ORCLCDB=
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.0.0.2)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = ORCLCDB)
    )
  )
```

### 1.9 Prueba de acceso remoto

```shell
vagrant@clienteoracle:~$ sqlplus bibliofilos_admin/1234@ORCLCDB

SQL*Plus: Release 21.0.0.0.0 - Production on Fri Oct 28 12:23:58 2022
Version 21.8.0.0.0

Copyright (c) 1982, 2022, Oracle.  All rights reserved.

Last Successful login time: Fri Oct 28 2022 12:23:47 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL> set linesize 32000
SQL> set pagesize 400
SQL> select * from bibliotecas;

	ID CIUDAD			                                      CALLE
---------- -------------------------------------------------- --------------------------------------------------
	 1 Utrera			                                      Alvarez Quintero
	 2 Dos Hermanas 		                              Plaza Huerta Palacios
```

### 1.10 Mejoras de sqlplus

#### 1.10.1

Para tener un formateado correcto del output **permanente**, tenemos que editar el fichero `/opt/oracle/product/19c/dbhome_1/sqlplus/admin/glogin.sql`.

Lo podemos definir como queramos, en mi caso lo dejaré de la siguiente manera:

```sql
--
-- Copyright (c) 1988, 2005, Oracle.  All Rights Reserved.
--
-- NAME
--   glogin.sql
--
-- DESCRIPTION
--   SQL*Plus global login "site profile" file
--
--   Add any SQL*Plus commands here that are to be executed when a
--   user starts SQL*Plus, or uses the SQL*Plus CONNECT command.
--
-- USAGE
--   This script is automatically run
--

set linesize 32000
set pagesize 400
```

Me conecto y hago una select para comprobar que funciona:

```shell
oracle@servidororacle:~$ sqlplus bibliofilos_admin/1234@ORCLCDB

SQL*Plus: Release 19.0.0.0.0 - Production on Fri Oct 28 12:39:06 2022
Version 19.3.0.0.0

Copyright (c) 1982, 2019, Oracle.  All rights reserved.

Last Successful login time: Fri Oct 28 2022 12:23:58 +00:00

Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.3.0.0.0

SQL> select * from bibliotecas;

	ID CIUDAD			                                      CALLE
---------- -------------------------------------------------- --------------------------------------------------
	 1 Utrera			                                      Alvarez Quintero
	 2 Dos Hermanas 		                              Plaza Huerta Palacios
```

#### 1.10.2

Por defecto sqlplus no trae la funcionalidad de historial de comandos y uso de las flechas para mover el cursor en los comandos.

Si lo intentamos, nos sucede lo siguiente:

![sqlplusinwrapper](https://i.postimg.cc/Jn6ZpCz3/sqlplussinwrapper.gif)

Para arreglarlo, tenemos que instalar:

```shell
sudo apt install rlwrap
```

A partir de ahora, tenemos que ejecutar sqlplus de la siguiente manera:

```shell
rlwrap sqlplus bibliofilos_admin/1234@ORCLCDB
```

Básicamente añadiendo `rlwrap` al principio de cualquier comando de conexión sqlplus que queramos hacer.

Hacer esto siempre es tedioso, así que podemos crear un alias en `~/.bashrc`:

```shell
alias sqlplus='rlwrap sqlplus'
```

Aplico los cambios:

```shell
source ~/.bashrc
```

Muestro el funcionamiento correcto:

![sqlplusconwrapper](https://i.postimg.cc/qRg3FDY8/rlwrapper.gif)

## 2. PostgreSQL

### 2.1

> Instalar el servidor

```shell
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Compruebo que funciona:

```shell
vagrant@servidorpostgresql:~$ sudo systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2022-10-18 10:12:46 UTC; 2min 50s ago
   Main PID: 2772 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 527)
     Memory: 0B
        CPU: 0
     CGroup: /system.slice/postgresql.service

Oct 18 10:12:46 servidorpostgresql systemd[1]: Starting PostgreSQL RDBMS...
Oct 18 10:12:46 servidorpostgresql systemd[1]: Finished PostgreSQL RDBMS.
```

### 2.2

> Acceder con el usuario administrador localmente

Añado la contraseña `1234` a `postgres` para habilitarlo:

```shell
sudo passwd postgres
```

Pruebo el acceso con `postgres`:

```shell
vagrant@servidorpostgresql:~$ su - postgres
Password:
postgres@servidorpostgresql:~$ psql
psql (13.8 (Debian 13.8-0+deb11u1))
Type "help" for help.

postgres=#
```

### 2.3

> Crear la bd

```sql
CREATE DATABASE bibliofilos;
```

Compruebo que se ha creado:

```shell
postgres=# \l
                               List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |   Access privileges
-------------+----------+----------+---------+---------+-----------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
             |          |          |         |         | postgres=CTc/postgres
(4 rows)
```

Me conecto:

```shell
postgres=# \connect bibliofilos
You are now connected to database "bibliofilos" as user "postgres".
bibliofilos=#
```

### 2.4

> Crear tablas

```shell
CREATE TABLE bibliotecas (
  id serial PRIMARY KEY,
  ciudad VARCHAR ( 50 ) NOT NULL,
  calle VARCHAR ( 50 ) NOT NULL
);

CREATE TABLE libros (
  id serial PRIMARY KEY,
  id_biblioteca serial NOT NULL,
  n_paginas int NOT NULL,
  autor VARCHAR ( 50 ) NOT NULL,
  FOREIGN KEY (id_biblioteca)
    REFERENCES Bibliotecas (id)
);

CREATE TABLE trabajadores (
  id serial PRIMARY KEY,
  id_biblioteca serial NOT NULL,
  nombre VARCHAR ( 50 ) NOT NULL,
  apellido VARCHAR ( 50 ) NOT NULL,
  FOREIGN KEY (id_biblioteca)
    REFERENCES Bibliotecas (id)
);
```

Compruebo que se han creado:

```shell
bibliofilos=# \dt
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | bibliotecas  | table | postgres
 public | libros       | table | postgres
 public | trabajadores | table | postgres
(3 rows)
```

### 2.5

> Introducir registros

```sql
INSERT INTO bibliotecas (ciudad, calle) VALUES('Utrera', 'Alvarez Quintero');
INSERT INTO bibliotecas (ciudad, calle) VALUES('Dos Hermanas', 'Plaza Huerta Palacios');

INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES(1, 320, 'Dale Carnegie');
INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES(2, 714, 'Anne Frank');

INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES(1, 'Pepe', 'Pepito');
INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES(2, 'Jose', 'Joselito');
```

Compruebo que se han añadido:

```shell
bibliofilos=# select * from bibliotecas;
 id |    ciudad    |         calle
----+--------------+-----------------------
  1 | Utrera       | Alvarez Quintero
  2 | Dos Hermanas | Plaza Huerta Palacios
(2 rows)

bibliofilos=# select * from libros;
 id | id_biblioteca | n_paginas |     autor
----+---------------+-----------+---------------
  1 |             1 |       320 | Dale Carnegie
  2 |             2 |       714 | Anne Frank
(2 rows)

bibliofilos=# select * from trabajadores;
 id | id_biblioteca | nombre | apellido
----+---------------+--------+----------
  1 |             1 | Pepe   | Pepito
  2 |             2 | Jose   | Joselito
(2 rows)
```

### 2.6

> Crear usuario con todos los privilegios sobre la base de datos anterior

En debian:

```shell
sudo adduser bibliofilos_admin
```

En PostgreSQL:

```sql
create user bibliofilos_admin with encrypted password '1234';
```

Compruebo que se ha creado:

```shell
postgres=# \du
                                       List of roles
     Role name     |                         Attributes                         | Member of
-------------------+------------------------------------------------------------+-----------
 bibliofilos_admin |                                                            | {}
 postgres          | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

Le doy privilegios sobre la base de datos:

```sql
GRANT all privileges ON DATABASE bibliofilos TO bibliofilos_admin;
```

Al listar las bases de datos podemos ver los privilegios de acceso:

```shell
postgres=# \l
                                   List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |       Access privileges
-------------+----------+----------+---------+---------+--------------------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | bibliofilos_admin=CTc/postgres
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
(4 rows)
```

Le doy privilegios sobre las tablas:

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO bibliofilos_admin;
```

Muestro que se han aplicado los privilegios:

```sql
bibliofilos=# SELECT * from information_schema.table_privileges WHERE grantee = 'bibliofilos_admin';
 grantor  |      grantee      | table_catalog | table_schema |  table_name  | privilege_type | is_grantable | with_hierarchy
----------+-------------------+---------------+--------------+--------------+----------------+--------------+----------------
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | INSERT         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | SELECT         | NO           | YES
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | UPDATE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | DELETE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | TRUNCATE       | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | REFERENCES     | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | bibliotecas  | TRIGGER        | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | INSERT         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | SELECT         | NO           | YES
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | UPDATE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | DELETE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | TRUNCATE       | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | REFERENCES     | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | libros       | TRIGGER        | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | INSERT         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | SELECT         | NO           | YES
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | UPDATE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | DELETE         | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | TRUNCATE       | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | REFERENCES     | NO           | NO
 postgres | bibliofilos_admin | bibliofilos   | public       | trabajadores | TRIGGER        | NO           | NO
(21 rows)
```

### 2.7

> Permitir el acceso remoto

Modifico la siguiente línea en `/etc/postgresql/13/main/postgresql.conf`:

```shell
listen_addresses = '*'
```

Añado las siguientes líneas a `/etc/postgresql/13/main/pg_hba.conf`:

```shell
# Remote connections
host    all             all             0.0.0.0/0               md5
```

Reinicio:

```shell
sudo systemctl restart postgresql 
```

### 2.8

> Instalar el cliente de PostgreSQL en `clientepostgresql`

```shell
sudo apt update
sudo apt install postgresql-client
```

### 2.9

> Probar el acceso remoto

```shell
vagrant@clientepostgresql:~$ psql -U bibliofilos_admin -h 10.0.1.2 -p 5432 bibliofilos
Password for user bibliofilos_admin:
psql (13.8 (Debian 13.8-0+deb11u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

bibliofilos=> \l
                                   List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |       Access privileges
-------------+----------+----------+---------+---------+--------------------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | bibliofilos_admin=CTc/postgres
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
(4 rows)

bibliofilos=> \dt
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | bibliotecas  | table | postgres
 public | libros       | table | postgres
 public | trabajadores | table | postgres
(3 rows)

bibliofilos=> select * from bibliotecas;
 id |    ciudad    |         calle
----+--------------+-----------------------
  1 | Utrera       | Alvarez Quintero
  2 | Dos Hermanas | Plaza Huerta Palacios
(2 rows)
```

## 3. MariaDB

### 3.1

> Instalar el servidor

```shell
sudo apt update
sudo apt install mariadb-server
```

Compruebo que funciona:

```shell
vagrant@servidormariadb:~$ sudo systemctl status mariadb
● mariadb.service - MariaDB 10.5.15 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-10-19 11:01:36 UTC; 11min ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 2111 ExecStartPre=/usr/bin/install -m 755 -o mysql -g root -d /var/run/mysqld (code=exited, status=0/SUCCESS)
    Process: 2112 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 2114 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`cd /usr/bin/..; /usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-envi>
    Process: 2174 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Process: 2176 ExecStartPost=/etc/mysql/debian-start (code=exited, status=0/SUCCESS)
   Main PID: 2161 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 8 (limit: 527)
     Memory: 61.7M
        CPU: 425ms
     CGroup: /system.slice/mariadb.service
             └─2161 /usr/sbin/mariadbd

Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] Server socket created on IP: '127.0.0.1'.
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] InnoDB: Buffer pool(s) load completed at 221019 11:01:36
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] Reading of all Master_info entries succeeded
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] Added new Master_info '' to hash table
Oct 19 11:01:36 servidormariadb mariadbd[2161]: 2022-10-19 11:01:36 0 [Note] /usr/sbin/mariadbd: ready for connections.
Oct 19 11:01:36 servidormariadb mariadbd[2161]: Version: '10.5.15-MariaDB-0+deb11u1'  socket: '/run/mysqld/mysqld.sock'  port: 3306  Debian 11
Oct 19 11:01:36 servidormariadb systemd[1]: Started MariaDB 10.5.15 database server.
Oct 19 11:01:36 servidormariadb /etc/mysql/debian-start[2178]: Upgrading MySQL tables if necessary.
Oct 19 11:01:36 servidormariadb /etc/mysql/debian-start[2189]: Checking for insecure root accounts.
Oct 19 11:01:36 servidormariadb /etc/mysql/debian-start[2193]: Triggering myisam-recover for all MyISAM tables and aria-recover for all Aria tables
```

### 3.2

> Acceder con el usuario administrador localmente

```shell
vagrant@servidormariadb:~$ sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 34
Server version: 10.5.15-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

### 3.3

> Crear la bd

```sql
create database bibliofilos;
```

Compruebo que se ha creado:

```shell
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| bibliofilos        |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec)
```

Me conecto:

```shell
MariaDB [(none)]> use bibliofilos;
Database changed
MariaDB [bibliofilos]>
```

### 3.4

> Crear tablas

```sql
create table bibliotecas(
  id int auto_increment,
  ciudad varchar(50) not null,
  calle varchar(50) not null,
  primary key(id)
);

create table libros(
  id int auto_increment,
  id_biblioteca int not null,
  n_paginas int not null,
  autor varchar(50) not null,
  primary key(id),
  foreign key(id_biblioteca)
    references bibliotecas(id)
);

create table trabajadores(
  id int auto_increment,
  id_biblioteca int not null,
  nombre varchar(50) not null,
  apellido varchar(50) not null,
  primary key(id),
  foreign key(id_biblioteca)
    references bibliotecas(id)
);
```

Compruebo que se han creado:

```shell
MariaDB [bibliofilos]> show tables;
+-----------------------+
| Tables_in_bibliofilos |
+-----------------------+
| bibliotecas           |
| libros                |
| trabajadores          |
+-----------------------+
3 rows in set (0.001 sec)
```

### 3.5

> Introducir registros

```sql
INSERT INTO bibliotecas (ciudad, calle) VALUES ('Utrera', 'Alvarez Quintero');
INSERT INTO bibliotecas (ciudad, calle) VALUES ('Dos Hermanas', 'Plaza Huerta Palacios');

INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES (1, 320, 'Dale Carnegie');
INSERT INTO libros (id_biblioteca, n_paginas, autor) VALUES (2, 714, 'Anne Frank');

INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES (1, 'Pepe', 'Pepito');
INSERT INTO trabajadores (id_biblioteca, nombre, apellido) VALUES (2, 'Jose', 'Joselito');
```

Compruebo que se han añadido:

```shell
MariaDB [bibliofilos]> select * from bibliotecas;
+----+--------------+-----------------------+
| id | ciudad       | calle                 |
+----+--------------+-----------------------+
|  1 | Utrera       | Alvarez Quintero      |
|  2 | Dos Hermanas | Plaza Huerta Palacios |
+----+--------------+-----------------------+
2 rows in set (0.001 sec)

MariaDB [bibliofilos]> select * from libros;
+----+---------------+-----------+---------------+
| id | id_biblioteca | n_paginas | autor         |
+----+---------------+-----------+---------------+
|  1 |             1 |       320 | Dale Carnegie |
|  2 |             2 |       714 | Anne Frank    |
+----+---------------+-----------+---------------+
2 rows in set (0.001 sec)

MariaDB [bibliofilos]> select * from trabajadores;
+----+---------------+--------+----------+
| id | id_biblioteca | nombre | apellido |
+----+---------------+--------+----------+
|  1 |             1 | Pepe   | Pepito   |
|  2 |             2 | Jose   | Joselito |
+----+---------------+--------+----------+
2 rows in set (0.001 sec)
```

### 3.6

> Crear usuario con todos los privilegios sobre la base de datos anterior

```sql
CREATE USER 'bibliofilos_admin'@'localhost' IDENTIFIED BY '1234';
CREATE USER 'bibliofilos_admin'@'%' IDENTIFIED BY '1234';
```

Compruebo que se ha creado:

```shell
MariaDB [(none)]> SELECT user, host FROM mysql.user;
+-------------------+-----------+
| User              | Host      |
+-------------------+-----------+
| bibliofilos_admin | %         |
| bibliofilos_admin | localhost |
| mariadb.sys       | localhost |
| mysql             | localhost |
| root              | localhost |
+-------------------+-----------+
5 rows in set (0.002 sec)
```

Le doy privilegios:

```sql
GRANT ALL ON bibliofilos.* TO 'bibliofilos_admin'@'localhost';
GRANT ALL ON bibliofilos.* TO 'bibliofilos_admin'@'%';
FLUSH PRIVILEGES;
```

Muestro que se han aplicado:

```sql
MariaDB [(none)]> SHOW GRANTS FOR 'bibliofilos_admin';
+------------------------------------------------------------------------------------------------------------------+
| Grants for bibliofilos_admin@%                                                                                   |
+------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `bibliofilos_admin`@`%` IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' |
| GRANT ALL PRIVILEGES ON `bibliofilos`.* TO `bibliofilos_admin`@`%`                                               |
+------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)

MariaDB [(none)]> SHOW GRANTS FOR 'bibliofilos_admin'@'localhost';
+--------------------------------------------------------------------------------------------------------------------------+
| Grants for bibliofilos_admin@localhost                                                                                   |
+--------------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `bibliofilos_admin`@`localhost` IDENTIFIED BY PASSWORD '*A4B6157319038724E3560894F7F932C8886EBFCF' |
| GRANT ALL PRIVILEGES ON `bibliofilos`.* TO `bibliofilos_admin`@`localhost`                                               |
+--------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)
```

### 3.7

> Permitir el acceso remoto

Modifico la siguiente línea en `/etc/mysql/mariadb.conf.d/50-server.cnf`:

```shell
bind-address            = 0.0.0.0
```

Reinicio:

```shell
sudo systemctl restart mariadb
```

Pruebo que el cambio se aplicó:

```shell
vagrant@servidormariadb:~$ netstat -ant | grep 3306
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN
```

### 3.8

> Instalar el cliente de MariaDB en `clientemariadb`

```shell
sudo apt update
sudo apt install mariadb-client
```

### 3.9

> Probar el acceso remoto

```shell
vagrant@clientemariadb:~$ mariadb -u bibliofilos_admin -h 10.0.2.2 -p bibliofilos
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 35
Server version: 10.5.15-MariaDB-0+deb11u1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [bibliofilos]> show databases;
+--------------------+
| Database           |
+--------------------+
| bibliofilos        |
| information_schema |
+--------------------+
2 rows in set (0.002 sec)

MariaDB [bibliofilos]> show tables;
+-----------------------+
| Tables_in_bibliofilos |
+-----------------------+
| bibliotecas           |
| libros                |
| trabajadores          |
+-----------------------+
3 rows in set (0.001 sec)

MariaDB [bibliofilos]> select * from bibliotecas;
+----+--------------+-----------------------+
| id | ciudad       | calle                 |
+----+--------------+-----------------------+
|  1 | Utrera       | Alvarez Quintero      |
|  2 | Dos Hermanas | Plaza Huerta Palacios |
+----+--------------+-----------------------+
2 rows in set (0.002 sec)
```

## 4. MongoDB

### 4.1

> Instalar el servidor

Añado el repositorio:

```shell
echo "deb http://repo.mongodb.org/apt/debian bullseye/mongodb-org/5.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

Añado la clave GPG:

```shell
curl -sSL https://www.mongodb.org/static/pgp/server-5.0.asc  -o mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --import ./mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --export > ./mongoserver_key.gpg
sudo mv mongoserver_key.gpg /etc/apt/trusted.gpg.d/
```

Actualizo:

```shell
sudo apt update
```

Instalo:

```shell
sudo apt install mongodb-org
```

Inicio el servidor:

```shell
sudo systemctl enable --now mongod
```

Compruebo que funciona:

```shell
vagrant@servidormongodb:~$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-10-20 08:20:59 UTC; 2min 26s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 2695 (mongod)
     Memory: 120.9M
        CPU: 1.866s
     CGroup: /system.slice/mongod.service
             └─2695 /usr/bin/mongod --config /etc/mongod.conf

Oct 20 08:20:59 servidormongodb systemd[1]: Started MongoDB Database Server.
```

### 4.2

> Probar el acceso

```shell
vagrant@servidormongodb:~$ mongosh
Current Mongosh Log ID:63510a480bb54d2af15cd761
Connecting to:	        mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.6.0
Using MongoDB:	        5.0.13
Using Mongosh:	        1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2022-10-20T08:21:00.034+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2022-10-20T08:21:00.617+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
------

------
   Enable MongoDB's free cloud-based monitoring service, which will then receive and display
   metrics about your deployment (disk utilization, CPU, operation statistics, etc).

   The monitoring data will be available on a MongoDB website with a unique URL accessible to you
   and anyone you share the URL with. MongoDB may use this information to make product
   improvements and to suggest MongoDB products and deployment options to you.

   To enable free monitoring, run the following command: db.enableFreeMonitoring()
   To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
------

Warning: Found ~/.mongorc.js, but not ~/.mongoshrc.js. ~/.mongorc.js will not be loaded.
  You may want to copy or rename ~/.mongorc.js to ~/.mongoshrc.js.
test>
```

### 4.3

> Crear la bd

Si no existe la bd con el nombre que le digamos, la creará:

```sql
test> use bibliofilos
switched to db bibliofilos
bibliofilos>
```

Compruebo que se ha creado:

```shell
bibliofilos> show dbs
admin    40.00 KiB
config  108.00 KiB
local    40.00 KiB
```

La bd no aparece listada por ahora porque necesita contener datos para aparecer.

### 4.4

> Crear colecciones

```sql
db.createCollection("bibliotecas")

db.createCollection("libros")

db.createCollection("trabajadores")
```

Compruebo que se han creado:

```shell
bibliofilos> show collections
bibliotecas
libros
trabajadores
```

### 4.5

> Insertar documentos

```sql
db.bibliotecas.insertMany( [
  { ciudad: "Utrera", calle: "Alvarez Quintero" },
  { ciudad: "Dos Hermanas", calle: "Plaza Huerta Palacios" }
] )

db.libros.insertMany( [
  { n_paginas: 320, autor: "Dale Carnegie" },
  { n_paginas: 714, autor: "Anne Frank" }
] )

db.trabajadores.insertMany( [
  { nombre: "Pepe", apellido: "Pepito" },
  { nombre: "Jose", apellido: "Joselito" },
] )
```

Compruebo que se han insertado:

```shell
bibliofilos> db.bibliotecas.find()
[
  {
    _id: ObjectId("635126caf4c5855bf45547d3"),
    ciudad: 'Utrera',
    calle: 'Alvarez Quintero'
  },
  {
    _id: ObjectId("635126caf4c5855bf45547d4"),
    ciudad: 'Dos Hermanas',
    calle: 'Plaza Huerta Palacios'
  }
]
bibliofilos> db.libros.find()
[
  {
    _id: ObjectId("63512712f4c5855bf45547d5"),
    n_paginas: 320,
    autor: 'Dale Carnegie'
  },
  {
    _id: ObjectId("63512712f4c5855bf45547d6"),
    n_paginas: 714,
    autor: 'Anne Frank'
  }
]
bibliofilos> db.trabajadores.find()
[
  {
    _id: ObjectId("6351271cf4c5855bf45547d7"),
    nombre: 'Pepe',
    apellido: 'Pepito'
  },
  {
    _id: ObjectId("6351271cf4c5855bf45547d8"),
    nombre: 'Jose',
    apellido: 'Joselito'
  }
]
```

### 4.6

> Crear usuario con todos los privilegios sobre la base de datos anterior

```sql
use bibliofilos
db.createUser(
  {
    user: "bibliofilos_admin",
    pwd: "1234",
    roles: [ "dbOwner" ]
  }
)
```

Compruebo que se ha creado:

```shell
bibliofilos> db.getUsers()
{
  users: [
    {
      _id: 'bibliofilos.bibliofilos_admin',
      userId: new UUID("e420bea0-b85f-4914-a395-2f2b20b3aa48"),
      user: 'bibliofilos_admin',
      db: 'bibliofilos',
      roles: [ { role: 'dbOwner', db: 'bibliofilos' } ],
      mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
    }
  ],
  ok: 1
}
```

### 4.7

> Crear usuario administrador

```sql
use admin
db.createUser(
  {
    user: "admin",
    pwd: "1234",
    roles: [
      { role: "userAdminAnyDatabase", db: "admin" },
      { role: "readWriteAnyDatabase", db: "admin" }
    ]
  }
)
```

Compruebo que se ha creado:

```shell
admin> db.getUsers()
{
  users: [
    {
      _id: 'admin.admin',
      userId: new UUID("022fafb9-953b-4e5e-985f-6634d0ed954c"),
      user: 'admin',
      db: 'admin',
      roles: [
        { role: 'userAdminAnyDatabase', db: 'admin' },
        { role: 'readWriteAnyDatabase', db: 'admin' }
      ],
      mechanisms: [ 'SCRAM-SHA-1', 'SCRAM-SHA-256' ]
    }
  ],
  ok: 1
}
```

### 4.8

> Permitir el acceso remoto

Modifico la siguiente línea en `/etc/mongod.conf`:

```shell
bindIp: 0.0.0.0
```

Añado las siguientes líneas en `/etc/mongod.conf`:

```shell
security:
  authorization: enabled
```

Reinicio:

```shell
sudo systemctl restart mongod
```

Pruebo que el cambio se aplicó:

```shell
vagrant@servidormongodb:~$ netstat -ant | grep 27017
tcp        0      0 0.0.0.0:27017           0.0.0.0:*               LISTEN
```

### 4.9

> Instalar el cliente de MongoDB en `clientemongodb`

Añado el repositorio:

```shell
echo "deb http://repo.mongodb.org/apt/debian bullseye/mongodb-org/5.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

Añado la clave GPG:

```shell
curl -sSL https://www.mongodb.org/static/pgp/server-5.0.asc  -o mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --import ./mongoserver.asc
gpg --no-default-keyring --keyring ./mongo_key_temp.gpg --export > ./mongoserver_key.gpg
sudo mv mongoserver_key.gpg /etc/apt/trusted.gpg.d/
```

Actualizo:

```shell
sudo apt update
```

Instalo:

```shell
sudo apt install mongodb-org-shell mongodb-mongosh
```

### 4.10

> Probar el acceso remoto

```shell
vagrant@clientemongodb:~$ mongosh -u bibliofilos_admin -p 1234 10.0.3.2/bibliofilos
Current Mongosh Log ID:6351bcb93f3b38d91c3c973b
Connecting to:	        mongodb://<credentials>@10.0.3.2:27017/bibliofilos?directConnection=true&appName=mongosh+1.6.0
Using MongoDB:	        5.0.13
Using Mongosh:	        1.6.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

Warning: Found ~/.mongorc.js, but not ~/.mongoshrc.js. ~/.mongorc.js will not be loaded.
  You may want to copy or rename ~/.mongorc.js to ~/.mongoshrc.js.
bibliofilos> show dbs
bibliofilos  120.00 KiB
bibliofilos> show collections
bibliotecas
libros
trabajadores
bibliofilos> db.bibliotecas.find()
[
  {
    _id: ObjectId("635126caf4c5855bf45547d3"),
    ciudad: 'Utrera',
    calle: 'Alvarez Quintero'
  },
  {
    _id: ObjectId("635126caf4c5855bf45547d4"),
    ciudad: 'Dos Hermanas',
    calle: 'Plaza Huerta Palacios'
  }
]
```

## 5. Cliente remoto PostgreSQL

### 5.1

> Instalar el cliente de PostgreSQL en `clientepostgresql`

```shell
sudo apt update
sudo apt install postgresql-client
```

### 5.2

> Probar el acceso remoto

```shell
vagrant@clientepostgresql:~$ psql -U bibliofilos_admin -h 10.0.1.2 -p 5432 bibliofilos
Password for user bibliofilos_admin:
psql (13.8 (Debian 13.8-0+deb11u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

bibliofilos=> \l
                                   List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |       Access privileges
-------------+----------+----------+---------+---------+--------------------------------
 bibliofilos | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =Tc/postgres                  +
             |          |          |         |         | postgres=CTc/postgres         +
             |          |          |         |         | bibliofilos_admin=CTc/postgres
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres                   +
             |          |          |         |         | postgres=CTc/postgres
(4 rows)

bibliofilos=> \dt
            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | bibliotecas  | table | postgres
 public | libros       | table | postgres
 public | trabajadores | table | postgres
(3 rows)

bibliofilos=> select * from bibliotecas;
 id |    ciudad    |         calle
----+--------------+-----------------------
  1 | Utrera       | Alvarez Quintero
  2 | Dos Hermanas | Plaza Huerta Palacios
(2 rows)
```

## 6. App Flask con MongoDB

En [este repositorio](https://github.com/adriasir123/flask-mongo) se encuentran tanto el código de la aplicación como su guía de uso y pruebas de funcionamiento.
