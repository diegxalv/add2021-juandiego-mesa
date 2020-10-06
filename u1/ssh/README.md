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
* `lsblk`

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
Configurar el cliente 2 de Windows con los siguientes valores:
* SO Windows
* Nombre de equipo: `client14w`
* Añadiremos en `C:\Windows\System32\drivers\etc\hosts` los equipos `server14g` y `client14g`.
* Comprobar haciendo ping a ambos equipos.
