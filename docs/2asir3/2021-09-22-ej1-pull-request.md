---
title: "Ejercicio 1: Pull Request"
---

# Paso 1

Hacemos fork del repositorio https://github.com/josedom24/prueba-pr-asir para tenerlo en nuestra cuenta de Github.



# Paso 2

Clonamos este repositorio *(el repo de nuestra cuenta, no el original)* a nuestra máquina

```
git clone git@github.com:adriasir123/prueba-pr-asir.git
```



# Paso 3

Creamos una nueva branch para los cambios

```
git checkout -b adrian_jaramillo
```

> Github al aceptar Pull Requests fusiona branches, de ahí que tengamos que crear una



# Paso 4

Añadimos un nuevo remote, apuntando al repositorio original.  
Hacemos esto para poder hacer pull desde la versión original del repositorio, y traernos siempre los últimos cambios.

```
git remote add upstream https://github.com/josedom24/prueba-pr-asir.git
```

> Como curiosidad, `git remote show` muestra la lista de remotes. Ahora tengo origin y upstream.

Ahora tenemos que hacer lo siguiente para enlazar la branch main de upstream, a mi branch personalizada.

```
git branch --set-upstream-to=upstream/main adrian_jaramillo
```

Después de haber hecho esto, AHORA SÍ puedo hacer pull desde upstream, aunque me aparece el siguiente warning:

```
hint: Pulling without specifying how to reconcile divergent branches is
hint: discouraged. You can squelch this message by running one of the following
hint: commands sometime before your next pull:
hint:
hint:   git config pull.rebase false  # merge (the default strategy)
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint:
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
```

A pesar de ser tan intimidante, podemos hacer pull sin problemas.



# Paso 5

Antes de modificar el repositorio, **tenemos que asegurarnos de que estamos en la última versión del repositorio original**, para no quedarnos atrás en commits y crear conflictos.

Para ello, hacemos

```
atlas@olympus:~/Desktop/iaw/ud1-intro/prueba-pr-asir$ git status
On branch adrian_jaramillo
Your branch is up to date with 'upstream/main'.
```

En el caso de habernos quedado atrás en commits, haríamos

```
git pull
```

# Paso 6

En el paso anterior, nos hemos asegurado de estar en la última versión del repositorio.

Procedemos a hacer los siguientes cambios...

## Cambio 1

Modifico README.md para añadir un enlace a la lista, con mis iniciales "ajr" y apuntando al fichero que crearé en el directorio files

```
**¿Qué asignatura te gusta más? Y ¿por qué?**

* [jdmr](files/jdmr.md)
* [ada](files/ada.md)
* [lpt](files/lpt.md)
* [cmgpdg](files/cmgpdg.md)
* [dpg](files/dpg.md)
* [dpg2](files/dpg2.md)
* [oeb](files/oeb.md)
* [ajr](files/ajr.md)

```

Al final de esa lista, vemos mi línea añadida.

## Cambio 2

Creo el fichero files/ajr.md

```
atlas@olympus:~/Desktop/iaw/ud1-intro/prueba-pr-asir/files$ ls -la
total 40
drwxr-xr-x 2 atlas atlas 4096 Sep 23 06:50 .
drwxr-xr-x 4 atlas atlas 4096 Sep 23 06:41 ..
-rw-r--r-- 1 atlas atlas  418 Sep 22 13:44 ada.md
-rw-r--r-- 1 atlas atlas  400 Sep 23 06:50 ajr.md
-rw-r--r-- 1 atlas atlas  119 Sep 22 14:16 cmgpdg.md
-rw-r--r-- 1 atlas atlas  313 Sep 22 14:16 dpg.md
-rw-r--r-- 1 atlas atlas  166 Sep 22 13:44 jdmr.md
-rw-r--r-- 1 atlas atlas 1024 Sep 22 14:16 .jdmr.md.swp
-rw-r--r-- 1 atlas atlas  265 Sep 22 14:16 lpt.md
-rw-r--r-- 1 atlas atlas  115 Sep 22 14:28 oeb.md
```

Lo vemos en la segunda fila, y su contenido es el siguiente

```
# ¿Qué asignatura te gusta más?

Sinceramente nunca tuve una asignatura preferida, me gusta aprender en general.  
Pero si tengo que elegir, siempre me inclino más por cualquier tema relacionado con sysadmin:
- **Sistemas operativos**
- **Redes**
- **¡¡VIRTUALIZACIÓN!!**

## ¿Por qué?

*¿Necesitamos un por qué para dar una explicación lógica a nuestros gustos? ;)*

> Quote de ejemplo
```

# Paso 7

Añadimos nuestros cambios para el siguiente commit

```
git add .
```

> Este comando añade **TODOS los cambios del repositorio** al siguiente commit

Hacemos el commit, con un mensaje significativo

```
git commit -m "Modificaciones de AJR"
```

Hacemos push a **nuestro repo**

```
git push origin
```



# Paso 8

Después del push, tendremos en github el botón "Compare & Pull Request"

![](https://i.postimg.cc/PJ7dC3YC/Screenshot-from-2021-09-23-08-25-58.png)

Aquí muestro el pull request

![](https://i.postimg.cc/7PtHt6dq/Screenshot-from-2021-09-23-08-36-21.png)

¿Lo siguiente? Esperar a que el dueño del repositorio acepte los cambios.

# Paso 9

Para finalizar este ejercicio, mi repositorio se debe actualizar con todos los cambios posteriores que mis compañeros hagan

```
git pull
git push origin
```

URL a mi repositorio: https://github.com/adriasir123/prueba-pr-asir/tree/adrian_jaramillo
