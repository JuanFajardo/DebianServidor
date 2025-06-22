# 2. Configuración

> El poder no es nada sin control.
> 
> __Pirelli__

El servidor recién instalado no tendrá ninguna funcionalidad a menos que sea configurado correctamente. El siguiente paso, por lo tanto, es la configuración del servidor.

- __[2.0 Antes de Comenzar](#20-antes-de-comenzar)__

- __[2.1 Red Local](#21-red-local)__
     - __[2.1.1 Diagrama de Red](#211-diagrama-de-red)__
     - __[2.1.2 Dirección IP Estática](#212-direccion-ip-estatica)__
     - __[2.1.3 Nombre del Sistema](#213-nombre-del-sistema)__
     - __[2.1.4 Agregación de Interfaces de Red](#214-agregacion-de-interfaces-de-red)__
     - __[2.1.5 Interfaz de Red Virtual](#215-interfaz-de-red-virtual)__

- __[2.2 Software](#22-software)__
     - __[2.2.1 Gestor de Paquetes APT](#221-gestor-de-paquetes-apt)__
     - __[2.2.2 Actualizaciones de Software](#222-actualizaciones-de-software)__
     - __[2.2.3 Actualización de Procesos en Curso](#223-actualizacion-de-procesos-en-curso)__
     - __[2.2.4 Repositorios](#224-repositorios)__
     - __[2.2.5 Repositorio Backports](#225-repositorio-backports)__
     - __[2.2.6 Notificación de Actualizaciones](#226-notificacion-de-actualizaciones)__
     - __[2.2.7 Actualizaciones Automáticas](#227-actualizaciones-automaticas)__

- __[2.3 Acceso Remoto](#23-acceso-remoto)__
     - __[2.3.1 El Protocolo SSH](#231-el-protocolo-ssh)__
     - __[2.3.2 Servidor SSH](#232-servidor-ssh)__
     - __[2.3.3 Acceso vía SSH sin Contraseña](#233-acceso-via-ssh-sin-contrasena)__

- __[2.4 Reloj del Sistema](#24-reloj-del-sistema)__
     - __[2.4.1 Fecha, Hora y Zona Horaria](#241-fecha-hora-y-zona-horaria)__
     - __[2.4.2 Protocolo NTP](#242-protocolo-ntp)__
     - __[2.4.3 Servidor NTP](#243-servidor-ntp)__

- __[2.5 Usuarios](#25-usuarios)__
     - __[2.5.2 Cuotas](#252-cuotas)__



## 2.0 Antes de Comenzar

La configuración del servidor consiste, básicamente, en la instalación de paquetes de software y su parametrización específica para nuestro sistema.

Cuando se instala un paquete de software, se genera una configuración muy básica que ofrece una funcionalidad relativamente limitada. Algunos paquetes permanecen inactivos tras la instalación, como medida de seguridad. Para activar y aprovechar todo el potencial de un software, es necesario personalizar y ajustar su configuración.

Sin embargo, dado que un pequeño error de configuración puede inutilizar todo el sistema, es importante tomar algunas precauciones antes de realizar cualquier cambio.

__Copias de Seguridad__

Nunca modifiques un archivo de configuración sin antes hacer una copia de seguridad. En caso de problemas, será posible restaurar el archivo original.

Por ejemplo, si necesitas modificar el archivo de configuración /etc/network/interfaces, primero realiza una copia del original:

```bash
root@server:~# cp /etc/network/interfaces /etc/network/interfaces.ori
```
Luego, edita el archivo:

```bash
root@server:~# nano -w /etc/network/interfaces
```

Puedes verificar las diferencias entre el archivo original y el modificado con el comando diff:

```bash
root@server:~# diff /etc/network/interfaces.ori /etc/network/interfaces
```

El resultado será algo como esto:

```bash
9,10c9,20
< allow-hotplug enp0s3
< iface enp0s3 inet dhcp
---
> # allow-hotplug enp0s3
> # iface enp0s3 inet dhcp
>
> # Static IP address
> auto enp0s3
> iface enp0s3 inet static
>       address 192.168.1.100
>       netmask 255.255.255.0
>       network 192.168.1.0
>       broadcast 192.168.1.255
>       gateway 192.168.1.1
```

Si deseas restaurar el archivo original, realiza primero una copia de seguridad del archivo modificado y luego restaura el original:

```bash
root@server:~# mv /etc/network/interfaces /etc/network/interfaces.bak
root@server:~# cp /etc/network/interfaces.ori /etc/network/interfaces
```

Ahora el archivo /etc/network/interfaces tiene su contenido original.

__Qué Editor Usar__

Los archivos de configuración, en general, son archivos de texto, por lo que pueden modificarse con un editor de texto.

Los puristas suelen preferir vi o vim como su editor de elección (Debian instala por defecto una versión ligera de vim llamada vim.tiny). Una alternativa más amigable es nano.

Cualquiera de estos editores está disponible en la instalación base y puede usarse inmediatamente después de instalar el sistema.

Para editar un archivo con vi o vim.tiny, ejecuta el programa deseado con el archivo como argumento. Por ejemplo, para editar el archivo /etc/network/interfaces:

- Con __vi__:

```bash
root@server:~# vi /etc/network/interfaces
```

- Con __vim.tiny__:

```bash
root@server:~# vim.tiny /etc/network/interfaces
```

- Con __nano__ (usa la opción -w para evitar insertar saltos de línea, que podrían causar problemas en un archivo de configuración):

```bash
root@server:~# nano -w /etc/network/interfaces
```


## 2.1 Red Local

La red local es un elemento esencial para un servidor. Sin embargo, su configuración es extremadamente sencilla.

### 2.1.1 Diagrama de Red

__El diagrama de la red__
Este manual está diseñado pensando en la integración de un servidor en una típica red doméstica cerrada. El acceso a Internet se realiza a través de un router/modem ADSL o de cable, que cuenta con un firewall integrado.

Diagrama de una pequeña red local
![diagrama de red ](img/2_1.png)

__El gateway__
Un gateway es el dispositivo que permite el acceso a Internet desde la red interna o Intranet. Este dispositivo también filtra o bloquea los accesos externos a la red interna, formando así una barrera de seguridad (firewall) entre la red interna y la Internet.

Por lo general, este dispositivo es el router/modem ADSL o de cable que proporciona el acceso a Internet.

El router/firewall necesita un nombre y una dirección IP estática para que sea fácilmente accesible desde los demás dispositivos de la red.

En este ejemplo, el nombre del router/firewall será gateway.home.lan y la dirección IP será 192.168.1.1.

__El servidor__

El servidor Debian, al estar integrado en la red interna, está protegido contra accesos indiscriminados e indeseados desde Internet.

El servidor necesita un nombre y una dirección IP estática para ser fácilmente accesible desde los demás dispositivos de la red. Entre otras funciones, el servidor permitirá, por ejemplo, la asignación automática de direcciones IP a los clientes, así como indicar que el acceso a Internet se realiza a través del gateway.

En este ejemplo, el nombre del servidor será server.home.lan y la dirección IP será 192.168.1.100.

__Los clientes__

Los clientes serán todos los dispositivos conectados a la red interna. Su configuración dentro de la red será gestionada por los distintos servicios que ofrezca el servidor. El acceso de los clientes al exterior se realizará exclusivamente a través del gateway, manteniéndolos en un entorno seguro y protegidos de posibles accesos malintencionados externos.


### 2.1.2 Direccion IP Estatica

La instalación predeterminada de Debian configura la red para obtener una dirección dinámica a través de DHCP. Sin embargo, para que el sistema funcione como servidor, debe tener una dirección IP estática.

El objetivo es configurar la interfaz de red eno1 con la dirección IP estática 192.168.1.100. Al mismo tiempo, se indicará la dirección del dispositivo de acceso a Internet o “gateway” (192.168.1.1). En una configuración doméstica, esta será la dirección estática del router ADSL o de cable.

> Nota: Los nombres de las interfaces de red varían según el tipo de adaptador instalado. Este documento se refiere a la primera interfaz de red como eno1. Sin embargo, el nombre debe ser reemplazado por la designación real de su interfaz de red.
> Para más información, consulte:
>
> [Nuevo método para nombres de interfaces de red](https://wiki.debian.org/NetworkInterfaceNames)
>
> [Nombres predecibles de interfaces de red.](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)

__Configuración__
Antes de cambiar la configuración, verifiquemos la configuración actual:

```bash
root@server:~# ip -4 a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.1.123/24 brd 192.168.1.255 scope global eno1
       valid_lft forever preferred_lft forever
```

Aquí podemos observar que la interfaz de red tiene la designación eno1 y que la dirección asignada dinámicamente es 192.168.1.123.

La configuración de las interfaces de red se guarda en el archivo /etc/network/interfaces:

```bash
cat /etc/network/interfaces
# Este archivo describe las interfaces de red disponibles en el sistema
# y cómo activarlas. Para más información, vea interfaces(5).

# La interfaz de red loopback
auto lo
iface lo inet loopback

# La interfaz de red primaria
# allow-hotplug eno1
# iface eno1 inet dhcp

# Dirección IP estática
auto eno1
iface eno1 inet static
        address 192.168.1.100
        netmask 255.255.255.0
        network 192.168.1.0
        broadcast 192.168.1.255
        gateway 192.168.1.1
```

También es necesario especificar la dirección del servidor DNS. En esta configuración, el servidor DNS opera en el módem/router ADSL o de cable, por lo que el parámetro nameserver debe tener el valor 192.168.1.1, ademas de poder poner una direccion de DNS alternativas como la de Cloudfare y Google en el archivo /etc/resolv.conf:

```bash
cat /etc/resolv.conf
nameserver 1.1.1.1
nameserver 8.8.8.8
nameserver 192.168.1.1
```

Reiniciar la configuración de red
Para activar la nueva configuración, reinicie la interfaz de red:

```bash
root@server:~# ifup eno1
```

En algunos casos, puede ser necesario reiniciar el sistema para que la configuración de red surta efecto:

```bash
root@server:~# reboot
```

__Verificación__
El comando ifconfig proporciona información detallada sobre la configuración de las interfaces de red. La configuración de la interfaz eno1 ahora debería mostrar los parámetros previamente definidos:

```bash
root@server:~# ip -4 a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eno1
       valid_lft forever preferred_lft forever
```

También debería ser posible contactar servidores en Internet:

```bash
root@server:~# ping -c3 www.debian.org
PING www.debian.org (5.153.231.4) 56(84) bytes of data.
64 bytes from senfter.debian.org (5.153.231.4): icmp_seq=1 ttl=63 time=56.3 ms
64 bytes from senfter.debian.org (5.153.231.4): icmp_seq=2 ttl=63 time=57.3 ms
64 bytes from senfter.debian.org (5.153.231.4): icmp_seq=3 ttl=63 time=57.2 ms

--- www.debian.org ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2007ms
rtt min/avg/max/mdev = 56.330/56.995/57.380/0.472 ms
```



### 2.1.3 Nombre del Sistema

Después de la instalación, el nombre del sistema puede ser modificado.

__Configuración__
El nombre del sistema, o hostname, se guarda en el archivo /etc/hostname. Este archivo debe contener únicamente el nombre del sistema, y no el nombre completo del dominio:

```bash
cat /etc/hostname
server
```

El nuevo nombre debe ser asignado al sistema a partir del archivo recién creado:

```bash
root@server:~# hostname -F /etc/hostname
```

Finalmente, el nombre del servidor debe asociarse a un nombre completo de dominio y a una dirección IP en el archivo /etc/hosts:

```bash 
cat /etc/hosts

127.0.0.1       localhost  
192.168.1.100   server.home.lan server  

# Las siguientes líneas son recomendadas para hosts compatibles con IPv6  
::1     localhost ip6-localhost ip6-loopback  
ff02::1 ip6-allnodes  
ff02::2 ip6-allrouters  
```

__Verificación__

Ejecutar los siguientes comandos para verificar la configuración:

```bash
root@server:~# hostname --short  
server  

root@server:~# hostname --domain  
home.lan  

root@server:~# hostname --fqdn  
server.home.lan  

root@server:~# hostname --ip-address  
192.168.1.100  
```

### 2.1.4 Agregacion de Interfaces de Red

La mayoría de los sistemas actuales cuentan con 2 (o más) conexiones Ethernet. Estas pueden ser utilizadas de forma independiente o en paralelo mediante una técnica llamada bonding. Esta técnica es muy útil, ya que permite balanceo de carga (los datos se transmiten por ambas interfaces) y tolerancia a fallos (si una conexión falla, la transmisión es gestionada por la otra).

En nuestro servidor, agregaremos las dos interfaces físicas del sistema, eno1 y eno2, para crear una nueva interfaz de red bond0 de alta disponibilidad.

> __Ethernet bonding__
> El Ethernet bonding, regulado por la norma IEEE 802.3ad bajo el título link aggregation, es un término en redes de computadoras que describe el acoplamiento de dos o más canales Ethernet en paralelo para formar un único canal con mayor velocidad y/o aumentar la disponibilidad y redundancia del mismo.

__Instalación__

```bash
root@server~# apt install ifenslave
```

__Configuración__

Para crear la interfaz bond0, se debe cargar y configurar el módulo bonding del kernel. Este módulo se cargará automáticamente después de la configuración; por ahora, debe cargarse manualmente:

```bash
root@server:~# modprobe bonding
```

Verificar que se ha cargado correctamente:

```bash
root@server:~# lsmod | grep bonding
bonding               147456  0
```

El siguiente paso es parametrizar la nueva interfaz de red y eliminar (o comentar) cualquier referencia a las interfaces físicas usadas por ella. Esto se realiza en el archivo /etc/network/interfaces:

```bash 
cat /etc/network/interfaces

plaintext
Copiar
Editar
# Este archivo describe las interfaces de red disponibles en el sistema
# y cómo activarlas. Para más información, consulte interfaces(5).

# La interfaz de red de bucle local
auto lo
iface lo inet loopback

# Configuración estática para bonding
auto bond0
iface bond0 inet static
        slaves eno1 eno2
        bond-mode balance-rr
        bond-miimon 100
        bond_downdelay 200
        bond_updelay 200

        address 192.168.1.100
        netmask 255.255.255.0
        network 192.168.1.0
        broadcast 192.168.1.255
        gateway 192.168.1.1
```

Así se crea la interfaz bond0, compuesta por la agregación de las interfaces "esclavas" eno1 y eno2. El parámetro bond-mode balance-rr indica que la interfaz operará en modo balanceo round-robin, transmitiendo datos alternadamente por las distintas interfaces físicas que componen la nueva interfaz bond0.

Finalmente, reiniciar los servicios de red:

```bash
root@server:~# systemctl restart networking
```

__Verificación__
El comando ifconfig permite verificar el estado de las interfaces de red:

```bash
root@server:~# ifconfig
bond0     Link encap:Ethernet  HWaddr 08:00:27:69:7a:b5
          inet addr:192.168.56.100  Bcast:192.168.56.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MASTER MULTICAST  MTU:1500  Metric:1
          RX packets:47 errors:0 dropped:0 overruns:0 frame:0
          TX packets:62 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:5803 (5.6 KiB)  TX bytes:8608 (8.4 KiB)
 
eno1      Link encap:Ethernet  HWaddr 08:00:27:69:7a:b5
          UP BROADCAST RUNNING SLAVE MULTICAST  MTU:1500  Metric:1
          RX packets:44 errors:0 dropped:0 overruns:0 frame:0
          TX packets:31 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:5601 (5.4 KiB)  TX bytes:2842 (2.7 KiB)
 
eno2      Link encap:Ethernet  HWaddr 08:00:27:69:7a:b5
          UP BROADCAST RUNNING SLAVE MULTICAST  MTU:1500  Metric:1
          RX packets:3 errors:0 dropped:0 overruns:0 frame:0
          TX packets:31 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:202 (202.0 B)  TX bytes:5766 (5.6 KiB)
```

La interfaz bond0 tiene asignada una dirección IP y las 3 interfaces (bond0, eno1 y eno2) comparten la misma dirección física (HWaddr 08:00:27:69:7a:b5), para que la red las identifique como una sola.

Más información sobre el estado de bond0 y sus componentes puede obtenerse con:

```bash
root@server:~# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)
 
Bonding Mode: load balancing (round-robin)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 200
Down Delay (ms): 200
 
Slave Interface: eno1
MII Status: up
Speed: Unknown
Duplex: Unknown
Link Failure Count: 0
Permanent HW addr: 08:00:27:69:7a:b5
Slave queue ID: 0
 
Slave Interface: eno2
MII Status: up
Speed: Unknown
Duplex: Unknown
Link Failure Count: 0
Permanent HW addr: 08:00:27:22:af:bf
Slave queue ID: 0
```


### 2.1.5 Interfaz de Red Virtual

En ciertos casos, es ventajoso asignar más de una dirección IP a un sistema. Si el sistema tiene varias interfaces de red, basta con asignar direcciones diferentes a cada una. En caso de que solo haya una conexión de red, es posible crear interfaces virtuales. Así, a partir de una interfaz eno1, se pueden crear una o más interfaces virtuales, como eno1:0, eno1:1, etc.

__Configuración__

La configuración de una interfaz virtual se realiza en el archivo /etc/network/interfaces:

```bash
cat /etc/network/interfaces

# Este archivo describe las interfaces de red disponibles en el sistema
# y cómo activarlas. Para más información, consulte interfaces(5).

# Interfaz de red loopback
auto lo
iface lo inet loopback

# Interfaz de red principal
# allow-hotplug eno1
# iface eno1 inet dhcp

# Dirección IP estática
auto eno1
iface eno1 inet static
        address 192.168.1.100
        netmask 255.255.255.0
        network 192.168.1.0
        broadcast 192.168.1.255
        gateway 192.168.1.1

# Interfaz virtual
# Dirección IP estática
auto eno1:0
iface eno1:0 inet static
        address 192.168.1.101
        netmask 255.255.255.0
```

En la configuración de la interfaz eno1:0, solo se definen los parámetros address y netmask, ya que los demás parámetros son iguales a los de la interfaz eno1.

Gracias a la línea auto eno1:0 en el archivo /etc/network/interfaces, la interfaz virtual se activará automáticamente en cada reinicio del sistema. Por ahora, actívela manualmente ejecutando:

```bash
root@server:~# ifup eno1:0
```

Asigne un nombre al nuevo sistema o dirección IP en el archivo /etc/hosts:

```bash
cat /etc/hosts

127.0.0.1       localhost
192.168.1.100   server.home.lan server
192.168.1.101   virtual.home.lan virtual

# Las siguientes líneas son recomendadas para hosts con capacidad IPv6
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

__Verificación__

El comando ifconfig mostrará la interfaz virtual eno1:0 activada, junto con la dirección IP y demás parámetros asignados:

```bash
root@server:~# ifconfig
eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::d6ae:52ff:fec5:4e01  prefixlen 64  scopeid 0x20<link>
        ether d4:ae:52:c5:4e:01  txqueuelen 1000  (Ethernet)
        RX packets 218167200  bytes 191175932825 (178.0 GiB)
        RX errors 0  dropped 1704  overruns 0  frame 0
        TX packets 226986879  bytes 225966867470 (210.4 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 16

eno1:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.101  netmask 255.255.255.0  broadcast 192.168.1.255
        ether d4:ae:52:c5:4e:01  txqueuelen 1000  (Ethernet)
        device interrupt 16

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 186225885  bytes 154799856220 (144.1 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 186225885  bytes 154799856220 (144.1 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

El comando ping permite verificar si un servidor está accesible y responde a las comunicaciones de red:

```bash
root@server:~# ping -c3 virtual
PING virtual.home.lan (192.168.1.101) 56(84) bytes of data.
64 bytes from 192.168.1.101 (192.168.1.101): icmp_seq=1 ttl=64 time=0.048 ms
64 bytes from 192.168.1.101 (192.168.1.101): icmp_seq=2 ttl=64 time=0.045 ms
64 bytes from 192.168.1.101 (192.168.1.101): icmp_seq=3 ttl=64 time=0.040 ms

--- virtual.home.lan ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2040ms
rtt min/avg/max/mdev = 0.040/0.044/0.048/0.006 ms
```


## 2.2 Software

La distribución Debian ofrece varias decenas de miles de paquetes de software, listos para usar en nuestro sistema.

### 2.2.1 Gestor de Paquetes APT

Uno de los puntos fuertes de la distribución Debian es su gestor de paquetes APT y su interfaz Aptitude. La gestión de actualizaciones e instalaciones de software se realiza con la ayuda de este poderoso y amigable gestor de paquetes. Gracias a APT y Aptitude, es posible, por ejemplo, actualizar todo un sistema con solo un par de comandos.

__¿APT, apt-get o Aptitude?__

Existe un debate sobre si se debe usar APT o Aptitude como gestor de paquetes. Sin embargo, en esta guía se dará preferencia al uso de APT.

El gestor Aptitude, aunque es un front-end de APT, tiene algunas ventajas, como su interfaz gráfica y la capacidad de mantener un registro (log) de las acciones realizadas, lo que permite una eliminación más "limpia" de paquetes.

__Instalación de Aptitude__

El paquete APT está incluido en la instalación de Debian. El paquete Aptitude puede instalarse con un simple comando:

```bash
root@server:~# apt install aptitude
```

__Guía rápida de APT__

APT es un gestor de paquetes basado en línea de comandos que proporciona comandos para buscar, instalar y desinstalar paquetes de software, además de permitir consultar información sobre ellos. Ofrece las mismas funcionalidades que las herramientas APT especializadas, como apt-get o apt-cache, pero añade opciones más interesantes para un uso interactivo.

A continuación, se presentan ejemplos y opciones de APT, junto con los comandos equivalentes en apt-get, apt-cache y Aptitude:

__Actualizar la lista de paquetes__

```bash
apt update
```
- Actualiza la lista de paquetes y metadatos existentes en los repositorios. Es el primer comando que se debe ejecutar al gestionar paquetes.
Equivalente a: apt-get update o aptitude update.

__Instalar paquetes__

```bash
apt install <paquete>
```
- Instala un paquete de software y todas sus dependencias. Se pueden instalar varios paquetes a la vez:

```bash
apt install <paquete1> <paquete2> ...
```
- Equivalente a: apt-get install <paquete> o aptitude install <paquete>.

__Reinstalar un paquete__

```bash
apt install --reinstall <paquete>
```
- Reinstala un paquete, reemplazando sus archivos. Útil para restaurar archivos modificados.
- Equivalente a: apt-get install --reinstall <paquete> o aptitude reinstall <paquete>.

__Actualizar el sistema__

```bash
apt upgrade
```
- Instala todas las actualizaciones disponibles, incluyendo nuevas dependencias.
Equivalente a: apt-get upgrade o aptitude safe-upgrade.

```bash
apt full-upgrade
```
- Instala todas las actualizaciones disponibles y elimina o instala paquetes según las nuevas dependencias.
- Equivalente a: apt-get dist-upgrade o aptitude full-upgrade.

__Eliminar paquetes__

```bash
apt remove <paquete>
```
- Elimina un paquete. Se pueden eliminar varios a la vez:

```bash
apt remove <paquete1> <paquete2> ...
```
- Equivalente a: apt-get remove <paquete> o aptitude remove <paquete>.
- Eliminar completamente un paquete

```bash
apt purge <paquete>
```
- Elimina un paquete y sus archivos de configuración.
Equivalente a: apt-get purge <paquete> o aptitude purge <paquete>.
- Eliminar paquetes innecesarios

```bash
apt autoremove
```
- Elimina paquetes instalados automáticamente como dependencias que ya no son necesarios.
- Equivalente a: apt-get autoremove.
- Eliminar con configuración

```bash
apt autoremove --purge
```
- Elimina dependencias no necesarias y sus archivos de configuración.


__Buscar paquetes__

```bash
apt search <criterio>
```
- Busca paquetes en la lista y muestra resultados que coincidan con el criterio.
- Equivalente a: apt-cache search <criterio> o aptitude search <criterio>.

__Mostrar información sobre un paquete__

```bash
apt show <paquete>
```
- Muestra detalles del paquete.
- Equivalente a: apt-cache show <paquete> o aptitude show <paquete>.

__Limpiar el repositorio local__

```bash
apt clean
```
- Elimina archivos de paquetes del repositorio local.
- Equivalente a: apt-get clean o aptitude clean.

__Actualizar con frecuencia para mayor seguridad__

Para garantizar la seguridad del sistema, es imprescindible mantener el servidor lo más actualizado posible. La comunidad de Debian actualiza constantemente el software para corregir errores y vulnerabilidades de seguridad. Un sistema desactualizado es una invitación para hackers y atacantes.

Es importante elegir cuidadosamente las fuentes de software o repositorios y realizar actualizaciones frecuentes. Además, se debe estar al tanto de las listas de anuncios de seguridad y correcciones publicadas en el sitio web de Debian o en listas de distribución oficiales.


### 2.2.2 Actualizaciones de Software

Para garantizar que el sistema tenga las versiones de software y correcciones de seguridad más recientes, las actualizaciones deben realizarse regularmente. La actualización del software se divide en dos partes: la actualización de la lista de software disponible en los repositorios y la instalación de las nuevas versiones disponibles. Ambas operaciones se realizan utilizando el comando apt con diferentes opciones.

> __Atención__
>
> Es imperativo realizar actualizaciones con frecuencia para asegurar que el sistema siempre disponga de las correcciones de errores y actualizaciones de seguridad más recientes.

__Actualización de los repositorios__

La actualización de la lista de software disponible en los repositorios es muy sencilla:

```bash
root@server:~# apt update
Hit:1 http://security.debian.org/debian-security buster/updates InRelease
Hit:2 http://deb.debian.org/debian buster InRelease
Hit:3 http://deb.debian.org/debian buster-updates InRelease
Reading package lists... Done
Building dependency tree
Reading state information... Done
3 packages can be upgraded. Run 'apt list --upgradable' to see them.
Actualmente, hay 3 actualizaciones disponibles para el software instalado.
```

__Listado de los paquetes a actualizar__

Puedes obtener una lista de los paquetes que se pueden actualizar con el siguiente comando:

```bash
root@server:~# apt list --upgradable
Listing... Done
dpkg/testing 1.18.24 amd64 [upgradable from: 1.18.23]
libdpkg-perl/testing 1.18.24 all [upgradable from: 1.18.23]
libtiff5/testing 4.0.7-7 amd64 [upgradable from: 4.0.7-6]
```

__Instalación de las actualizaciones__

La instalación de las actualizaciones es igualmente sencilla:

```bash
root@server:~# apt upgrade
Reading package lists... Done
Building dependency tree
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  dpkg libdpkg-perl libtiff5
3 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 3,622 kB of archives.
After this operation, 93.2 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://deb.debian.org/debian buster/main amd64 dpkg amd64 1.18.24 [2,107 kB]
Get:2 http://deb.debian.org/debian buster/main amd64 libdpkg-perl all 1.18.24 [1,283 kB]
Get:3 http://deb.debian.org/debian buster/main amd64 libtiff5 amd64 4.0.7-7 [232 kB]
Fetched 3,622 kB in 0s (4,170 kB/s)
Reading changelogs... Done
(Reading database ... 87905 files and directories currently installed.)
Preparing to unpack .../dpkg_1.18.24_amd64.deb ...
Unpacking dpkg (1.18.24) over (1.18.23) ...
Setting up dpkg (1.18.24) ...
(Reading database ... 87905 files and directories currently installed.)
Preparing to unpack .../libdpkg-perl_1.18.24_all.deb ...
Unpacking libdpkg-perl (1.18.24) over (1.18.23) ...
Preparing to unpack .../libtiff5_4.0.7-7_amd64.deb ...
Unpacking libtiff5:amd64 (4.0.7-7) over (4.0.7-6) ...
Setting up libdpkg-perl (1.18.24) ...
Setting up libtiff5:amd64 (4.0.7-7) ...
Processing triggers for libc-bin (2.24-10) ...
Processing triggers for man-db (2.7.6.1-2) ...
```

El sistema ahora está actualizado con las versiones más recientes de los paquetes de software.


### 2.2.3 Actualizacion de Procesos en Curso

Como se realizao la ctualizaciones de software con el comando apt upgrade, las versiones actualizadas reemplazan las obsoletas en el disco del sistema.

Sin embargo, las aplicaciones que están cargadas en memoria y en ejecución no se actualizan en este proceso. Como resultado, aplicaciones importantes, como servicios o algunas bibliotecas, continúan funcionando con versiones anteriores y, por lo tanto, con vulnerabilidades no corregidas.

Es absolutamente esencial reiniciar esos servicios para garantizar la seguridad del sistema.

__Instalación__

```bash
root@server:~# apt install debian-goodies
```

__Uso__

El utilitario checkrestart permite listar los procesos que se están ejecutando con versiones obsoletas de los paquetes actualizados.

```bash
root@server:~# checkrestart
Found 1 processes using old versions of upgraded files
(1 distinct programs)
(1 distinct packages)

Of these, 1 seem to contain systemd service definitions or init scripts which can be used to restart them.
The following packages seem to have definitions that could be used
to restart their services:
bind9:
        7746    /usr/sbin/named
These are the initd scripts:
service bind9 restart
```

El resultado nos informa que, aunque los paquetes se hayan actualizado, todavía hay algunos procesos ejecutándose con versiones obsoletas, relacionados con el servicio bind9 de resolución de nombres.

Reinicio de los procesos en ejecución
Para actualizar los procesos en ejecución, simplemente sigue las instrucciones y reinicia el servicio bind9:

```bash
root@server:~# systemctl restart bind9
```

__Verificación__

Una última verificación:

```bash
root@server:~# checkrestart
Found 0 processes using old versions of upgraded files
```

El sistema ahora está realmente actualizado y seguro, ejecutando las versiones más recientes.



### 2.2.4 Repositorios

Además del repositorio principal de software de Debian, conocido como main, configurado durante la instalación del sistema base, existen otros repositorios que no se incluyen inicialmente por diversas razones. Sin embargo, pueden añadirse en cualquier momento.

__Lista de repositorios__

Para facilitar la instalación de ciertos paquetes de software, se recomienda agregar los repositorios contrib y non-free a la lista existente.

Además, como no se compilarán paquetes desde el código fuente, sus referencias (deb-src) pueden ser desactivadas (comentadas).

La lista de repositorios se administra en el archivo de configuración /etc/apt/sources.list:

```bash
cat /etc/apt/sources.list

#
# [...]
deb http://deb.debian.org/debian/ buster main contrib non-free
# deb-src http://deb.debian.org/debian/ buster main contrib non-free

deb http://security.debian.org/debian-security buster/updates main contrib non-free
# deb-src http://security.debian.org/debian-security buster/updates main contrib non-free
```

Actualización de la lista local de paquetes
Después de agregar los nuevos repositorios, es necesario actualizar la lista local de paquetes:

```bash
root@server:~# apt update
Hit:1 http://security.debian.org/debian-security buster/updates InRelease
Hit:2 http://deb.debian.org/debian buster InRelease
Get:3 http://security.debian.org/debian-security buster/updates/non-free amd64 Packages [1,272 B]
Get:4 http://deb.debian.org/debian buster/contrib amd64 Packages [50.9 kB]
Get:5 http://security.debian.org/debian-security buster/updates/non-free Translation-en [481 B]
Get:6 http://deb.debian.org/debian buster/contrib Translation-en [45.9 kB]
Get:7 http://deb.debian.org/debian buster/non-free amd64 Packages [77.9 kB]
Get:8 http://deb.debian.org/debian buster/non-free Translation-en [79.2 kB]
Fetched 256 kB in 0s (456 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
All packages are up to date.
```

__Uso de un proxy__

En algunos casos, puede ser necesario acceder a Internet a través de un proxy. El sistema apt puede configurarse para utilizarlo mediante dos métodos posibles:

- Método 1: Variable de entorno
Definir una variable de entorno http_proxy o ftp_proxy con la URL del servidor proxy. El comando apt usará esta variable al conectarse a Internet:

```bash
root@server:~# export http_proxy="http://proxy.example.com:3128/"
```

- Método 2: Configuración del proxy en apt
Agregar las configuraciones del proxy al archivo /etc/apt/apt.conf.d/99proxy. Si no existe, se puede crear con el siguiente contenido:
```
cat /etc/apt/apt.conf.d/99proxy

Acquire::http::Proxy "http://proxy.home.lan:3128/";
```

Formato de la URL del proxy
El formato para definir un proxy es:

```bash 
http://user:pass@xxx.xxx.xxx.xxx:port/
```

Donde:

- user:pass: Nombre de usuario y contraseña, si el proxy requiere autenticación.
- xxx.xxx.xxx.xxx: Dirección o nombre del servidor proxy.
- port: Puerto de conexión al servicio proxy.



### 2.2.5 Repositorio Backports

Uno de los objetivos de Debian es garantizar su estabilidad, lograda mediante el uso de software ampliamente probado, lo cual implica que no siempre se incluya el software más reciente. Sin embargo, puede surgir la necesidad de instalar software más moderno o no disponible en el momento del lanzamiento de Debian 10 "Buster".

Para estos casos, existe el repositorio backports, que proporciona versiones más recientes o no incluidas en el lanzamiento original de "Buster". Dado que los paquetes y versiones en el repositorio backports se basan en la futura versión de Debian (versión "testing"), no deberían surgir problemas de incompatibilidad al actualizar el sistema.

> __Nota__
> 
> Los paquetes del repositorio backports se compilan a partir de las fuentes de la próxima versión de la distribución Debian (versión actual +1).

__Configuración__

Para habilitar el repositorio backports, se debe añadir su ubicación a la lista de repositorios existente, creando o editando el archivo /etc/apt/sources.list.d/backports.list:

```bash
cat /etc/apt/sources.list.d/backports.list

#
# Debian backports
#

# buster-backports
deb http://deb.debian.org/debian buster-backports main contrib non-free
#deb-src http://deb.debian.org/debian buster-backports main contrib non-free
```

__Verificación__

Una simple actualización de los repositorios debería incluir el repositorio buster-backports:

```bash
root@server:~# apt update
Hit:1 http://ftp.pt.debian.org/debian buster InRelease
Hit:2 http://security.debian.org/debian-security buster/updates InRelease
Get:3 http://ftp.debian.org/debian buster-backports InRelease [46.8 kB]
Fetched 46.8 kB in 0s (121 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
All packages are up to date.
```

__Uso__

Por defecto, los paquetes de los repositorios backports están desactivados. Para instalar un paquete de estos repositorios, es necesario especificar su origen al momento de instalarlo:

```bash
root@server:~# apt -t buster-backports install <paquete>
```

### 2.2.6 Notificacion de Actualizaciones

> __Atención:__
> 
> El paquete apticron depende de la instalación de un agente de transporte de correo electrónico o MTA (Mail Transfer Agent). Por lo tanto, es necesario instalar un servidor SMTP, como Postfix, antes de proceder con la instalación de este paquete.

Para garantizar que el sistema pueda mantenerse actualizado, es fundamental ser notificado sobre las actualizaciones disponibles. Apticron es una utilidad que se ejecuta diariamente de manera automática, verifica si hay actualizaciones disponibles para el sistema y notifica al administrador por correo electrónico.

__Instalación__

```bash
root@server:~# apt install apticron
```

__Configuración__

En el archivo /etc/apticron/apticron.conf, se puede configurar la dirección de correo electrónico a la que se enviarán las notificaciones sobre actualizaciones del sistema:

```bash
cat /etc/apticron/apticron.conf

# apticron.conf
#
# set EMAIL to a space separated list of addresses which will be notified of
# impending updates
#
EMAIL="root"
 
#[...]
```

__Resultado__

Siempre que haya una actualización disponible para el sistema, se enviará un correo electrónico al administrador del sistema con información sobre los paquetes pendientes de actualizar. Ejemplo de correo:

```bash
Subject: 1 Debian package update(s) for server.home.lan
To: <root@server.home.lan>
Date: Thu,  6 Jul 2017 01:52:05 +0100 (WEST)
From: root@server.home.lan (root)
 
apticron report [Thu, 06 Jul 2017 01:52:05 +0100]
========================================================================
 
apticron has detected that some packages need upgrading on:
 
    server.home.lan
    [ 192.168.1.100 ]
 
The following packages are currently pending an upgrade:
 
    libtiff5 4.0.8-2+deb9u1
 
========================================================================
 
Package Details:
 
apt-listchanges: Reading changelogs...
apt-listchanges: Changelogs
---------------------------
 
--- Changes for tiff (libtiff5) ---
tiff (4.0.8-2+deb9u1) buster-security; urgency=high
 
  * Backport security fixes:
    - CVE-2017-9936, memory leak in error code path of JBIGDecode()
      (closes: #866113),
    - prevent out of memory in gtTileContig() on corrupted files,
    - CVE-2017-10688, assertion failure in TIFFWriteDirectoryTagCheckedXXXX()
      (closes: #866611).
  * Add required _TIFFReadEncodedStripAndAllocBuffer@LIBTIFF_4.0 symbol to the
    libtiff5 package.
 
 -- Laszlo Boszormenyi (GCS) <gcs@debian.org>  Sun, 02 Jul 2017 08:36:06 +0000
 
========================================================================
 
You can perform the upgrade by issuing the command:
 
    apt-get dist-upgrade
 
as root on server.home.lan
 
--
apticron
```

El administrador podrá realizar la actualización del sistema usando los comandos:

- ```apt full-upgrade```
o
- ```apt-get dist-upgrade```


### 2.2.7 Actualizaciones Automaticas

> __Atención__
> 
> Para notificar por correo electrónico al administrador del sistema, el paquete unattended-upgrades depende de la instalación de un agente de transporte de correo electrónico o MTA (Mail Transfer Agent). Por lo tanto, es necesaria la instalación de un servidor SMTP, como Postfix, antes de proceder a la instalación de este paquete.

Una preocupación del administrador es mantener siempre el sistema con las actualizaciones y correcciones de software más recientes. El paquete unattended-upgrades permite la instalación de las actualizaciones de los paquetes de software de forma totalmente automática.

__Instalación__

```bash
root@server:~# apt install unattended-upgrades
```

__Configuración__

Por una cuestión de seguridad, la instalación del paquete unattended-upgrades no activa inmediatamente las actualizaciones automáticas. Estas deben ser activadas en el archivo /etc/apt/apt.conf.d/20auto-upgrades. Un ejemplo de este archivo está disponible en la carpeta /usr/share/unattended-upgrades/, y debe copiarse a la carpeta /etc/apt/apt.conf.d/:

```bash
root@server:~# cp /usr/share/unattended-upgrades/20auto-upgrades /etc/apt/apt.conf.d/
```

El archivo de configuración /etc/apt/apt.conf.d/20auto-upgrades deberá activar la actualización de la base de datos de los paquetes disponibles, activar la actualización automática de los paquetes instalados y efectuar una limpieza semanal de los archivos descargados:

```bash
/etc/apt/apt.conf.d/20auto-upgrades
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Download-Upgradeable-Packages "1";
```

En el archivo de configuración /etc/apt/apt.conf.d/50unattended-upgrades podemos configurar la versión de Debian utilizada para las actualizaciones:

```bash 
catg /etc/apt/apt.conf.d/50unattended-upgrades
// Within lines unattended-upgrades allows 2 macros whose values are
// derived from /etc/debian_version:
//   ${distro_id}            Installed origin.
//   ${distro_codename}      Installed codename (eg, "jessie")
Unattended-Upgrade::Origins-Pattern {
    // Codename based matching:
    // This will follow the migration of a release through different
    // archives (e.g. from testing to stable and later oldstable).
    "o=Debian,n=buster";
    "o=Debian,n=buster-updates";
    "o=Debian,n=buster,l=Debian-Security";
};
```

Notificar por correo electrónico al administrador del sistema siempre que se realicen actualizaciones:

```bash
/etc/apt/apt.conf.d/50unattended-upgrades
// Send email to this address for problems or packages upgrades
// If empty or unset then no email is sent, make sure that you
// have a working mail setup on your system. A package that provides
// 'mailx' must be installed. E.g. "user@example.com"
Unattended-Upgrade::Mail "root";
```

Reinicie el servicio para tener en cuenta los cambios:

```bash
root@server:~# systemctl restart unattended-upgrades
```

La lista de paquetes disponible será actualizada diariamente y, en caso de haber actualizaciones disponibles, estas serán instaladas de manera automática.

__Verificación__

Podemos verificar el funcionamiento con el siguiente comando:

```bash
root@server:~# unattended-upgrades --dry-run --debug

Initial blacklisted packages:
Initial whitelisted packages:
Starting unattended upgrades script
Allowed origins are: ['o=Debian,n=buster', 'o=Debian,n=buster-updates', 'o=Debian,n=buster,l=Debian-Security']
pkgs that look like they should be upgraded:
Fetched 0 B in 0s (0 B/s)
fetch.run() result: 0
blacklist: []
whitelist: []
No packages found that can be upgraded unattended and no pending auto-removals
```

__Utilización__

Siempre que se realice una actualización, se enviará un correo electrónico al administrador del sistema:

```bash
Subject: unattended-upgrades result for 'server': 'True'
To: root@home.lan
Date: Thu, 30 Apr 2015 01:27:55 +0100 (WEST)
From: root@home.lan (root)
 
Unattended upgrade returned: True
 
Packages that were upgraded:
 curl libcurl3 libcurl3-gnutls=20
 
Unattended-upgrades log:
Initial blacklisted packages:=20
Initial whitelisted packages:=20
Starting unattended upgrades script
Allowed origins are: ['o=3DDebian,n=3Djessie', 'o=3DDebian,n=3Djessie-propo=
sed-updates', 'o=3DDebian,n=3Djessie,l=3DDebian-Security', 'origin=3DDebian=
,archive=3Djessie,label=3DDebian-Security']
Packages that will be upgraded: curl libcurl3 libcurl3-gnutls
Writing dpkg log to '/var/log/unattended-upgrades/unattended-upgrades-dpkg.=
log'
All upgrades installed
```

Para verificar si hubo actualizaciones, basta con consultar los archivos de registro en /var/log/unattended-upgrades/unattended-upgrades.log. 

```bash
2017-07-08 06:24:05,337 INFO Initial blacklisted packages:
2017-07-08 06:24:05,342 INFO Initial whitelisted packages:
2017-07-08 06:24:05,342 INFO Starting unattended upgrades script
2017-07-08 06:24:05,342 INFO Allowed origins are: ['o=Debian,n=buster', 'o=Debian,n=buster-updates', 'o=Debian,n=buster-proposed-updates', 'o=Debian,n=buster,l=Debian-Security', 'origin=Debian,codename=buster,label=Debian-Security']
2017-07-08 06:24:07,485 INFO No packages found that can be upgraded unattended and no pending auto-removals
2017-07-09 06:51:40,881 INFO Initial blacklisted packages:
2017-07-09 06:51:40,881 INFO Initial whitelisted packages:
2017-07-09 06:51:40,881 INFO Starting unattended upgrades script
2017-07-09 06:51:40,881 INFO Allowed origins are: ['o=Debian,n=buster', 'o=Debian,n=buster-updates', 'o=Debian,n=buster-proposed-updates', 'o=Debian,n=buster,l=Debian-Security', 'origin=Debian,codename=buster,label=Debian-Security']
2017-07-09 06:51:43,013 INFO No packages found that can be upgraded unattended and no pending auto-removals
```


## 2.3 Acceso Remoto

Como regla general, no se accede físicamente a un servidor salvo por razones excepcionales (como actualizaciones o reparaciones de hardware, por ejemplo).

La mejor manera de gestionar un servidor es de forma remota. Sin embargo, este acceso debe realizarse de manera segura y garantizar que la comunicación no sea interceptada por terceros.


### 2.3.1 El Protocolo SSH

El protocolo SSH (abreviatura de Secure Shell) es un protocolo de comunicación que cifra todos los datos intercambiados, haciendo prácticamente imposible la violación de la privacidad en la comunicación.

El protocolo SSH es extremadamente versátil, y actualmente existen software cliente que permiten el acceso a la línea de comandos, transferencia de archivos y la creación de túneles seguros como soporte de comunicación para otros protocolos.

__Clientes SSH__

Los clientes SSH se dividen en dos grandes grupos:

Terminal SSH: Emulador de terminal que permite acceder remotamente a la línea de comandos utilizando el protocolo SSH.

Cliente SFTP: Aplicación cliente para la transferencia de archivos que utiliza el Protocolo de Transferencia Segura de Archivos (Secure File Transfer Protocol, SFTP).

__Clientes para Linux__

- [openssh-client](https://packages.debian.org/stable/net/openssh-client): Paquete de software que proporciona utilidades para acceso remoto (cliente SSH), copia segura de archivos (SCP) y transferencia segura de archivos (SFTP), entre otros.

- [FileZilla](https://packages.debian.org/stable/net/openssh-client): Cliente SFTP.

__Clientes para Windows__

- [Git](https://git-scm.com/downloads): Terminal SSH.
- [PuTTY](https://www.putty.org): Terminal SSH.



### 2.3.2 Servidor SSH

Regla general, no se debe acceder físicamente a un servidor a menos que sea por razones excepcionales (como actualización o reparación de hardware). La mejor manera de administrar un servidor es de forma remota. Sin embargo, este acceso debe realizarse utilizando un protocolo seguro para garantizar que las comunicaciones no sean interceptadas por terceros.

__Instalación__

```bash
root@server:~# apt install openssh-server openssh-client
```

__Configuración__
Todas las configuraciones del servidor SSH se encuentran en el archivo /etc/ssh/sshd_config.

Especificar las direcciones en las que el servicio responderá. En este caso, solo se aceptarán conexiones en la dirección 192.168.1.100:

```bash
cat /etc/ssh/sshd_config:

plaintext
Copiar
Editar
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
ListenAddress 192.168.1.100

#[...]
```

Por seguridad, el servidor SSH no permite el acceso remoto como root de forma interactiva. Por lo tanto, para obtener privilegios de root, primero se debe iniciar sesión como un usuario no privilegiado y luego elevar los privilegios. De esta manera, la contraseña de root no está expuesta a ataques de fuerza bruta.

```bash 
cat /etc/ssh/sshd_config:

#[...]

# Authentication:

#LoginGraceTime 2m
#PermitRootLogin yes
PermitRootLogin no

#[...]
```

Verificar que no se permitan inicios de sesión con contraseñas vacías:

```bash
cat /etc/ssh/sshd_config:

#[...]

#PermitEmptyPasswords no

#[...]
```

Permitir que los usuarios inicien sesión con sus contraseñas:

```bash
cat /etc/ssh/sshd_config:

plaintext
Copiar
Editar
#[...]

PasswordAuthentication yes

#[...]
```

Verificar el archivo de configuración:

```bash 
root@server:~# sshd -t
root@server:~#
```

Reiniciar el servicio:

```bash
root@server:~# systemctl restart ssh
```

__Verificación__

Clientes Linux

Ahora debería ser posible establecer una conexión SSH a la dirección 192.168.1.100.

La primera vez que se establezca esta conexión, se debe confirmar la autenticidad del servidor, ya que este aún no figura en la lista de sistemas conocidos por el cliente.

```bash
ssh bett0@192.168.1.2
The authenticity of host '192.168.1.2 (192.168.1.2)' can't be established.
ED25519 key fingerprint is SHA256:XOj9MZeGhHo4kmKY1suLrce2ZSn3LhRR/X/EMD4DLyM.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.2' (ED25519) to the list of known hosts.
bett0@192.168.1.2's password:
Linux potosi 6.1.0-37-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.140-1 (2025-05-22) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jun 13 21:22:37 2025
bett0@potosi:~$ logout
Connection to 192.168.1.2 closed.
```

Los inicios de sesión con el usuario root no serán aceptados:

```bash
fribeiro@laptop:~$ ssh root@192.168.1.100
root@192.168.1.100´s password:
Permission denied, please try again.
root@192.168.1.100´s password:
Permission denied, please try again.
root@192.168.1.100´s password:
Permission denied (publickey,password).
fribeiro@laptop:~$
```

__Clientes Windows__

El acceso desde clientes Windows es posible con un programa emulador de terminal que soporte SSH, como PuTTY:

Sesión remota vía SSH con PuTTY o Git
![ssh](img/2.3.2_1.png)

__Obtener privilegios de root__
Dado que el inicio de sesión como root está deshabilitado, la manera de obtener privilegios de root en una conexión SSH es iniciando sesión como un usuario común y luego elevando los privilegios con el comando su:

```bash
~$ssh bett0@192.168.1.2
bett0@192.168.1.2's password:
Linux potosi 6.1.0-37-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.140-1 (2025-05-22) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jun 13 21:24:32 2025 from 192.168.1.12
bett0@potosi:~$ su -
Contraseña:
root@potosi:~#
```


### 2.3.3 Acceso vía SSH sin Contraseña

Más seguro que utilizar una contraseña, ¡es no usar ninguna contraseña!

El acceso a un sistema remoto vía SSH sin contraseña es posible al generar un par de claves RSA.

__Configuración__

La configuración se realiza en dos fases: en la primera, se genera un par de claves (pública y privada), y luego se instala la clave pública en el o los sistemas remotos.

__Configuración local__

Durante la generación del par de claves, se solicitará una contraseña para protegerlas. Esta contraseña es opcional: si se ingresa, será requerida al intentar acceder al sistema remoto. En este caso, optaremos por no usar contraseña.

```bash
bett0@potosi:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bett0/.ssh/id_rsa):
Created directory '/home/bett0/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/bett0/.ssh/id_rsa
Your public key has been saved in /home/bett0/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:J/5tasOqB6HKzJy87hB69//g5kbgLkAWgF2gqWlOHm0 bett0@potosi
The key's randomart image is:
+---[RSA 3072]----+
|oo.o.            |
|.oo              |
|o  .             |
|..+   o          |
|+* E o oS .      |
|*.+ . o..o       |
|oO.+.. oo.       |
| oO.....+o+..    |
| o+. .oB=+++.    |
+----[SHA256]-----+
```

Al final de este proceso, se generarán 2 archivos en el directorio ~/.ssh: la clave pública y la clave privada.


- __id_rsa__	La clave privada: debe permanecer en el sistema local y NUNCA debe ser compartida ni divulgada.
- __id_rsa.pub__	La clave pública: debe instalarse en los sistemas remotos para permitir el acceso sin contraseña.

__Configuración remota__

El primer paso de la configuración consiste en copiar la clave pública desde el sistema local al sistema remoto. Esta clave pública se almacenará en el archivo ~/.ssh/authorized_keys del sistema remoto.

```bash
bett0@potosi:~$ ssh-copy-id bett0@192.168.1.20
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/bett0/.ssh/id_rsa.pub"
The authenticity of host '192.168.1.20 (192.168.1.20)' can't be established.
ED25519 key fingerprint is SHA256:goTv4d/Po/NYVD0cNG7YTGQbwop93HdhiZOWo6F4WBs.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
bett0@192.168.1.20's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bett0@192.168.1.20'"
and check to make sure that only the key(s) you wanted were added.
```

La configuración restante se realiza en el sistema remoto. Por ahora, todavía será necesario acceder con la contraseña:

```bash
$ ssh bett0@192.168.1.6
bett0@192.168.1.6's password:
Linux potosi 6.1.0-37-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.140-1 (2025-05-22) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Jun 21 21:40:55 2025 from 192.168.1.20
bett0@potosi:~$
```

Opción: habilitar logins como root
Con esta configuración, es seguro habilitar inicios de sesión como root sin contraseñas, únicamente usando claves. Para ello, modifique la opción PermitRootLogin en el archivo /etc/ssh/sshd_config por PermitRootLogin without-password:

```bash
cat /etc/ssh/sshd_config:

plaintext
Copiar
Editar
#[...]

# Authentication:
LoginGraceTime 120
# PermitRootLogin prohibit-password
PermitRootLogin without-password
StrictModes yes

#[...]
```

Verifique el archivo de configuración:

```bash
root@potosi:~# sshd -t
root@potosi:~#
```

Reinicie el servicio:

```bash
root@potosi:~# systemctl restart ssh
```

__Verificación__

Si la configuración es correcta, será posible acceder al sistema remoto sin que se solicite la contraseña:

```bash
bett0@potosi:~$ ssh 192.168.1.100

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
bett0@potosi:~$
```


## 2.4 Reloj del Sistema

En un servidor, es esencial garantizar que el reloj del sistema funcione correctamente. Además de que varios servicios dependen de ello, el reloj puede usarse para sincronizar otros sistemas.

### 2.4.1 Fecha, Hora y Zona Horaria

Ajustar la fecha y hora del sistema.

__Configuración__

__Zona horaria__

Es imperativo configurar correctamente la zona horaria en la que se encuentra el servidor. La información sobre la zona horaria se encuentra en el archivo /etc/timezone y puede consultarse con el comando cat:

```bash
root@potosi:~# cat /etc/timezone
America/La_Paz
```

Para configurar la zona horaria, utiliza el comando dpkg-reconfigure tzdata y selecciona la zona correcta en el menú interactivo:


```bash
root@potosi:~# dpkg-reconfigure tzdata
```

Un diálogo permitirá elegir la región geográfica:

![Dialogo](img/241_1.png)

Ajuste del reloj del sistema: zona horaria

A continuación, selecciona la región o el país donde el servidor está físicamente ubicado:

![Dialogo](img/241_2.png)

Ajuste del reloj del sistema: ciudad o región

El resultado se mostrará de la siguiente manera:

```bash
Current default time zone: 'America/La_Paz'
Local time is now:      Sat Jun 21 21:51:39 -04 2025.
Universal Time is now:  Sun Jun 22 01:51:39 UTC 2025.

```

__Fecha y hora__
El comando date muestra la fecha actual del sistema:

```bash
root@potosi:~# date
sáb 21 jun 2025 21:54:28 -04
```

El comando date también permite ajustar manualmente la hora del sistema utilizando la sintaxis abreviada date <MMDDhhmm>:

```bash
root@potosi:~# date 06182140
mié 18 jun 2025 21:40:00 -04
```

### 2.4.2 Protocolo NTP

El protocolo NTP (Network Time Protocol) se utiliza para mantener el reloj del sistema correcto, sincronizándolo a partir de una red de servidores NTP vía Internet.

__Objetivo__

Utilizar la red de servidores NTP para ajustar el reloj del sistema.

__Instalación__

```bash
root@potosi:~# apt install ntpdate ntp-doc
```

__Utilización__

En primer lugar, verifica que la zona horaria del sistema está correctamente definida. Ver Zona horaria.

A continuación, ajusta el reloj del sistema utilizando como referencia uno de los servidores NTP:

```bash
root@potosi:~# ntpdate -u pool.ntp.org
2025-06-21 21:57:15.227683 (-0400) +0.075146 +/- 0.101719 pool.ntp.org 143.107.229.211 s1 no-leap
```

### 2.4.3 Servidor NTP
El protocolo NTP se utiliza para mantener el reloj del sistema correcto, sincronizándolo a partir de otros servidores NTP a través de Internet. Una vez instalado y configurado, el servidor NTP puede utilizarse para sincronizar otros sistemas.

Instalación

```bash
root@potosi:~# apt install ntp ntp-doc~
```

__Configuración__

Verifica la configuración de la zona horaria y la fecha y hora del sistema.

Verificación
El comando ntpq -p permite verificar a qué servidores NTP estamos conectados. Los símbolos *, + y - indican, respectivamente, conexiones exitosas, sincronizaciones en curso y servidores menos confiables. Puede tomar algunos minutos para que aparezca la lista y hasta 30 minutos para que ocurra la primera corrección.

```bash
root@server:~# ntpq -p
root@potosi:~# ntpq -p
     remote            refid      st t when poll reach   delay   offset   jitter
================================================================================
 0.debian.pool.nt .POOL.          16 p    -   64    0   0.0000   0.0000   0.0001
 1.debian.pool.nt .POOL.          16 p    -   64    0   0.0000   0.0000   0.0001
 2.debian.pool.nt .POOL.          16 p    -   64    0   0.0000   0.0000   0.0001
 3.debian.pool.nt .POOL.          16 p    -   64    0   0.0000   0.0000   0.0001
```

El servidor NTP está listo para ser utilizado.

__Utilización__

__Clientes Linux__

Para ajustar la fecha y hora de un sistema cliente, basta con usar el comando ntpdate (ver Protocolo NTP), indicando la dirección del servidor:

```bash
root@potosi:~# ntpdate -u 192.168.1.6
2025-06-21 22:03:01.760078 (-0400) +0.000007 +/- 0.000044 192.168.1.6 s2 no-leap
```

__Clientes Windows__

En sistemas Windows, podemos configurar el reloj del sistema para sincronizarlo con nuestro servidor NTP:

Configuración del servidor NTP en Microsoft Windows 11.
![WindowsReloj](img/243_1.png)

## 2.5 Usuarios

De que sirve un sistema si no tiene usuarios?

### 2.5.2 Cuotas

El espacio en disco no es infinito, por lo que a veces es necesario limitar la cantidad de datos (cuotas) que cada usuario puede almacenar.

En esta configuración, definiremos cuotas para los usuarios en la partición /dev/sda6, montada en /home.

__Instalación__

```bash
root@potosi:~# apt install quota
```

__Configuración__

El sistema de archivos debe montarse con las opciones necesarias para soportar cuotas. Para ello, debemos editar el archivo de configuración /etc/fstab y añadir las opciones usrjquota=aquota.user,grpjquota=aquota.group,jqfmt=vfsv0 en las opciones de montaje de /home:

```bash
root@potosi:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# systemd generates mount units based on this file, see systemd.mount(5).
# Please run 'systemctl daemon-reload' after making changes here.
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=0ee12c91-1204-4a25-8abc-ccf198bad354 /               ext4    errors=remount-ro 0       1
# /home was on /dev/sda6 during installation
UUID=410154d9-056e-480f-8470-1db7a8d1c720 /home           ext4    defaults        0       2
# swap was on /dev/sda5 during installation
UUID=53626796-6d67-4599-958f-43b02bd9545d none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
```

El sistema de archivos debe volver a montarse para aplicar los cambios:

```bash
root@potosi:~#  mount -o remount /home
```

A continuación, inicializamos el sistema de cuotas:

```bash
root@potosi:~# quotacheck -cugm /home
quotacheck: Mountpoint (or device) /home not found or has no quota enabled.
quotacheck: Cannot find filesystem to check or filesystem not mounted with quota option.
```

Finalmente, activamos el sistema de cuotas:

```bash
root@potosi:~# quotaon -guvp /home
group quota on /home (/dev/sda6) is on
user quota on /home (/dev/sda6) is on
```

__Gestión de cuotas__

Definición de cuotas
Las cuotas pueden definirse por usuario utilizando el editor de cuotas con el comando edquota -u <usuario>:

```bash
root@potosi:~# edquota -u bett0
```

El valor hard es el límite absoluto que el usuario puede utilizar, mientras que el valor soft puede ser excedido por un tiempo limitado, definido como un período de gracia (grace time), que por defecto es de 7 días.

Ejemplo:

```plaintext

Disk quotas for user fribeiro (uid 1000):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/sda6                      4852   10223616   10485760        469        0        0
```

En este caso, hemos definido cuotas de 9,5 GB (10223616 KB) como límite soft y 10 GB (10485760 KB) como límite hard para el usuario fribeiro.

__Verificación de cuotas__

Las cuotas por usuario pueden consultarse con el comando quota:

```bash
root@potosi:~# quota -s bett0
Disk quotas for user fribeiro (uid 1000):
     Filesystem   space   quota   limit   grace   files   quota   limit   grace
      /dev/sda6   4852K   9984M  10240M             469       0       0
```

También es posible generar un reporte de las cuotas para todos los usuarios con el comando repquota:

```bash
root@potosi:~# repquota -as
*** Report for user quotas on device /dev/sda6
Block grace time: 7days; Inode grace time: 7days
                        Space limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --     28K      0K      0K              4     0     0
bett0     --   4852K   9984M  10240M            469     0     0
```