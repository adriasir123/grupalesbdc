# Herramienta web MongoDB

## MongoDB-PHP-GUI

https://github.com/SamuelTallet/MongoDB-PHP-GUI

### 1. Configuración previa

Antes de instalar el contenedor vamos a instalar docker con los siguientes pasos.

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Actualizar los repositorios e instalar docker.

`sudo apt update`

`sudo apt install docker-ce docker-ce-cli containerd.io`

Comprobar si están activados los servicios.

`systemctl status docker containerd`


### 2. Instalación MongoDB-PHP-GUI

Descargamos el contenedor docker.

`sudo docker pull samueltallet/mongodb-php-gui`

### 3. Conexión

Correr el docker con el siguiente comando:

`sudo docker run --add-host localhost:192.168.122.98 --publish 5000:5000 --rm samueltallet/mongodb-php-gui`

Nos aparece una dirección que tendremos que escribir en el navegador. En mi caso es "http://0.0.0.0:5000/".

![mongo-url](/img/capturas-arantxa/65.png)

![mongo-gui](/img/capturas-arantxa/64.png)

Escribimos "mongodb://admin:admin@192.168.122.98:27017" y hacemos click en "Login".

![mongo-gui2](/img/capturas-arantxa/66.png)

Nos aparecerán las bases de datos que tenemos en MongoDB.

![mongo-gui3](/img/capturas-arantxa/67.png)

Seleccionamos la base de datos *maravilla* y se nos abre un desplegable con las colecciones. Selecciono la colección película.

![mongo-gui4](/img/capturas-arantxa/68.png)

Tenemos opciones para insertar, borrar, contar o buscar por filtro, y las opciones limit, sort y order.

![mongo-gui5](/img/capturas-arantxa/69.png)

También tenemos arriba pestañas con diferentes opciones, entre ellas para administrar los usuarios.

![mongo-gui6](/img/capturas-arantxa/70.png)



### 4. Conexión remota

Para conectarnos desde un cliente remotamente solo tendremos que seguir los pasos anteriores y podremos acceder sin ningún problema.

![mongo-gui-remoto](/img/capturas-arantxa/71.png)

En la siguiente captura vemos la ip del cliente desde el que estamos accediendo y al correr docker nos conectamos a la ip del servidor MongoDB.

![mongo-gui-remoto2](/img/capturas-arantxa/72.png)

Copiamos la dirección en el navegador.

![mongo-gui-remoto3](/img/capturas-arantxa/73.png)

Hacemos "Login" y ya abremos entrado de forma remota al servidor MongoDB.

![mongo-gui-remoto4](/img/capturas-arantxa/74.png)

