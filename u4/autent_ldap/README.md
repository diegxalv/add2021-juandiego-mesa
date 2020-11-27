# 1. Preparativos

* Supondremos que tenemos una MV1 (server14g) con DS-389 instalado, y con varios usuarios dentro del DS.
* Necesitamos una segunda máquina virtual (MV2) con S.O. OpenSUSE.

Comprobamos el acceso al LDAP desde el cliente:
* `nmap -Pn 172.19.14.31 | grep -P '389|636'`, para comprobar que el servidor LDAP es accesible desde el cliente.
* `ldapsearch -H ldap://172.19.14.31:389 -W -D "cn=Directory Manager" -b "dc=ldap14,dc=curso2021" "(uid=*)" | grep dn`, comprobamos que los usuarios del LDAP remoto son visibles en el cliente.

# 2. Configurar autenticación LDAP

## 2.1 Crear conexión con servidor

Vamos a configurar de la conexión del cliente con el servidor LDAP.

* Ir a la MV cliente.
* No aseguramos de tener bien el nombre del equipo y nombre de dominio (`/etc/hostname`, `/etc/hosts`)
* Ir a `Yast -> Cliente LDAP y Kerberos`.
* Configurar como la imagen de ejemplo:

![opensuse422-ldap-client-conf.png](./images/opensuse422-ldap-client-conf.png)

* DN de usuario: `cn=Directory Manager,dc=ldap14,dc=curso2021`
* Contraseña: CLAVE del usuario cn=Directory Manager
* Al final usar la opción de `Probar conexión`

## 2.2 Comprobar con comandos

* Vamos a la consola con usuario root, y probamos lo siguiente:
```
id mazinger
su -l mazinger   # Entramos con el usuario definido en LDAP

getent passwd mazinger          # Comprobamos los datos del usuario
cat /etc/passwd | grep mazinger # El usuario NO es local
```

# 3. Crear usuarios y grupos dentro del LDAP

En este punto vamos a escribir información dentro del servidor de directorios LDAP.
Este proceso se debe poder realizar tanto desde el Yast del servidor, como desde el Yast
del cliente.

* Ir a la MV cliente.
* `Yast -> Usuarios Grupos`.
* Set filter: `LDAP users`.
* Bind DN: `cn=Directory Manager,dc=ldapXX,dc=curso2021`.
* Crear el grupo `villanos` (Estos se crearán dentro de la `ou=groups`).
* Crear los usuarios `robot`, `baron` (Estos se crearán dentro de la `ou=people`).
* Consultar/comprobar el contenido de la base de datos LDAP.
    * `ldapsearch -H ldap://IP-LDAP-SERVER -W -D "cn=Directory Manager" -b "dc=ldapXX,dc=curso2021" "(uid=NOMBRE-DEL-UUARIO)"` comando para consultar en la base de datos LDAP la información del usuario con uid concreto.

# 4. Autenticación

Con autenticacion LDAP prentendemos usar la máquina servidor LDAP, como repositorio centralizado de la información de grupos, usuarios, claves, etc. Desde otras máquinas conseguiremos autenticarnos (entrar al sistema) con los usuarios definidos no en la máquina local, sino en la máquina remota con LDAP. Una especie de *Domain Controller*.

## 4.1 Comprobamos autenticación desde el cliente con comandos

* Vamos a la consola con nuestro usuario normal, y probamos lo siguiente:
```
id robot
su -l robot   # Entramos con el usuario definido en LDAP

getent group villanos           # Comprobamos los datos del grupo
cat /etc/group | grep villanos  # El grupo NO es local

getent passwd baron             # Comprobamos los datos del usuario
cat /etc/passwd | grep baron    # El usuario NO es local
```

## 4.2 Comprobar autenticación desde el cliente por el entorno gráfico

* Ir a la MV cliente.
* Iniciar sesión gráfica con algún usuario LDAP.
* Iniciar sesión con usuario local.
* Abrir una consola y hacer lo siguiente:

```
id robot
su -l robot   # Entramos con el usuario definido en LDAP
```

---
# ANEXO

Enlaces de interés:
* Cliente LDAP: [PAM, NIS, LDAP, Kerberos, DS y Samba 4 AD-CD - Redes PYMES](http://blog.desdelinux.net/pam-nis-ldap-kerberos-ds-samba-4-ad-dc-redes-pymes/#Cliente_LDAP)

```
* Permitir autenticación      : SI
* Almacenar entradas en caché : SI
* Crear directorio personal   : SI
* Leer -> Usuarios            : SI
* Leer -> Grupos              : NO
* Leer -> Sudo                : NO
* Leer -> Discos              : NO
* Ubicaciones de servidores   : IP-LDAP-SERVERXX:389
* DN de la base               : dc=ldapXX,dc=curso202
* DN usuario                  : (Vacío)
* Contraseña usuario          : (Vacío)
* Miembros de grupo por DN    : SI
* Dejar conexiones abiertas   : SI
* Comunicación LDAP segura    : No usar seguridad
```

`ldapsearch -x -L -u -t "(uid=nombre-del-usuario)"`,
