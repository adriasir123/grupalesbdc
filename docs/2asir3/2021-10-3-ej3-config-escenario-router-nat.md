---
title: "Ejercicio 3: Configuración del escenario router-nat"
---

![](https://fp.josedomingo.org/sri2122/u01/img/router.png)

# ENUNCIADO

Crear playbook en ansible con los roles:

- **common**: tareas para ambos nodos. Actualizar paquetes.
- **router**: tareas necesarias para configurar router como router-nat y que salga a internet por eth1. Las configuraciones deben ser permanentes.
- **cliente**: salir a internet por eth1 (cambiar default gateway, configuración dns…)



# REALIZACIÓN

## Rol common

```
- name: Actualizando paquetes...
  apt: update_cache=yes upgrade=yes
```

## Rol router
Uso este plugin de ansible para el forwarding:
```
ansible-galaxy collection install ansible.posix
```

Está muy interesante porque hace las 2 cosas que necesitamos a la vez:
 - Modifica el fichero `/proc/sys/net/ipv4/ip_forward`
 - Modifica el fichero `/etc/sysctl.conf` para que los cambios sean persistentes

```

```

FALTA QUE FORWARDING A VECES FUNCIONA, A VECES NO, CON LA CORRECTA CONFIGURACIÓN, ES INCONSISTENTE


Rutas por defecto no son permanentes by default
IDEA: PASAR FICHERO DE SCRIPTS DE INICIO, QUE BORREN Y AÑADAN RUTA

LAS RUTAS POR DEFECTO LAS HACE CON UN TEMPLATE. ES UN FICHERO DE INTERFACES, QUE LO COPIA AL DESTINO, CON VARIABLES, Y ELIMINANDO LA RUTA DE ETH0 (POST-UP), Y QUITÁNDOSELA A ETH1
UTILIZA GROUP VARS PARA EL TEMPLATE DE INTERFACES

dirección red
interfaz salida
direccion router
mascara

También un notify reboot para cuando cambie el interfaces

## Rol cliente

```
- name: Borrando ruta por defecto...
  command: ip route del default

- name: Añadiendo nueva ruta por defecto...
  command: ip route add default via 10.0.0.1

```

RUTAS NO PERSISTENTES

AÚN ESTANDO FORWARDING ACTIVADO NO FUNCIONA EL PASO DE PAQUETES







# ENTREGA

1. Desde cliente hacer ping a nombre de una web. Mostrar rutas para comprobar el camino.

2. Entrega una captura de pantalla accediendo por ssh a las dos máquinas. Configura el sistema para que podamos acceder acceder a las máquinas por ssh. Te doy algunas ideas:

(desde nuestro host, sin vagrant ssh?)

    Puedes usar las claves privadas generadas para cada una de las máquinas, o puedes generar nuevas claves que introduces en las máquinas.
    A lo mejor te viene bien la opción -A de ssh.
    Estudia el fichero ~/.ssh/config. Configurando de forma adecuada el fichero de configuración de ssh en tu equipo hasta puedes hacer que se conecte directamente con cliente.
    Las conexiones ssh nuncan las tienes que realizar por eth0.

CREAR FICHERO CONFIG EN SSH, DONDE SE CONFIGUREN CONEXIONES Y SE ACCEDA SOLO POR NOMBRE

EN LA DEFINICIÓN DE CLIENTE, PROXYJUMP ROUTER COMO SI PUSIERA UN -A EN SSH, PARA QUE ACCEDA POR SSH A UNO Y LUEGO AL OTRO


3. Estudia la forma de integrar la receta ansible en vagrant, de tal manera que una vez se cree el escenario se ejecuta la configuración. Enseñale el funcionamiento al profesor.
