**PARA DESPUES DE SEGUNDA PRACTICA NO USAR VIRT-MANAGER SALVO EXCEPCIONES** (VER QUE SE HA CREADO DE VERDAD POR EJEMPO)

//Maquinas definidas en 203
	vm3 = 2001:470:736b:2ff::3 (DNS maestro)
	vm4 = 2001:470:736b:2ff::4 (DNS esclavo)
	vm5 = 2001:470:736b:2ff::5 (Servidor NFS)

//maquinas definidas en 199
	vm6 = 2001:470:736b:2fe:5054:ff:fe02:ff06 (Cliente NFS) 
		maquina con IP dinamica

//definimos las maquinas VM3 y VM4
qemu-img create -f qcow2 -o backing_file=o2.qcow2,backing_fmt=qcow2 o2ff3.qcow2

qemu-img create -f qcow2 -o backing_file=o2.qcow2,backing_fmt=qcow2 o2ff4.qcow2

//copiamos el fichero xml
cp o2.xml o2ff3.xml
cp o2.xml o2ff4.xml

//modificamos grupo y permisos de los ficheros
chmod 660 o2ff3.qcow2
chgrp vmu o2ff3.xml

//modificamos los ficheros xml con el name, uuid, mac y dir

//definimos las maquinas
virsh -c qemu+ssh://a868801@155.210.154.203/system define o2ff3.xml
virsh -c qemu+ssh://a868801@155.210.154.203/system define o2ff4.xml

//Cambiamos el nombre de las maquinas en /etc/myname
ns1 y ns2

//creamos el fichero /ect/hostname.vlan299 con el siguiente contenido para asignar una IPv6 estatica
```
vlan 299 vlandev vio0 up
inet 2001:470:736b:2ff::3 64 -soii -temporary
-autonconfprivacy
```

//modificamos el fichero vio0 con el siguiente contenido para que no asigne IPv6 de manera automatica
```
-inet6
up
```

---------AÑADIR LA SINCRONIZACIÓN DE TIEMPO COMO CLIENTE ROUTER


--- Una vez configuradas las maquinas anteriores
# Parte 1, DNS

//modificamos el fichero /etc/rc.conf.local añadiendo nsd_flags=""

//activamos el servicio nsd de nuevo
doas rcctl check nsd
doas rcctl restart nsd

//modificamos el fichero /var/nsd/etc/nsd.conf donde modificamos las secciones de server, remote-control, key, pattern y las dos zones (inversa y directa)

## Master

/var/nsd/etc/nsd.conf
```
server:
        ip-address: 2001:470:736b:2ff::3
        hide-version: yes
        database: "/var/nsd/db/nsd.db"
        verbosity: 1
        logfile: "/var/log/nsd.log"
        pidfile: "/var/nsd/run/nsd.pid"
        ip6-only: yes
        zonesdir: "/var/nsd/zones"

remote-control:
        control-enable: yes
        control-interface: /var/run/nsd.sock

pattern:
        name: "toslave"
        notify: 2001:470:736b:2ff::4 NOKEY
        provide-xfr: 2001:470:736b:2ff::4 NOKEY

zone:
        name: "2.ff.es.eu.org"
        zonefile: "2.ff.es.eu.org.directo"
        include-pattern: "toslave"

zone:
        name: "2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa"
        zonefile: "2.ff.es.eu.org.inverso"
        include-pattern: "toslave"

```

**Zona directa** /var/nsd/zones/2.ff.es.eu.org.directo
```
$ORIGIN 2.ff.es.eu.org. ; default zone domain
$TTL 86400
@       IN      SOA     ns1.2.ff.es.eu.org. 868801.unizar.es. (
        2013022502      ; serial number
        21600           ; refresh
        3600            ; retry
        604800          ; expire
        86400   )       ; TTL

@       IN NS ns1.2.ff.es.eu.org.
@       IN NS ns2.2.ff.es.eu.org.

ns1     IN      AAAA    2001:470:736b:2ff::3
ns2     IN      AAAA    2001:470:736b:2ff::4
router1 IN      AAAA    2001:470:736b:2ff::1

```
**Zona indirecta** /var/nsd/zones/2.ff.es.eu.org.inverso
```
$ORIGIN 2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa.
$TTL 86400
@       IN      SOA     ns1.2.ff.es.eu.org. 868801.unizar.es. (
        2013022504      ; serial number
        21600           ; refresh
        3600            ; retry
        604800          ; expire
        86400   )       ; TTL

@       IN NS ns1.2.ff.es.eu.org.
@       IN NS ns2.2.ff.es.eu.org.

3.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.f.f     IN      PTR ns1.2.ff.es.eu.org.
4.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.f.f     IN      PTR ns1.2.ff.es.eu.org.
1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.f.f     IN      PTR router1.2.ff.es.eu.org.

```

//comprobamos que esten bien escritos los ficheros
nsd-checkconf /var/nsd/etc/nsd.conf 
nsd-checkzone 2.ff.es.eu.org /var/nsd/zones/2.ff.es.eu.org.directo
nsd-checkzone 2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa /var/nsd/zones/2.ff.es.eu.org.inverso

//no existia el fichero nsd.db
nsd -f /var/nsd/db/nsd.db   //para crear el archivo de base de datos

nsd-control-setup
nsd-control reload


## Slave

/var/nsd/etc/nsd.conf
```
server:
        hide-version: yes
        ip-address: 2001:470:736b:2ff::4
        database: "/var/nsd/db/nsd.db"
        logfile: "/var/log/nsd.log"
        pidfile: "/var/nsd/run/nsd.pid"
        verbosity: 1
        zonesdir: "/var/nsd/zones"

remote-control:
        control-enable: yes
        control-interface: /var/run/nsd.sock

zone:
        name: "2.ff.es.eu.org"
        zonefile: "2.ff.es.eu.org.directo"
        allow-notify: 2001:470:736b:2ff::3 NOKEY
        request-xfr: AXFR 2001:470:736b:2ff::3 NOKEY

zone:
        name: "2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa"
        zonefile: "2.ff.es.eu.org.inverso"
        allow-notify: 2001:470:736b:2ff::3 NOKEY
        request-xfr: AXFR 2001:470:470:736b:2ff::3 NOKEY

```

//en ambas maquinas ns1 y ns2
nsd-control reload

//comprobaciones nsd
//desde el slave
	nsd-control zonestatus 2.ff.es.eu.org
	nsd-control zonestatus 2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa.
	//preguntamos al sevidor de google cual es la IPv6 de ns1.2.ff.es.eu.org
	dig -6 @2001:4860:4860::8888 AAAA ns1.2.ff.es.eu.org
	//preguntamos al servidor de google que dominio está asociado a esta dirección IPv6 (busqueda inversa)
	dig -6 @2001:4860:4860::8888 -x 2001:470:736b:2ff::3
	//preguntamos al servidor de la maquina ns1 sobre el nombre del dominio asociado a la IP de la maquina ns2
	dig -6 @2001:470:736b:2ff::3 -x 2001:470:736b:2ff::4 
	//preguntamos al servidor de la maquina ns1 sobre la IPv6 del orouter2
	dig -6 @2001:470:736b:2ff::3 AAAA router1.2.ff.es.eu.org.


//dig directo
dig -6 @2001:470:736b:2ff::3 AAAA ns1.2.ff.es.eu.org
//dig inverso
dig -6 @2001:470:736b:2ff::3 -x 2001:470:736b:2ff::4 

//cambiamos la IPv6 de la maquina o2ff2 a estatica y no dinámica
//fichero /var/unbound/etc/unbound.conf
```
server:
        interface: 0.0.0.0
        interface: ::1
        interface: ::0

        access-control: 0.0.0.0/0 refuse # bloqueamos el trafico IPv4
        access-control: 2001:470:736b::/48 allow # permitimos acceso dentro de este rango
        access-control: ::0/0 refuse # bloqueamos el trafico IPv6
        access-control: ::1 allow # permitimos localhost

        hide-identity: yes
        hide-version: yes

        verbosity: 1

remote-control:
        control-enable: yes
        control-interface: /var/run/unbound.sock

forward-zone:
        name: "."                               # use for ALL queries
        forward-addr: 2001:470:20::2            # servidor hurricane
#       forward-first: yes                      # try direct if forwarder fails

stub-zone:
        name: "2.ff.es.eu.org"
        stub-addr: 2001:470:736b:2ff::3
        stub-addr: 2001:470:736b:2ff::4

stub-zone:
        name: "2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa."
        stub-addr: 2001:470:736b:2ff::3
        stub-addr: 2001:470:736b:2ff::4
```

//comprobamos que este bien escrito el fichero
unbound-checkconf

//reiniciamos el servidor 
rcctl restart unbound

//comprobaciones
dig -6 @2001:470:736b:2ff::2 -x 2001:470:736b:2ff::1
dig -6 @2001:470:736b:2ff::2 -x 2001:470:736b:2ff::4   

//añadir nueva maquina "nuevo_servidor" a las zonas
	//en directo
	```otro_servidor   IN      AAAA    2001:470:736b:2ff::f```
	//en inverso
	```f.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.f.f     IN      PTR otro_servidor.2.ff.es.eu.org.```

//algunas pruebas para esta nueva maquina
dig @2001:470:736b:2ff::2 -x 2001:470:736b:2ff::f
dig @2001:470:736b:2ff::2 AAAA otro_servidor.2.ff.es.eu.org.

//poner en /etc/resolv.conf
nameserver 2001:470:736b:2ff::2

# Parte 2,  Servicio NFS sobre FreeBSD y ZFS y cliente NFS sobre Alpine Linux

## 2.1 Maquinas base FreeBSD y Alpine Linux
//Definimos las nuevas maquinas frb2 y al2, maquinas base con las siguientes especidficaciones en el archivo xml
- name: frb2 y al2 
- uuid: 2115 y 2116
- source-file: lo mismo que el name al final de la ruta
- mac address: FF:05 y FF06 respectivamente

//les damos permisos de grupo
chmod 660 frb2.qcow2 //solo para los qcow2
chgrp vmu frb2.xml

### frb2
Empezamos con frb2
// en el fichero /etc/resolv.conf 
```
nameserver 2001:4860:4860::8888
```

//en fichero /etc/rc.conf, añadir al final del todo
```
ipv6_enable="YES" 
ipv6_defaultrouter="2001:470:736b:f000::1" # gateway
ifconfig_vtnet0_ipv6="inet6 accept_rtadv"
```

//tras esto hacer
. /etc/netstart

//**error de almacenamiento** -> desdefinimos la maquina y aumentamos el tamaño de esta en las lineas memory unit y currentMemory unit
le ponemos 512\*1024  = 524288

//descargamos el doas
pkg install doas

//configuramos el doas donde añadimos al fichero /usr/local/etc/doas.conf la linea
permit nopass :wheel

//probamos el comando para el usuario recien creado con por ejemplo
doas su

//desde central copio mis claves a la maquina 
scp ~/.ssh/authorized_keys a868801@[2001:470:736b:f000:5054:ff:fe02:ff05]:~/.ssh/authorized_keys

//añadimos la siguiente linea al final del fichero /etc/rc.conf
```
sysrc chronyd_enable="YES"
```

service chronyd start

//añadimos al fichero /usr/local/etc/chrony.conf
```
server 2001:470:736b:2ff::1 iburst
server 2001:470:736b:2ff::2 iburst
```

iburst hace que, si el servidor no responde, se envíen rápidamente varias solicitudes en lugar de una, reduciendo el tiempo de sincronización inicial.


service chronyd restart
	//con suerte funciona tras 15 min de espera para el stratum del router
//para comprobar que se han sincronizado los relojes
chronyc tracking 

//modificamos el fichero /etc/resolv.conf con la siguiente linea
```
nameserver 2001:470:736b:2ff::2 
```
//con el fin de activar solo la @ link-local Ipv6 en la tarjeta de red

finalmente 
shutdown -h now

### al2

//creamos el nuevo usuario 
adduser a868801 -u 1000 -G wheel
// e introducimos la contraseña despues

//modificamos el fichero /etc/doas.conf con 
permit persist :wheel
permit nopass :wheel

//modificamos el fichero /etc/resolv.conf
```
nameserver 2001:4860:4860::8888 # acceso a internet, luego se elimina
nameserver 2001:470:736b:2ff::2 
```

//copiamos las claves de central a la maquina
scp ~/.ssh/authorized_keys a868801@[2001:470:736b:f000:5054:ff:fe02:ff06]:~/claves
cat claves >> .ssh/authorized_keys

//en el fichero /etc/chrony/chrony.conf añadir las lines 
```
server 2001:470:736b:2ff::1 iburst
server 2001:470:736b:2ff::2 iburst
```

//reiniciamos el servicio chronyd
rc-service chronyd restart

//comprobamos que funcione
chronyc tracking

//Para poder tener solo la ipv6 local modificamos el fichero /etc/network/interfaces y ponemos lo siguiente en eth0
```
auto eth0
iface eth0 inet6 manual
	up ip link set dev eth0 up
	up sysctl -w net.ipv6.conf.eth0.autoconf=0 # deshailitamos slaac
	up sysctl -w net.ipv6.conf.eth0.accept_ra=0 # deshabilitamos ra
```
//esto en la maquina base se cambia 

//reiniciamos la red
service networking restart

//comprobamos con ifconfig que solo exista la ipv6 local


## 2.2 Servidor NFS sobre FreeBSD y ZFS

virsh -c qemu+ssh://a868801@155.210.154.199/system undefine al2
virsh -c qemu+ssh://a868801@155.210.154.203/system undefine frb2

virsh -c qemu+ssh://a868801@155.210.154.199/system define al2ff6.xml
virsh -c qemu+ssh://a868801@155.210.154.203/system define frb2ff5.xml

//poner esto en /etc/rc.conf
```
hostname="nfs1"
keymap="es.kbd"
sshd_enable="YES"
moused_nondefault_enable="NO"
\# Set dumpdev to "AUTO" to enable crash dumps, "NO" to disable
dumpdev="NO"
zfs_enable="YES"
ipv6_enable="YES"
ipv6_defaultrouter="2001:470:736b:f000::1" # gateway
ifconfig_vtnet0_ipv6="inet6 -auto_linklocal" # que no reciba automaticamente direcciñon IPv6
vlans_vtnet0="299"
ifconfig_vtnet0_299="inet6 2001:470:736b:2ff::5/64" # IPv6 de la maquina
# Servicio chrony
sysrc chronyd_enable="YES"
chronyd_enable="YES"
```

//hacemos reboot
reboot

//En maquina ns1 (o 3)
//añadimos esto al fichero /var/nsd/zones/2.ff.es.eu.org.directo
```
nfs1  IN  AAAA  2001:470:736b:2ff::5
frb2ff5 IN CNAME nfs1.2.ff.es.eu.org.
```
aumentamos el serial

//añadimos esto al fichero /var/nsd/zones/2.ff.es.eu.org.inverso
```
5.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.f.f     IN      PTR nfs1.2.ff.es.eu.org.
```
aumentamos serial

//comprobacion con dig desde ns1 (o2ff3)
dig -6 @2001:470:736b:2ff::2 AAAA nfs1.2.ff.es.eu.org 

//comprobaciones desde frb2ff5
host 2001:470:736b:2ff::3


### Añadir disco adicional de 300MB

//crear un nuevo disco
 qemu-img create -f qcow2 /misc/alumnos/as2/as22024/a868801/nfs1-disk2.qcow2 300M

//le damos permisos
chmod 660 nfs1-disk2.qcow2

//asociamos el disco a la maquina frb2ff5
virsh -c qemu+ssh://a868801@155.210.154.203/system attach-disk frb2ff5 /misc/alumnos/as2/as22024/a868801/nfs1-disk2.qcow2 vdb --persistent --driver qemu --subdriver qcow2 --targetbus virtio

//verificar los discos
virsh -c qemu+ssh://a868801@155.210.154.203/system domblklist frb2ff5
salida:
```
Destino    Fuente
------------------------------------------------
vda        /misc/alumnos/as2/as22024/a868801/frb2ff5.qcow2
vdb        /misc/alumnos/as2/as22024/a868801/nfs1-disk2.qcow2
```


//comprobamos que se haya montado en la maquina frb2ff5
ls /dev/vtbd*
	//si aparece /dev/vtbd0 se supone montado

//creamos la tabla de particiones
gpart create -s gpt vtbd0

//comprobamos el espacio
gpart show /dev/vtbd0

//añadimos una partición para zfs
gpart add -t freebsd-zfs vtbd0

//creamos el pool zfs "netshare"
zpool create netshare /dev/vtbd0p1

//comprobamos que se haya creado
zpool list netshare

//creamos los sistemas de archivos
zfs create netshare/netlogs
zfs create netshare/nettmp

//ahora creamos un nuevo dataset para netuser
zfs create zroot/home/netuser
zfs set mountpoint=/home/netuser zroot/home/netuser

//ahora le añadimos la UID 1010 al user
 pw useradd netuser -u 1010 -m -d /home/netuser -s /bin/sh
chown -R netuser:netuser /home/netuser
passwd netuser

//activamos el servicio NFS en frb2ff5 en el fichero /etc/rc.conf
	//lo ponemos al final del fichero
``` 
rpcbind_enable="YES"
nfs_server_enable="YES" # permitimos compartir archivos en la red
mountd_enable="YES" # servicio de exportacion NFS
nfsv4_server_enable="YES"
nfsuserd_enable="NO"  
```


//exportamos los directorios
	//para NFS version 3 exportamos /home/user sin que un root externo adquiera el uid 0
	//para NFS version 4 exportamos /netshare como raiz
//en /etc/exports
```
/home/netuser -network 2001:470:736b:2ff::/64 -maproot=0
V4: /netshare # raiz de exportacion
/netshare/netlogs -network 2001:470:736b::/48 -maproot=root
/netshare/nettmp -network 2001:470:736b::/48 -maproot=root
```

//reiniciamos los servicios nsfd y mountd
service nfsd restart
service mountd restart

//creacion de un script para la creacion de un nuevo user en los servidores DNS
```
#!/bin/sh
server=(2001:470:736b:2ff::2 2001:470:736b:2ff::3 2001:470:736b:2ff::4)
for host in ${server[@]}
do
    ssh $host "doas useradd -u 1010 -m -d /home/netusr -s /bin/ksh netusr"
done
```

## 2.3 Cliente NFS sobre Alpine Linux

### Configurar la vlan298
//creamos la vlan298, en /etc/hostname.vlan298
```
inet6 alias 2001:470:736b:2fe::1 64 vlan 298 vlandev vio0
up
-autoconfprivacy
```

//añadimos lo siguiente en /etc/rad.conf debajo de la anterior vlan299
```
interface vlan298
```

//reinicamos la red con 
. /etc/netstart

//reiniciamos el servicio rad
 rcctl restart rad

### Configuración de al2ff6

//Configuramos la vlan
ip link add link eth0 name eth0.298 type vlan id 298
ip link set dev eth0.298 up

//introducir lo siguiente en el fichero /etc/network/interfaces
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet manual
	up ip link set dev eth0 up

auto eth0.298
iface eth0.298 inet6 auto
	vlan-raw-device eth0
	up ip link set dev eth0.298 up
	up ip -6 addr flush dev eth0 # Elimina cualquier IPv6 asignada a eth0
	up sysctl -w net.ipv6.conf.eth0.disable_ipv6=1 # Desactiva IPv6 en eth0
	up sysctl -w net.ipv6.conf.eth0.298.accept_ra=1 # Acepta RA
    up sysctl -w net.ipv6.conf.eth0.298.use_tempaddr=0 # Desactiva direcciones temporales
	up ip -6 route add default via 2001:470:736b:2fe::1 dev eth0.298
```

//rebooteamos la maquina

//para hacer el dig o host de la maquina hacer
nslookup ns1.2.ff.es.eu.org

//creamos los directorios de montaje
mkdir -p /mnt/netlogs
mkdir -p /mnt/nettmp

//instalamos paquete NFS
apk add nfs-utils

//activamos el servicio de forma permanente ante posibles restarts
rc-update add rpc.statd default

//condirmamos que este encendido
doas service rpc.statd restart
doas service rpc.statd status

//montamos 
mount -a -t nfs nfs1.2.ff.es.eu.org:/netshare/netlogs /mnt/netlogs
mount -a -t nfs nfs1.2.ff.es.eu.org:/netshare/nettmp /mnt/nettmp

mount -t nfs -o vers=4 nfs1.2.ff.es.eu.org:/netlogs /mnt/netlogs
mount -t nfs -o vers=4 nfs1.2.ff.es.eu.org:/nettmp /mnt/nettmp
	// si quiere hacer las comprobaciones con ficheros para saber si se ha realizado el montaje solo tendras que hacer esto sin la /netshare ya que nfs4 lo preasume y parte de este


//para verificar que estan creados ejecutar
ls -l /mnt

//pruebas sobre los ficheros
	//hacer la creación del fichero y modificación del mismo desde el cliente NFS
	//verificar que se ha modificado desde el servidor NFS
touch /mnt/netlogs/fichero_prueba.txt
echo "Hola Hola Hola" > /mnt/netlogs/fichero_prueba.txt
more /mnt/netlogs/fichero_prueba.txt

more /netshare/netlogs/fichero_pruebas.txt

# Parte 3, script en Ruby
ruby -v //para ver si ya esta instalado

apt install ruby-full //instalamos en caso de no tenerlo

sudo gem install net-ssh //instalamos libreria de ssh para ruby
	//el sudo es por que sino no tiene permisos 

sudo gem install ed25519 bcrypt_pbkdf //para que soporte claves de tipo ed25519

//para hacer el comando en central desde cualquier sitio añadir en .bashrc
```
export PATH="$HOME/.gem/ruby/$(ruby -e 'print RUBY_VERSION.split(".")[0,2].join(".")'))/bin:$PATH"
export PATH="/home/a868801/.u:$PATH"
```

# Problemas encontrados
No se puede alcanzar la maquina 202 en el lab, que es donde tenia la maquina orouter2
Procedemos a cambiar la maquina virtual de locaclización
- 201 -> sigue sin poderse hacer la definición de la maquina virtual debido a un error de la maquina

Se ha decidido montar "de momento" la maquina orouter2 en la maquina fisica 199 del lab102
