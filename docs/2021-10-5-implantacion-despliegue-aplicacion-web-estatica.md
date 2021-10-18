---
title: "Práctica: Implantación y despliegue aplicación web estática"
---

# Parte 1
> Elije generador de páginas estáticas y servicio donde desplegar la página web

- MkDocs
- Neocities



# Parte 2
> Documentar la instalación de MkDocs en el entorno de desarrollo.  

```
sudo apt install pip
pip install mkdocs
```

Generamos la estructura inicial
```
mkdocs new sysadmin-docs
```
```
INFO     -  Creating project directory: sysadmin-docs
INFO     -  Writing config file: sysadmin-docs/mkdocs.yml
INFO     -  Writing initial docs: sysadmin-docs/docs/index.md
```

Entramos
```
cd sysadmin-docs
```

Esta es la estructura generada
```
vagrant@webestatica:~/sysadmin-docs$ tree
.
├── docs
│   └── index.md
└── mkdocs.yml

1 directory, 2 files
```

*Estando en la raíz* de nuestro proyecto, ejecutamos
```
mkdocs serve -a 0.0.0.0:8000
```
Lo ejecutamos de esta manera porque mi entorno de desarrollo es una VM, y necesito acceso externo.

```
INFO     -  Building documentation...
WARNING  -  Config value: 'dev_addr'. Warning: The use of the IP address '0.0.0.0' suggests a production environment or the use of a proxy to connect to the MkDocs server. However, the
            MkDocs' server is intended for local development purposes only. Please use a third party production-ready server instead.
INFO     -  Cleaning site directory
INFO     -  Documentation built in 0.05 seconds
INFO     -  [08:43:27] Serving on http://0.0.0.0:8000/
```

Nos aparece ese warning, pero en nuestro caso no es relevante y lo ignoramos.

Como vemos, `mkdocs serve` genera todo el código necesario para que la web funcione, e inicia un servidor web para que podamos acceder a ella.

Entramos a nuestra web:
![](https://i.postimg.cc/B66s6RSb/Screenshot-from-2021-10-07-10-55-39.png)

**¡¡Dato importante!** El servidor web que lanza MkDocs soporta **auto-reloading**.

Esto significa que **ante cualquier cambio** de configuración, ficheros, tema... etc:

- El servidor volverá a generar todo el código necesario
- Nuestro navegador se autorecargará (*¡no hace falta hacer f5!*) y veremos los cambios automáticamente

> ¿En qué lenguaje está desarrollado MkDocs?

Python

> ¿Qué sistema de plantillas utiliza?

Jinja2



# Parte 3
> Cambiar el nombre de la página

El fichero a modificar es `mkdocs.yml`, con el siguiente contenido:
```
site_name: Sysadmin Docs
```

Nuestra página ahora se vería así:
![](https://i.imgur.com/iUT70c3.png)  

> Cambiar el tema de la página

Instalamos el tema:
```
pip install mkdocs-windmill
```

Modificamos `mkdocs.yml`
```
site_name: Sysadmin Docs
theme: windmill
```

Así se vería nuestra página tras modificar el tema:
![](https://i.imgur.com/pCbTw0i.png)



# Parte 4
> Añade 3 páginas a la web con:
- Encabezado
- Párrafos
- Enlace
- Lista
- Imagen

## Página about

1. Añadimos `docs/about.md`
2. Modificamos `mkdocs.yml` para definir la nav bar, y que la página sea accesible:

```
site_name: Sysadmin Docs
theme: windmill
nav:
    - Home: index.md
    - About: about.md
```

3. Así se vería:
![](https://i.imgur.com/8xMDfqw.png)

## Página "Sección 1"

1. Añadimos `docs/seccion1.md`
2. Modificamos `mkdocs.yml` para que la página sea accesible:

```
site_name: Sysadmin Docs
theme: windmill
nav:
    - Home: index.md
    - Sección 1: seccion1.md
    - About: about.md
```

3. Así se vería:
![](https://i.imgur.com/HUQNt0Z.png)

## Página "Sección 2"

1. Añadimos `docs/seccion2.md`
2. Modificamos `mkdocs.yml` para que la página sea accesible:

```
site_name: Sysadmin Docs
theme: windmill
nav:
    - Home: index.md
    - Sección 1: seccion1.md
    - Sección 2: seccion2.md
    - About: about.md
```

3. Así se vería:
![](https://i.imgur.com/oXVFAHZ.png)


> El código que estas desarrollando, configuración del generado, páginas en markdown... todo debe estar en un repositorio Git (no es necesario que el código generado se guarde en el repositorio, evítalo usando el fichero .gitignore)

Repositorio: https://github.com/adriasir123/practica-app-web-estatica-iaw-21-22

En el fichero `.gitignore` se excluye:
```
site/
```



# Parte 5
> Explicar el proceso de despliegue en Neocities

## Generar ficheros

Si no tenemos HTML, ¿no tenemos web que desplegar verdad? Por ello, antes de nada, generamos el código:
```
mkdocs build
```
```
INFO     -  Cleaning site directory
INFO     -  Building documentation to directory: /home/vagrant/sysadmin-docs/site
INFO     -  Documentation built in 0.04 seconds
```

Vemos que nos ha generado un directorio `site` con todo lo necesario. Esto nos encontramos:
```
vagrant@webestatica:~/sysadmin-docs$ ls -la site/
total 64
drwxr-xr-x 10 vagrant vagrant 4096 Oct 10 21:28 .
drwxr-xr-x  5 vagrant vagrant 4096 Oct 10 21:28 ..
-rw-r--r--  1 vagrant vagrant 1858 Oct 10 21:28 404.html
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 about
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 css
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 fonts
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 img
-rw-r--r--  1 vagrant vagrant 5899 Oct 10 21:28 index.html
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 js
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 search
-rw-r--r--  1 vagrant vagrant 2054 Oct 10 21:28 search.html
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 seccion1
drwxr-xr-x  2 vagrant vagrant 4096 Oct 10 21:28 seccion2
-rw-r--r--  1 vagrant vagrant  609 Oct 10 21:28 sitemap.xml
-rw-r--r--  1 vagrant vagrant  197 Oct 10 21:28 sitemap.xml.gz
```

## Despliegue en Neocities

### Crear cuenta
https://neocities.org/

Nuestra web será accesible en: https://sysadmin-docs.neocities.org/

### Instalar Neocities CLI
https://neocities.org/cli

```
sudo apt update
sudo apt install rubygems
sudo gem install neocities
```

Una vez instalada la herramienta, *¡no estamos logeados!*

Para conseguir esto no existe un comando concreto... al ejecutar cualquier orden que necesite comunicarse con el hosting nos pedirá logearnos.

Por ejemplo:
```
neocities list /
```

Con este comando mostramos ficheros remotos, y por lo tanto nos pide autenticación:
```
Please login to get your API key:
sitename: sysadmin-docs
password: •••••••••••••••
The api key for sysadmin-docs has been stored in /home/vagrant/.config/neocities/config.
index.html
```

*¡Cuidado!* En el sitename escribimos **sólo** la primera parte de nuestro nombre web.

Neocities genera una estructura web de prueba, que tendremos que borrar.  
En mi caso borré todo menos `index.html`... ni siquiera desde la CLI nos permite borrarlo.

### Subir ficheros

El siguiente comando subirá recursivamente todo **el contenido** del directorio que le indiquemos (*¡su **contenido**, el directorio indicado no lo sube!*)
```
neocities push site/
```
```
Uploading search.html ... SUCCESS
Uploading sitemap.xml ... SUCCESS
Uploading js/elasticlunr.js ... SUCCESS
Uploading js/elasticlunr.min.js ... SUCCESS
Uploading js/bootstrap-3.3.7.js ... SUCCESS
Uploading js/highlight.pack.js ... SUCCESS
Uploading js/jquery-3.2.1.js ... SUCCESS
Uploading js/base.js ... SUCCESS
Uploading js/bootstrap-3.3.7.min.js ... SUCCESS
Uploading js/jquery-3.2.1.min.js ... SUCCESS
Uploading sitemap.xml.gz ...
ERROR: sitemap.xml.gz is not a valid file type (or contains not allowed content) for this site, files have not been uploaded (invalid_file_type)
Uploading seccion1/index.html ... SUCCESS
Uploading about/index.html ... SUCCESS
Uploading 404.html ... SUCCESS
Uploading css/base.css ... SUCCESS
Uploading css/font-awesome-4.7.0.css ... SUCCESS
Uploading css/highlight.css ... SUCCESS
Uploading css/bootstrap-3.3.7.min.css ... SUCCESS
Uploading css/font-awesome-4.7.0.min.css ... SUCCESS
Uploading css/bootstrap-3.3.7.css ... SUCCESS
Uploading seccion2/index.html ... SUCCESS
Uploading img/favicon.ico ... SUCCESS
Uploading search/search_index.json ... SUCCESS
Uploading fonts/glyphicons-halflings-regular.svg ... SUCCESS
Uploading fonts/fontawesome-webfont.ttf ... SUCCESS
Uploading fonts/fontawesome-webfont.woff2 ... SUCCESS
Uploading fonts/fontawesome-webfont.eot ... SUCCESS
Uploading fonts/fontawesome-webfont.svg ... SUCCESS
Uploading fonts/glyphicons-halflings-regular.woff ... SUCCESS
Uploading fonts/glyphicons-halflings-regular.woff2 ... SUCCESS
Uploading fonts/glyphicons-halflings-regular.ttf ... SUCCESS
Uploading fonts/fontawesome-webfont.woff ... SUCCESS
Uploading fonts/glyphicons-halflings-regular.eot ... SUCCESS
Uploading index.html ... SUCCESS
```
Vemos que hay un error, pero no es relevante. Es un fichero `.gz` que MkDocs crea, pero no tiene función en que la web funcione o no.

`neocities push` sobreescribe, por lo tanto el antiguo `index.html` que no podíamos borrar, ya está reemplazado.

Si queremos hacer una prueba de push, sin que suba nada, podemos hacer:
```
neocities push --dry-run site/
```

Teniendo el código subido, ya tendremos nuestra web:
![](https://i.imgur.com/CjevHqk.png)



# Parte 6
> Usar Git Hook para automatizar: generación de la página + despliegue en Neocities.  
Mostrar al profesor un ejemplo de como al modificar la página se realiza la puesta en producción de forma automática.

Creamos el fichero de Git Hook:
```
touch .git/hooks/pre-push
```

Su contenido será:
```
#!/bin/sh

mkdocs build
neocities push site/
```

Lo hacemos ejecutable:
```
chmod u+x .git/hooks/pre-push
```

Un Git Hook de "pre-push" es un script que se ejecuta *antes* de que la acción de push tome efecto. Por lo tanto, al hacer `git push`, esto pasaría:
```
INFO     -  Cleaning site directory
INFO     -  Building documentation to directory: /home/vagrant/sysadmin-docs/site
INFO     -  Documentation built in 0.11 seconds
Uploading search.html ... EXISTS
Uploading sitemap.xml ... EXISTS
Uploading js/elasticlunr.js ... EXISTS
Uploading js/elasticlunr.min.js ... EXISTS
Uploading js/bootstrap-3.3.7.js ... EXISTS
Uploading js/highlight.pack.js ... EXISTS
Uploading js/jquery-3.2.1.js ... EXISTS
Uploading js/base.js ... EXISTS
Uploading js/bootstrap-3.3.7.min.js ... EXISTS
Uploading js/jquery-3.2.1.min.js ... EXISTS
Uploading sitemap.xml.gz ...
ERROR: sitemap.xml.gz is not a valid file type (or contains not allowed content) for this site, files have not been uploaded (invalid_file_type)
Uploading seccion1/index.html ... EXISTS
Uploading about/index.html ... SUCCESS
Uploading 404.html ... EXISTS
Uploading css/base.css ... EXISTS
Uploading css/font-awesome-4.7.0.css ... EXISTS
Uploading css/highlight.css ... EXISTS
Uploading css/bootstrap-3.3.7.min.css ... EXISTS
Uploading css/font-awesome-4.7.0.min.css ... EXISTS
Uploading css/bootstrap-3.3.7.css ... EXISTS
Uploading seccion2/index.html ... EXISTS
Uploading img/favicon.ico ... EXISTS
Uploading search/search_index.json ... SUCCESS
Uploading fonts/glyphicons-halflings-regular.svg ... EXISTS
Uploading fonts/fontawesome-webfont.ttf ... EXISTS
Uploading fonts/fontawesome-webfont.woff2 ... EXISTS
Uploading fonts/fontawesome-webfont.eot ... EXISTS
Uploading fonts/fontawesome-webfont.svg ... EXISTS
Uploading fonts/glyphicons-halflings-regular.woff ... EXISTS
Uploading fonts/glyphicons-halflings-regular.woff2 ... EXISTS
Uploading fonts/glyphicons-halflings-regular.ttf ... EXISTS
Uploading fonts/fontawesome-webfont.woff ... EXISTS
Uploading fonts/glyphicons-halflings-regular.eot ... EXISTS
Uploading index.html ... SUCCESS
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 529 bytes | 529.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:adriasir123/practica-app-web-estatica-iaw-21-22.git
   e0be61b..ecf9afe  main -> main
```

He añadido una línea a la página "About" para hacer esta prueba:
![](https://i.imgur.com/5L3GMGc.png)

Esa línea se ha añadido automáticamente al hacer push, porque el hook generaba la página y luego hacía push a Neocities.
