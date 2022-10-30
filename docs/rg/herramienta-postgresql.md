# Herramienta web PostgreSQL

La herramienta web que he elegido para configurar postgres se llama phpPgAdmin.

Para poder instalar phpPgAdmin primero debemos realizar la instalación y configuración postgres al igual que la instalación y configuración de apache2.

## Instalacion phpPgAdmin y configuración

- Primero Instalamos apache2

```bash
sudo apt install apache2
```

- Seguidamente, instalamos phppgadmin:

```bash
sudo apt install phppgadmin php-pgsql
```

- Una vez instalado, editamos el fichero de configuración phppgadmin.conf y comentamos la línea Require local:

![comentar](../img/alumno2/comentar-require.png)

- Ahora en el fichero de configuración de config.inc.php, cambiamos el valor true por false en la siguiente línea:

![cambiar](../img/alumno2/cambiar-true-false.png)

- Ahora definimos las siguientes líneas dentro de config.inc.php:

![configurar](../img/alumno2/configurar-ips.png)

- Reiniciamos apache y accedemos a phppgmyadmin:

![acceso](../img/alumno2/acceso-phppg.png)

- Introducimos nuestras credenciales que configuramos en la instalación y configuración de postgres y accedemos a la configuración web:

- Introduzco los datos:

![acceso](../img/alumno2/acceso-phppg.png)

- Accedo al sistema:

![acceso](../img/alumno2/panel-phppg.png)