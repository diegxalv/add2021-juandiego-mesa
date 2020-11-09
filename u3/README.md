# Servidor de Impresión en Windows

Necesitaremos 2 MV:
* MV1: Windows Server
* MV2: Windows 7/10 cliente

---

# 1. Impresora compartida

## 1.1 Rol impresión

* Vamos al servidor e instalaremos el rol de servidor de impresión. Incluiremos también impresión por Internet.
![](./images/1.png)

  ![](./images/2.png)
> Instalar también rol/función de cliente de impresión por Internet.
![](./images/3.png)

## 1.2 Instalar impresora PDF

Vamos a conectar e instalar localmente una impresora al servidor Windows Server, de modo que estén disponibles para ser accedidas por los clientes del dominio.

En nuestro caso, vamos a instalar un programa que simule una impresora de PDF.

PDFCreator es una utilidad completamente gratuita con la que podrás crear archivos PDF desde cualquier aplicación, desde el Bloc de notas hasta Word, Excel, etc. Este programa funciona simulando ser una impresora, de esta forma, instalando PDFCreator todas tus aplicaciones con opción para imprimir te permitirán crear archivos PDF en cuestión de segundos.

* Descargar PDFCreator (URL recomendada `www.pdfforge.org/pdfcreator/download`) y lo instalaremos. \
![](./images/4.png)

* En PDFCreator, configurar en `perfiles -> Guardar -> Automático`. Ahí establecemos la carpeta destino. \
![](./images/5.png)

> NOTA: PDFCreator puede requerir NET FrameWork v4.

## 1.3 Probar la impresora en local

Para crear un archivo PDF no hará falta que cambies la aplicación que estés usando, simplemente ve a la opción de imprimir y selecciona "Impresora PDF", en segundos tendrás creado tu archivo PDF. \
![](./images/6.png)

Puedes probar la nueva impresora abriendo el Bloc de notas y creando un fichero luego selecciona imprimir. Cuando finalice el proceso se abrirá un fichero PDF con el resultado de la impresión.

* Probar la impresora remota imprimiendo documento `imprimir14s-local`. \
![](./images/7.png)

![](./images/8.png)
---

# 2. Compartir por red

## 2.1 En el servidor

Vamos a la MV del servidor.
* Iremos al `Administrador de Impresión -> Impresoras`
* Elegiremos la impresora PDFCreator. \
![](./images/9.png)
    * `Botón derecho -> Propiedades -> Compartir`
    * Como nombre del recurso compartido utilizaremos `PDFdiego14`.
    ![](./images/10.png)

## 2.2 Comprobar desde el cliente

Vamos al cliente:
* Buscar recursos de red del servidor. Si tarda en aparecer ponemos `\\172.19.14.21` en la barra de navegación. \
![](./images/11.png)
* Seleccionar impresora -> botón derecho -> conectar.
    * Ponemos usuario/clave del Windows Server. \
    ![](./images/12.png)
* Ya tenemos la impresora remota configurada en el cliente.
* Probar la impresora remota imprimiendo documento `imprimir14w-remoto`. \
![](./images/13.png)

  ![](./images/14.png)

  ![](./images/15.png)

---

# 3. Acceso Web

Realizaremos una configuración para habilitar el acceso web a las impresoras del dominio.

## 3.1 Instalar característica impresión WEB

* Vamos al servidor.
* Nos aseguramos de tener instalado el servicio "Impresión de Internet".

## 3.2 Configurar impresión WEB

Vamos a la MV cliente:
* Abrimos un navegador Web.
* Ponemos URL `http://<ip-del-servidor>/printers`
(o `http://<nombre-del-servidor>/printers`) para que aparezca en nuestro navegador un entorno que permite gestionar las impresoras de dicho equipo, previa autenticación como uno de los usuarios del habilitados para dicho fin (por ejemplo el "Administrador"). \
![](./images/16.png)
* Pincha en la opción propiedades y captura lo que se ve. \
![](./images/17.png)

* Agregar impresora (NO es local)
* Crear nueva impresora usando el URL anterior. \
![](./images/19.png)

  ![](./images/20.png)

  ![](./images/21.png)


## 3.3 Comprobar desde el navegador

> * Comprobaremos que la versión Free tiene limitaciones. \
  ![](./images/26.png)


Vamos a realizar seguidamente una prueba sencilla en tu impresora de red:
* Accede a la configuración de la impresora a través del navegador.
    * Ponemos en `pausa` los trabajos de impresión de la impresora. \
    ![](./images/22.png)
* Vamos a la MV cliente.
* Probaremos la impresora remota imprimiendo el documento `imprimir14w-web`. \
    * Comprobar que al estar la impresora en pausa, el trabajo aparece en cola de impresión. \
    ![](./images/23.png)
* Finalmente pulsa en reanudar el trabajo para que tu documento se convierta a PDF. \
    ![](./images/24.png)
* Si tenemos problemas para que aparezca el PDF en el servidor, iniciar el
programa PDFCreator y esperar un poco. \
    ![](./images/25.png)

---
