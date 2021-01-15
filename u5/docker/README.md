

# 1. Contenedores con Docker

Es muy común que nos encontremos desarrollando una aplicación, y llegue el momento que decidamos tomar todos sus archivos y migrarlos, ya sea al ambiente de producción, de prueba, o simplemente probar su comportamiento en diferentes plataformas y servicios.

Para situaciones de este estilo existen herramientas que, entre otras cosas, nos facilitan el embalaje y despliegue de la aplicación, es aquí donde entra en juego los contenedores (Por ejemplo Docker o Podman).

Esta herramienta nos permite crear "contenedores", que son aplicaciones empaquetadas auto-suficientes, muy livianas, capaces de funcionar en prácticamente cualquier ambiente, ya que tiene su propio sistema de archivos, librerías, terminal, etc.

Docker es una tecnología contenedor de aplicaciones construida sobre LXC.

## 1.1 Instalación

Ejecutar como superusuario:
* `zypper in docker`, instalar docker en OpenSUSE (`apt install docker` en Debian/Ubuntu).

    ![](./images/1.png)

* `systemctl start docker`, iniciar el servicio. NOTA: El comando `docker daemon` hace el mismo efecto.
* `systemctl enable docker`, si queremos que el servicio de inicie automáticamente al encender la máquina.

    ![](./images/2.png)

* **IMPORTANTE**: Incluiremos a nuestro usuario (diego) como miembro del grupo `docker`. Solamente los usuarios dentro del grupo `docker` tendrán permiso para usarlo.

    ![](./images/3.png)

Iniciaremos sesión como usuario normal.
* `docker version`, comprobamos que se muestra la información de las versiones cliente y servidor.

    ![](./images/4.png)

* **OJO**: A partir de ahora todo lo haremos con nuestro usuario, sin usar `sudo`.

## 1.1 Habilitar el acceso a la red externa a los contenedores

Si queremos que nuestro contenedor tenga acceso a la red exterior, debemos activar tener activada la opción IP_FORWARD (`net.ipv4.ip_forward`).

* `cat /proc/sys/net/ipv4/ip_forward` para consultar el estado de IP_FORWARD (desactivado = 0, activo = 1).

![](./images/5.png)

En caso de estar a 0, para activarlo podemos hacerlo de diversas formas:
* Poner el valor 1 en el fichero de texto indicado.
* O ejecutar el siguiente comando `sysctl -w net.ipv4.ip_forward=1`
* También podemos crear el fichero `/etc/sysctl.d/diego14.conf` y poner dentro lo siguiente:
    ```
    ## Configuración para docker de diego14
    net.ipv4.ip_forward = 1
    ```
* También podemos **Usar YAST para activar IP_FORWARD**:

| Sistema operativo | Activar "forwarding" |
| ----------------- | -------------------- |
| OpenSUSE Leap (configuración de red es Wicked) | Yast -> Dispositivos de red -> Encaminamiento -> Habilitar reenvío IPv4 |
| Cuando la red está gestionada por Network Manager | En lugar de usar YaST debemos editar el fichero "/etc/sysconfig/SuSEfirewall2" y poner FW_ROUTE="yes" |
| OpenSUSE Tumbleweed  | Yast -> Sistema -> Configuración de red -> Menú de encaminamiento |

* Reiniciaremos el equipo para que se aplique el cambio de configuración anterior.

## 1.3 Primera prueba

* `docker run hello-world`, este comando hace lo siguiente:
    * Descarga una imagen "hello-world"
    * Crea un contenedor y
    * ejecuta la aplicación que hay dentro.

    ![](./images/6.png)

* `docker images`, ahora vemos la nueva imagen "hello-world" descargada en nuestro equipo local.
* `docker ps -a`, vemos que hay un contenedor en estado 'Exited'.
* `docker stop IDContainer`, parar el conteneder.
* `docker rm IDContainer`, eliminar el contenedor.

    ![](./images/7.png)


## 1.4 Sólo para LEER

Veamos un poco de teoría.

Tabla de referencia para no perderse:

| Software   | Base   | Sirve para crear   | Aplicaciones |
| ---------- | ------ | ------------------ | ------------ |
| VirtualBox | ISO    | Máquinas virtuales | N |
| Vagrant    | Box    | Máquinas virtuales | N |
| Docker     | Imagen | Contenedores       | 1 |


Comandos útiles de Docker:

| Comando                   | Descripción           |
| ------------------------- | --------------------- |
| docker stop CONTAINERID   | Parar un contenedor   |
| docker start CONTAINERID  | Iniciar un contenedor |
| docker attach CONTAINERID | Conectar el terminal actual con el contenedor |
| docker ps                 | mostrar los contenedores en ejecución |
| docker ps -a              | mostrar todos los contenedores (en ejecución o no) |
| docker rm CONTAINERID     | Eliminar un contenedor |
| docker rmi IMAGENAME      | Eliminar una imagen    |

## 1.5 Alias

Para ayudarnos a trabajar de forma más rápida con la línea de comandos podemos agregar los siguientes alias al fichero `/home/diego/.alias`:

```
alias di='docker images'
alias dp='docker ps'
alias dpa='docker ps -a'
alias drm='docker rm '
alias drmi='docker rmi '
alias ds='docker stop '
```

# 2. Creación manual de nuestra imagen

Nuestro SO base es OpenSUSE, pero vamos a crear un contenedor Debian,
y dentro instalaremos Nginx.

## 2.1 Crear un contenedor manualmente

**Descargar una imagen**
* `docker search debian`, buscamos en los repositorios de Docker Hub contenedores con la etiqueta `debian`.

    ![](./images/8.png)

* `docker pull debian`, descargamos una imagen en local.
* `docker images`, comprobamos.

    ![](./images/9.png)

**Crear un contenedor**: Vamos a crear un contenedor con nombre `app1debian` a partir de la imagen `debian`, y ejecutaremos el programa `/bin/bash` dentro del contendor:
* `docker run --name=app1debian -i -t debian /bin/bash`

    ![](./images/10.png)


## 2.2 Personalizar el contenedor

Ahora estamos dentro del contenedor, y vamos a personalizarlo a nuestro gusto:

**Instalar aplicaciones dentro del contenedor**

```
root@IDContenedor:/# cat /etc/motd            # Comprobamos que estamos en Debian
root@IDContenedor:/# apt-get update
root@IDContenedor:/# apt-get install -y nginx # Instalamos nginx en el contenedor
root@IDContenedor:/# apt-get install -y vim   # Instalamos editor vi en el contenedor
```

![](./images/11.png)

![](./images/12.png)

![](./images/13.png)

**Crear un fichero HTML** `holamundo1.html`.

```
root@IDContenedor:/# echo "<p>Hola Diego</p>" > /var/www/html/holamundo1.html
```

![](./images/14.png)

**Crear un script** `/root/server.sh` con el siguiente contenido:

```
#!/bin/bash
echo "Booting Nginx!"
/usr/sbin/nginx &

echo "Waiting..."
while(true) do
  sleep 60
done
```

![](./images/15.png)

Recordatorio:
* Hay que poner permisos de ejecución al script para que se pueda ejecutar.

    ![](./images/18.png)

* La primera línea de un script, siempre debe comenzar por `#!/`, sin espacios.
* Este script inicia el programa/servicio y entra en un bucle, para mantener el contenedor activo y que no se cierre al terminar la aplicación.

## 2.3 Crear una imagen a partir del contenedor

Ya tenemos nuestro contenedor auto-suficiente de Nginx, ahora debemos vamos a crear una nueva imagen que incluya los cambios que hemos hecho.

* Abrir otra ventana de terminal.
* `docker commit app1debian diego/nginx1`, a partir del CONTAINERID vamos a crear la nueva imagen que se llamará "diego/nginx1".
> NOTA:
>
> * Los estándares de Docker estipulan que los nombres de las imágenes deben seguir el formato `nombreusuario/nombreimagen`.
> * Todo cambio realizado que se acompañe de un commit a la imagen, se perderá en cuanto se cierre el contenedor.

* `docker images`, comprobamos que se ha creado la nueva imagen.
* Ahora podemos parar el contenedor, `docker stop app1debian` y
* Eliminar el contenedor, `docker rm app1debian`.

    ![](./images/17.png)


# 3. Crear contenedor a partir de nuestra imagen

## 3.1 Crear contenedor con Nginx

Ya tenemos una imagen "diego/nginx" con Nginx instalado.
* `docker run --name=app2nginx1 -p 80 -t diego/nginx1 /root/server.sh`, iniciar el contenedor a partir de la imagen anterior.

    ![](./images/19.png)

> El argumento `-p 80` le indica a Docker que debe mapear el puerto especificado del contenedor, en nuestro caso el puerto 80 es el puerto por defecto sobre el cual se levanta Nginx.

## 3.2 Comprobamos

* Abrimos una nueva terminal.
* `docker ps`, nos muestra los contenedores en ejecución. Podemos apreciar que la última columna nos indica que el puerto 80 del contenedor está redireccionado a un puerto local `0.0.0.0.:PORT -> 80/tcp`.

    ![](./images/20.png)

* Abrir navegador web y poner URL `0.0.0.0.:PORT`. De esta forma nos conectaremos con el servidor Nginx que se está ejecutando dentro del contenedor.

    ![](./images/21.png)

* Comprobar el acceso a `holamundo1.html`.

    ![](./images/22.png)

* Paramos el contenedor `app2nginx1` y lo eliminamos.

    ![](./images/23.png)

Como ya tenemos una imagen docker con Nginx (Servidor Web), podremos crear nuevos contenedores cuando lo necesitemos.


## 3.3 Migrar la imagen a otra máquina

¿Cómo puedo llevar los contenedores Docker a un nuevo servidor?

**Exportar** imagen Docker a fichero tar:
* `docker save -o diego14docker.tar diego/nginx1`, guardamos la imagen "diego/nginx1" en un fichero tar.

    ![](./images/24.png)

Intercambiaremos nuestra imagen exportada con la de un compañero de clase.

**Importar** imagen Docker desde fichero:
* Coger la imagen de un compañero de clase.
* Nos llevamos el tar a otra máquina con docker instalado, y restauramos.
* `docker load -i diego14docker.tar`, cargamos la imagen docker a partir del fichero tar. Cuando se importa una imagen se muestra en pantalla las capas que tiene. Las capas las veremos en un momento.
* `docker images`, comprobamos que la nueva imagen está disponible.

    ![](./images/25.png)

* Probar a crear un contenedor (`app3alumno`), a partir de la nueva imagen.

    ![](./images/26.png)

    ![](./images/27.png)

    ![](./images/28.png)


## 3.4 Capas

**Teoría sobre las capas**. Las imágenes de docker están creadas a partir de capas que van definidas en el fichero Dockerfile. Una de las ventajas de este sistema es que esas capas son cacheadas y se pueden compartir entre distintas imágenes, esto es que si por ejemplo la creación de nuestra imagen consta de 10 capas, y modificamos una de esas capas, a la hora de volver a construir la imagen solo se debe ejecutar esta nueva capa, el resto permanecen igual.

Estas capas a parte de ahorrarnos peticiones de red al bajarnos una nueva versión de una imagen también ahorra espacio en disco, ya que las capas que no se hayan cambiado entre versiones no se descargarán.

* `docker image history nombre_imagen:version`, para consultar las capas de la imagen del compañero.

# 4. Dockerfile

Ahora vamos a conseguir el mismo resultado del apartado anterior, pero
usando un fichero de configuración. Esto es, vamos a crear un contenedor a partir de un fichero `Dockerfile`.

## 4.1 Preparar ficheros

* Crear directorio `/home/diego/docker14a`.
* Entrar el directorio anterior.
* Crear fichero `holamundo2.html` con:
    * Proyecto: docker14a
    * Autor: Juan Diego
    * Fecha: 15/01/2021

    ![](./images/29.png)

* Crear el fichero `Dockerfile` con el siguiente contenido:

```
FROM debian

MAINTAINER diego14 1.0

RUN apt-get update
RUN apt-get install -y apt-utils
RUN apt-get install -y nginx

COPY holamundo2.html /var/www/html
RUN chmod 666 /var/www/html/holamundo2.html

EXPOSE 80

CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
```

![](./images/30.png)


## 4.2 Crear imagen a partir del `Dockerfile`

El fichero Dockerfile contiene toda la información necesaria para construir el contenedor, veamos:

* `cd docker14a`, entramos al directorio con el Dockerfile.
* `docker build -t diego/nginx2 .`, construye una nueva imagen a partir del Dockerfile. OJO: el punto final es necesario.

    ![](./images/31.png)

* `docker images`, ahora debe aparecer nuestra nueva imagen.

    ![](./images/32.png)


## 4.3 Crear contenedor y comprobar

A continuación vamos a crear un contenedor con el nombre `app4nginx2`, a partir de la imagen `diego/nginx2`. Probaremos con:

```
docker run --name=app4nginx2 -p 8082:80 -t diego/nginx2
```

![](./images/33.png)

Desde otra terminal:
* `docker ps`, para comprobar que el contenedor está en ejecución y en escucha por el puerto deseado.

    ![](./images/34.png)

* Comprobar en el navegador:
    * URL `http://localhost:PORTNUMBER`

    ![](./images/35.png)

    * URL `http://localhost:PORTNUMBER/holamundo2.html`

    ![](./images/36.png)

Ahora que sabemos usar los ficheros Dockerfile, nos damos cuenta que es más sencillo usar estos ficheros para intercambiar con nuestros compañeros que las herramientas de exportar/importar que usamos anteriormente.


## 4.4 Usar imágenes ya creadas

El ejemplo anterior donde creábamos una imagen Docker con Nginx se puede simplificar aún más aprovechando imágenes oficiales que ya existen.

> Enlace de interés:
> * [nginx - Docker Official Images] https://hub.docker.com/_/nginx

* Crea el directorio `docker14b`. Entrar al directorio.
* Crear fichero `holamundo3.html` con:
    * Proyecto: docker14b
    * Autor: Juan Diego
    * Fecha: 15/01/2021

    ![](./images/37.png)

* Crea el siguiente `Dockerfile`

```
FROM nginx

COPY holamundo3.html /usr/share/nginx/html
RUN chmod 666 /usr/share/nginx/html/holamundo3.html
```

![](./images/38.png)

* Poner el el directorio `docker14b` los ficheros que se requieran para construir el contenedor.
* `docker build -t  diego/nginx3 .`, crear la imagen.

    ![](./images/39.png)

* `docker run --name=app5nginx3 -d -p 8083:80 diego/nginx3`, crearemos el contenedor.

    ![](./images/40.png)

* Comprobaremos el acceso a "holamundo3.html".

    ![](./images/41.png)


# 5. Docker Hub

Ahora vamos a crear un contenedor "hola mundo" y subirlo a Docker Hub.

* Crear carpeta `docker14c`. Entrar en la carpeta.
* Crear fichero Dockerfile de modo que al ejecutar este comando `docker run diego/holamundo` se mostrará en pantalla el mensaje siguiente:
```
Hola Mundo!
diego14
Proyecto docker14c
Fecha actual 15/01/2021
```

    ![](./images/42.png)

> NOTA: Usaremos la imagen base `busybox` y la instrucción RUN o un script para mostrar mensajes por pantalla.

* Registrarse en Docker Hub.
* `docker login`, para abrir la conexión.
* `docker push ...`, para subir la imagen a los repositorios de Docker.

# 6. Limpiar contenedores e imágenes

Cuando terminamos con los contenedores, y ya no lo necesitamos, es buena idea pararlos y/o destruirlos.

* `docker ps -a`, identificar todos los contenedores que tenemos.
* `docker stop ...`, parar todos los contenedores.
* `docker rm ...`, eliminar los contenedores.

Hacemos lo mismo con las imágenes. Como ya no las necesitamos las eliminamos:

* `docker images`, identificar todas las imágenes.
* `docker rmi ...`, eliminar las imágenes.

---
