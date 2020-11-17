# Servicio de Directorio con comandos

## 1. Nombre de equipo FQDN

* Vamos a usar una MV OpenSUSE para montar nuestro servidor LDAP.
* Nuestra máquina debe tener un FQDN=`server14.curso1920`.
    * Revisaremos `/etc/hostname`
    * Revisaremos `/etc/hosts`

```
127.0.0.2   server14g.curso2021   server14g
```

* Comprobaremos la salida de: `hostname -a`, `hostname -d` y `hostname -f`.

---

# 2. Instalar el Servidor LDAP

## 2.1 Instalación del paquete

* Abriremos una consola como root.
* Con el comando `zypper in 389-ds`, vamos a instalar el script de instalación.
* Ahora debemos tener un script en `/usr/sbin/setup-ds.pl`.
* `/usr/sbin/setup-ds.pl`, para ejecutar el script de instalación.
A continuación iremos respondiendo a las preguntas de configuración del servicio (Cambiar XX por identificador de cada alumno):

```
==================================================
* Choose a setup type  => 2. Typical
* FQDN of the computer => server14.curso1920
* System User          => dirsrv
* System Group         => dirsrv
* Network port number  => 389
* DS identifier        => ldap13
* Suffix (valid DN)    => dc=ldap14,dc=curso1920
* Administrative user  => cn=Directory Manager
==================================================
```

Nos aparece este "Warning2 de SELinux, pero la instancia LDAP se ha creado:

```
ImportError: No module named selinux
Your new DS instance 'ldap42' was successfully created.
Exiting . . .
Log file is '/tmp/setupuofQkd.log'
```

> **IMPORTANTE**:
> * Cada vez que aparece ldapXX, hay que cambiar XX por el identificador de cada alumno.
> * Recordar el nombre y clave de nuestro usuario administrador del servidor de directorios LDAP.
> * Los ficheros de configuración de nuestro servicio/instancia los tenemos en `/etc/dirsrv/slapd-ldapXX`
> * El fichero de configuración `/etc/dirsrv/slapd-ldapXX/dse.ldif` contiene los parámetros principales del servicio de directorio. Como el DN de la Base, del usuario administrador, clave, etc.

## 2.2 Comprobamos el servicio

* `systemctl status dirsrv@ldap14`, comprobar si el servicio está en ejecución.

> Más ayuda:
> * `ps -ef |grep ldap`, para comprobar si el demonio está en ejecución.
> * `systemctl enable dirsrv@ldapXX`, activar al inicio.
> * `systemctl start dirsrv@ldapXX`, iniciar el servicio.

* `nmap -Pn serverXX | grep -P '389|636'`, para comprobar que el servidor LDAP es accesible desde la red. En caso contrario, comprobar cortafuegos.

> **Cortafuegos**: Abrir los puertos LDAP en el cortafuegos
>
> * `systemctl status firewalld`, comprobar el estado del cortafuegos. Debe estar en ejecución.
> * `firewall-cmd --permanent --add-port={389/tcp,636/tcp,9830/tcp}
`, abre determinados puertos en el cortafuegos usando la herramienta  "firewall-cmd"
> * `firewall-cmd --reload`, recargar la configuración del cortafuegos para asegurarnos de que se han leído los nuevos cambios.
>
> Recordatorio:
> * `systemctl enable firewalld`, activar contafuegos en el inicio del sistema.
> * `systemctl start firewalld`, iniciar el cortafuegos.  

## 2.3 Comprobamos el acceso al contenido del LDAP

* `ldapsearch -b "dc=ldap14,dc=curso1920" -x | grep dn`, muestra el contenido de nuestra base de datos LDAP.
* Comprobar que existen las OU Groups y People.
* `ldapsearch -H ldap://localhost -b "dc=ldap14,dc=curso1920" -W -D "cn=Directory Manager" | grep dn`, en este caso hacemos la consulta usando usuario/clave.

| Parámetro                   | Descripción                |
| --------------------------- | -------------------------- |
| -x                          | No se valida usuario/clave |
| -b "dc=ldap42,dc=curso1920" | Base/sufijo del contenido  |
| -H ldap://localhost:389     | IP:puerto del servidor     |
| -W                          | Se solicita contraseña     |
| -D "cn=Directory Manager"   | Usuario del LDAP           |

---
# 3. Añadir usuarios LDAP por comandos
## 3.1 Buscar Unidades Organizativas

Deberían estar creadas las OU People y Groups, es caso contrario hay que crearlas (Consultar ANEXO). Ejemplo para buscar las OU:

```
ldapsearch -H ldap://localhost:389
           -W -D "cn=Directory Manager"
           -b "dc=ldap14,dc=curso1920" "(ou=*) | grep dn"
```

> * Importante: No olvidar especificar la base (-b). De lo contrario probablemente no haya resultados en la búsqueda.
> * `"(ou=*)"` es un filtro de búsqueda de todas las unidades organizativas.
> * `"(uid=*)"` es un filtro de búsqueda de todos los usuarios.

## 3.2 Agregar usuarios

> Enlaces de interés:
> * VÍDEO Teoría [Los ficheros LDIF](http://www.youtube.com/watch?v=ccFT94M-c4Y)

Uno de los usos más frecuentes para el directorio LDAP es para la administración de usuarios. Vamos a utilizar ficheros **ldif** para agregar usuarios.

* Fichero `mazinger-add.ldif` con la información para crear el usuario `mazinger` (Cambiar el valor de dn por el nuestro):

```
dn: uid=mazinger,ou=People,dc=ldapXX,dc=curso1920
uid: mazinger
cn: Mazinger Z
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: {CLEARTEXT}escribir la clave secreta
shadowLastChange: 14001
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 2001
gidNumber: 100
homeDirectory: /home/mazinger
gecos: Mazinger Z
```

* `ldapadd -x -W -D "cn=Directory Manager" -f mazinger-add.ldif
`, escribir los datos del fichero **ldif** anterior en LDAP.

## 3.3 Comprobar el nuevo usuario

* `ldapsearch -W -D "cn=Directory Manager" -b "dc=ldapXX,dc=curso1920" "(uid=*)"`, para comprobar si se ha creado el usuario en el LDAP.

Estamos usando la clase `posixAccount`, para almacenar usuarios dentro de un directorio LDAP. Dicha clase posee el atributo `uid`.
Por tanto, para listar los usuarios de un directorio, podemos filtrar por `"(uid=*)"`.

> **Eliminar usuario del árbol del directorio**
>
> * Crear un archivo `mazinger-delete.ldif`:
>
> ```
> dn: uid=mazinger,ou=People,dc=ldapXX,dc=curso1920
> changetype: delete
> ```
>
> * Ejecutamos el siguiente comando para eliminar un usuario del árbol LDAP: `ldapmodify -x -D "cn=Directory Manager" -W -f mazinger-delete.ldif`

---
# 4. Contraseñas encriptadas

En el ejemplo anterior la clave se puso en texto plano. Cualquiera puede leerlo y no es seguro. Vamos generar valores de password encriptados.

## 4.1 TEORIA: Herramienta slappasswd

> Enlace de interés:
> * [Configurar password LDAP en MD5 o SHA-1](https://www.linuxito.com/seguridad/991-como-configurar-el-password-de-root-de-ldap-en-md5-o-sha-1)
> * [UNIX/GNU/Linux md5sum Command Examples](https://linux.101hacks.com/unix/md5sum/)´
> * [Cómo configurar el password de root de LDAP en MD5 o SHA-1](https://www.linuxito.com/seguridad/991-como-configurar-el-password-de-root-de-ldap-en-md5-o-sha-1)

* Ejecutar `zypper in openldap2`, para instalar la heramienta `slappasswd` en OpenSUSE.

La herramienta `slappasswd` provee la funcionalidad para generar un valor userPassword adecuado. Con la opción -h es posible elegir uno de los siguientes esquemas para almacenar la contraseña:
1. {CLEARTEXT} (texto plano),
1. {CRYPT} (crypt),
1. {MD5} (md5sum),
1. {SMD5} (MD5 con salt),
1. {SHA} (1ssl sha) y
1. {SSHA} (SHA-1 con salt, esquema por defecto).

**Ejemplo SHA-1**

Para generar un valor de contraseña hasheada utilizando SHA-1 con salt compatible con el formato requerido para un valor userPassword, ejecutar el siguiente comando:

```bash
$ slappasswd -h {SSHA}
New password:
Re-enter new password:
{SSHA}5uUxSgD1ssGkEUmQTBEtcqm+I1Aqsp37
```

**Ejemplo MD5**

También podemos usar el comando `md5sum` para crear claves md5. Ejemplo:

```bash
$ md5sum
clave secreta
43cff9e9a30167a1e383026bf61108f2  -
```

## 4.2 Agregar más usuarios

> Enlace de interés:
> * [Cómo cifra GNU/Linux las contraseñas](http://www.nexolinux.com/como-cifra-linux-las-contrasenas/)

Identificar el sistema de encriptación de contraseñas utilizado por GNU/Linux.
* Consultando nuestro fichero `/etc/shadow` podemos ver que las contraseñas tienen el esquema `$6$aaa$bbbb`.
* Por tanto, se deduce que:
    * $6$ => estamos usando SHA-512 (86 Caracteres) para encriptar.
    * aaa => salt bit
    * bbb => clave encriptada.

Agregar más usuarios:    
* Ir a la MV servidor LDAP.
* Crear los siguientes usuarios en LDAP con clave encriptada:

| Full name       | Login acount | uid  |
| --------------- | ------------ | ---- |
| Koji Kabuto     | koji         | 2002 |
| Boss            | boss         | 2003 |
| Doctor Infierno | drinfierno   | 2004 |

## 4.3 Comprobar los usuarios creados

* Ir a la MV cliente LDAP.
* Ejecutar comando `ldpasearch ... "(uid=*)" | grep dn` para consultar los usuarios LDAP en el servidor de directorios remoto.

---
