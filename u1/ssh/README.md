# Acceso remoto SSH

# 1. Preparativos

Vamos a necesitar las siguientes MVs:

| Función | Sistema Operativo     | IP        | Nombre |
| ------- |--------------------- | --------- | --------- |
| Un servidor SSH| GNU/Linux OpenSUSE (Sin entorno gráfico)| 172.19.14.31 | server14g |
| Un cliente SSH | GNU/Linux OpenSUSE | 172.19.14.32 | client14g |
| Un servidor SSH | Windows Server| 172.19.14.11 | server14s |
| Un cliente SSH | Windows | 172.19.14.12 | cliente14w |

## 1.1 Servidor SSH
Configuraremos el servidor GNU/Linux con siguientes valores:
* SO GNU/Linux: OpenSUSE - Sin entorno gráfico
* Nombre de equipo: `server14g`
* Pondremos una clave compleja al usuario root. (server14123)
* Añadiremos en `/etc/hosts` los equipos `client14g` y `client14w`.

![](./images/1.PNG)

Comprobaremos los cambios ejecutando los siguientes comandos:
* `ip a`

![](./images/2.PNG)
* `ip route`

![](./images/3.PNG)
* `ping 8.8.4.4 -i 2`

![](./images/4.PNG)
* `host www.nba.com`

![](./images/5.PNG)
* `ping client14g (con IP de casa)`

![](./images/6.PNG)
* `ping client14w (con IP de casa)`

![](./images/7.PNG)
*`lsblk`

![](./images/8.PNG)
* `blkid`

![](./images/9.PNG)

Crearemos los siguientes usuarios en server14g:
* `alvarez1`, `alvarez2`, `alvarez3`, `alvarez4`

![](./images/10.PNG)

## 1.2 Cliente GNU/Linux
Configuraremos el cliente1 GNU/Linux con los siguientes valores:
* SO OpenSUSE
* Nombre de equipo: `client14g`
* Añadir en `/etc/hosts` los equipos `server14g`, y `client14w`.

![](./images/11.PNG)
* Vamos a comprobar con ping que conecta a los siguientes equipos:

1) `client14g`

![](./images/12.PNG)

2) `client14w`

![](./images/13.PNG)


## 1.3 Cliente Windows
Vamos a instalar el software cliente SSH en Windows. Para este ejemplo usaremos PuTTY.
![](./images/14.PNG)
Configurar el cliente2 Windows con los siguientes valores:
* SO Windows
* Nombre de equipo: `client14w`
* Añadir en C:\Windows\System32\drivers\etc\hosts los equipos server14g y client14g.

![](./images/15.PNG)
* Comprobar haciendo `ping` a ambos equipos.

![](./images/16.PNG)

![](./images/17.PNG)

# 2 Instalación del servicio SSH
Instalaremos el servicio SSH en la máquina `server14g`.\
`zypper install openssh`

![](./images/18.PNG)

## 2.1 Comprobación
Desde el propio servidor, verificar que el servicio está en ejecución.
* `systemctl status sshd`, esta es la forma habitual de comprobar los servicios.

![](./images/19.PNG)

* `ps -ef|grep sshd`, esta es otra forma de comprobarlo mirando los procesos del sistema.

![](./images/20.PNG)

> Si el servicio estuviera apagado, ejecutaremos lo siguiente:
* systemctl enable sshd

* `sudo lsof -i:22`, comprobar que el servicio está escuchando por el puerto 22.

![](./images/21.PNG)


## 2.2 Primera conexión SSH desde cliente GNU/Linux
Ir al cliente client14g.

* `ping server14g`, comprobar la conectividad con el servidor.
![](./images/22.PNG)

* Con `nmap -Pn server14g`, comprobaremos los puertos abiertos en el servidor (SSH debe estar open). Debe mostrarnos que el puerto 22 está abierto. Si esto falla, debemos comprobar en el servidor la configuración del cortafuegos.
![](./images/23.PNG)

Vamos a comprobar el funcionamiento de la conexión SSH desde cada cliente usando el usuario `alvarez1`.

* Desde el cliente GNU/Linux nos conectamos mediante `ssh alvarez1@server14g`. Capturar imagen del intercambio de claves que se produce en el primer proceso de conexión SSH.
![](./images/24.PNG)

* A partir de ahora cuando nos conectamos sólo nos pide la contraseña:
![](./images/25.PNG)

* Comprobar contenido del fichero `$HOME/.ssh/known_hosts` en el equipo cliente. OJO el prompt nos indica en qué equipo estamos.
![](./images/26.PNG)

¿Te suena la clave que aparece? Es la clave de identificación de la máquina del servidor.
* Una vez llegados a este punto deben de funcionar correctamente las conexiones SSH desde el cliente. Comprobarlo.
![](./images/27.PNG)

## 2.3 Primera conexión SSH desde cliente Windows
Desde el cliente Windows nos conectamos usando PuTTY
* Capturaremos una imagen del intercambio de claves que se produce en el primer proceso de conexión SSH.
![](./images/28.PNG)

¿Te suena la clave que aparece? Es la clave de identificación de la máquina del servidor.\
Una vez llegados a este punto deben de funcionar correctamente las conexiones SSH desde el cliente. Comprobarlo.\
La siguiente vez que volvamos a usar PuTTY ya no debe aparecer el mensaje de advertencia porque hemos memorizado la identificación del servidor SSH. Comprobarlo.
![](./images/29.PNG)

# 3. Cambiamos la identidad del servidor
¿Qué pasaría si cambiamos la identidad del servidor? Esto es, ¿Y si cambiamos las claves del servidor? ¿Qué pasa?

* Los ficheros `ssh_host*key` y `ssh_host*key.pub`, son ficheros de clave pública/privada que identifican a nuestro servidor frente a nuestros clientes. Confirmar que existen el en `/etc/ssh`,:
![](./images/30.PNG)

* Modificar el fichero de configuración SSH (/etc/ssh/sshd_config) para dejar una única línea: HostKey /etc/ssh/ssh_host_rsa_key. Comentar el resto de líneas con configuración HostKey. Este parámetro define los ficheros de clave publica/privada que van a identificar a nuestro servidor. Con este cambio decimos que sólo se van a utilizar las claves del tipo RSA.
![](./images/31.PNG)

## 3.1 Regenerar certificados
Vamos a cambiar o volver a generar nuevas claves públicas/privadas que identifican nuestro servidor.

* Ir al servidor.
* Como usuario root ejecutamos: `ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key`. ¡OJO! No poner password al certificado.
![](./images/32.PNG)

* Reiniciaremos el servicio SSH: `systemctl restart sshd`.

![](./images/33.PNG)

* Comprobar que el servicio está en ejecución correctamente: `systemctl status sshd`.
![](./images/34.PNG)

## 3.2 Comprobamos
Comprobar qué sucede al volver a conectarnos desde los dos clientes, usando los usuarios alvarez2 y alvarez1. ¿Qué sucede?
