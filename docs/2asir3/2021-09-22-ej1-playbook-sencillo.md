---
title: "Ejercicio 1: Playbook sencillo"
---

# Parte 1

> Prepara una máquina virtual en KVM que es la que vamos a configurar de forma automática con ansible.

Ya la tengo, hecha en Vagrant con libvirt.

# Parte 2

Crea un playbook que realice las siguientes tareas de forma automática. Se usan módulos específicos para cada una de ellas.

## Tarea 1
> Crear un usuario "adrianj" en el servidor remoto

```
- name: Creando usuario adrianj...
  user:
    name: adrianj
    password: '$6$Rjn7CUlYJXM8Fvrk$4r/qu2Q6Abiq4iGJjOW2RtclK14AyB9/3va/ZWu3oPIjIo10sNPgoyM6/sthvocwux.7hXFk/O8QpNNUW3Xsr1'
    # He usado mkpasswd --method=sha-512
    # en plano: 1234
```

## Tarea 2
> Descarga el fichero https://wordpress.org/latest.zip

```
- name: Descargando Wordpress...
  get_url:
    url: https://wordpress.org/latest.zip
    dest: /tmp
```

## Tarea 3
> Descomprime ese fichero en el home del usuario "adrianj"

```
- name: Descompresión
  block:
   - name: Instalando unzip...
     become_user: root
     apt:
       pkg:
       - unzip
   - name: Descomprimiendo Wordpress en el home de adrianj...
     unarchive:
       src: /tmp/wordpress-5.8.1.zip
       dest: /home/adrianj
       remote_src: yes
```

## Tarea 4
> Instala mariadb

```
- name: Instalando MariaDB...
  apt:
    pkg:
    - mariadb-server
```


## Tarea 5
> Crea una base de datos que se llame adrian_wordpress

```
- name: Instalar python3-pip + pymysql + crear BD
  block:
   - name: Instalando python3-pip...
     apt:
       pkg:
       - python3-pip
   - name: Instalando pymysql...
     pip:
       name: pymysql
   - name: Creando la base de datos adrian_wordpress...
     mysql_db:
       login_unix_socket: /var/run/mysqld/mysqld.sock
       name: adrian_wordpress
       state: present
```

## Tarea 6
> Crea el usuario "adrian_jaramillo" con privilegios sobre la base de datos "adrian_wordpress"

```
- name: Creando usuario con privilegios sobre adrian_wordpress...
  mysql_user:
    login_unix_socket: /var/run/mysqld/mysqld.sock
    name: adrian_jaramillo
    password: '1234'
    priv: 'adrian_wordpress.*:ALL'
    state: present
```

## Tarea 7
> Clonar repositorio https://github.com/josedom24/ansible_ejemplos.git en el home del usuario "adrianj"

```
- name: Git + directorio vacío + repo
  block:
   - name: Instalando git...
     apt:
       pkg:
       - git
   - name: Creando un directorio vacío para el repo...
     file:
       path: /home/adrianj/repo
       state: directory
   - name: Clonando repo al home de adrianj...
     git:
       repo: https://github.com/josedom24/ansible_ejemplos.git
       dest: /home/adrianj/repo
```



# Parte 3
> Entrega la url del repositorio donde has guardado el playbook

https://github.com/adriasir123/ej1-playbook-sencillo

> Muestra la ejecución del playbook

```
PLAY [nodo1] ********************************************************************************

TASK [Gathering Facts] **********************************************************************
ok: [nodo1]

TASK [Creando usuario adrianj...] ***********************************************************
ok: [nodo1]

TASK [Descargando Wordpress...] *************************************************************
ok: [nodo1]

TASK [Instalando unzip...] ******************************************************************
ok: [nodo1]

TASK [Descomprimiendo Wordpress en el home de adrianj...] ***********************************
ok: [nodo1]

TASK [Instalando MariaDB...] ****************************************************************
ok: [nodo1]

TASK [Instalando python3-pip...] ************************************************************
ok: [nodo1]

TASK [Instalando pymysql...] ****************************************************************
ok: [nodo1]

TASK [Creando la base de datos adrian_wordpress...] *****************************************
ok: [nodo1]

TASK [Creando usuario con privilegios sobre adrian_wordpress...] ****************************
ok: [nodo1]

TASK [Instalando git...] ********************************************************************
ok: [nodo1]

TASK [Creando un directorio vacío para el repo...] ******************************************
ok: [nodo1]

TASK [Clonando repo al home de adrianj...] **************************************************
ok: [nodo1]

PLAY RECAP **********************************************************************************
nodo1                      : ok=13   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
