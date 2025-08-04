zz## Valores para esta practica
W = 2 como siempre
XY definidas en el dibujo de la red
Z depende de la maquina


usar TCPdump y traceroute para comprobarlos

# Parte 1, Preparación
## Preparacion maquina fedora
Poner a la imagen de fedora 756MB de almacenamiento

//copiamos la imagen y la configuramos como en practicas anteriores
scp /misc/usuarios/unai/vms/fed39.qcow2 fed2.qcow2
cp o2.xml fed2.xml
chgrp vmu fed2*
chmod 660 fed2.qcow2

name: fed2
uuid: 5ceb27cf-35ce-49b8-adf2-8ac0cc5f2101
mac: 52:54:00:02:10:01
memory: 716800

virsh -c qemu+ssh://a868801@155.210.154.199/system define fed2.xml

useradd a868801 -G wheel
passwd a868801 

//ifconfig en esta maquina
ip a 

//configuramos el comando sudo para el usuario recien creado
sudo visudo
//descomentamos la linea que pone WHEEL NOPASSWD

ip maquina: 2001:470:736b:f000:b644:e65f:be0a:5990
//copiamos las claves a la maquina base
scp ~/.ssh/authorized_keys a868801@2001:470:736b:f000:b644:e65f:be0a:5990:~/.ssh/authorized_keys

//Instalar servidores NTP para chrony en /etc/chrony.conf
```
server 2001:470:736b:2ff::1 iburst
server 2001:470:736b:2ff::2 iburst
```
//en las imagenes base se verificara si esta operativo

//deshabilitar la IPv6 automatico 
```
net.ipv6.conf.ens3.use_tempaddr = 0
net.ipv6.conf.ens3.autoconf = 0
net.ipv6.conf.ens3.accept_ra = 0
```

## Creación de las vlans
### En orouter2
 //especificamos la vlan 210 en /etc/hostname.vlan210
 ```
inet6 2001:470:736b:210::1 60 vlan 210 vlandev vio0
up
# Aniadimos las rutas a las vlans internas
!route add -inet6 2001:470:736b:211::/64 2001:470:736b:210::2
!route add -inet6 2001:470:736b:212::/64 2001:470:736b:210::2
-autoconfprivacy
 ```

//añadir al final del fichero /etc/rad.conf
```
interface vlan210
```

rcctl restart rad
/etc/netstart

//añadir al fichero /etc/ntpd.conf
```
listen on 2001:470:736b:2fe::1
listen on 2001:470:736b:210::1
```

rcctl restart ntpd
//esperamos a que se sincronicen

//desdefinimos la maquina
virsh -c qemu+ssh://a868801@155.210.154.199/system undefine fed2
### Creación de orouter21
// Creamos imagen diferencial 
qemu-img create -f qcow2 -o backing_file=o2.qcow2 orouter21.qcow2
chmod 660 orouter21.qcow2

cp o2.xml orouter21.xml
chgrp vmu orouter21*
//modificamos name, uuid = 5ceb27cf-35ce-49b8-adf2-8ac0cc5f2101, mac = 52:54:00:02:10:01 y ruta
W = 2, XY = 10, Z = 1

//definimos la maquina
virsh -c qemu+ssh://a868801@155.210.154.201/system define orouter21.xml

Dentro de la maquina:

// en /etc/myname
```
orouter21
```

//cambair /etc/hostname.vio0
```
-inet6
up
-soii -temporary
```

// en /etc/mygate
```
2001:470:736b:210::1
```

//en /etc/sysctl.conf
```
net.inet6.ip6.forwarding=1
```

// se puede probar la conexion con la misma haciendo ssh
ssh a868801@2001:470:736b:210::2

//copiamos las claves
scp ~/.ssh/authorized_keys a868801@[2001:470:736b:210::2]:~/.ssh/authorized_keys

//creamos la vlan 211 en /etc/hostanme.vlan211
```
up
inet6 2001:470:736b:211::1 64 vlan 211 vlandev vio0
-autoconfprivacy
```

//creamos la vlan 212 en /etc/hostanme.vlan212
```
up
inet6 2001:470:736b:212::1 64 vlan 212 vlandev vio0
-autoconfprivacy
```

//creamos el fichero /etc/rad.conf
```
interface vlan211
interface vlan212
```

//añadir al fichero /etc/ntpd.conf
```
server 2001:470:736b:210::1 trusted
server 2001:470:736b:2ff::2 trusted
```

//en /etc/rc.conf.local
```
ntpd_flags=""
```

//reiniciamos el servicio
doas rcctl restart ntpd


//comprobamos que se sincroniza
doas ntpctl -s all


## Creación maquinas y puesta en marcha de vlans
//creamos las maquinas una a una
// donde 2 = W + 11 = XY que es la subred + Z numero maquina
qemu-img create -f qcow2 -o backing_file=fed2.qcow2 ipa2111.qcow2
qemu-img create -f qcow2 -o backing_file=fed2.qcow2 ipa2112.qcow2
qemu-img create -f qcow2 -o backing_file=fed2.qcow2 nfs2111.qcow2
qemu-img create -f qcow2 -o backing_file=fed2.qcow2 cliente2121.qcow2
qemu-img create -f qcow2 -o backing_file=fed2.qcow2 cliente2122.qcow2

//damos permisos y grupo a cada una
chgrp vmu ipa2111.qcow2
chmod 660 ipa2111.qcow2

//aumentar el tamaño de las ipas a 2 gb -> 2097152

//hacemos los ficheros de configuracion de todas y les damos permisos
W = 2
//Ficheros y su subred
IPA1, IPA2 y NFS -> XY = 11
Cliente1 y Cliente2 -> XY 12

//Para cada maquina
IPA1 Z = 2
IPA1 Z = 3
NFS Z = 4
Cliente1 Z = 5
Cliente2 Z = 6


//definimos las maquinas
virsh -c qemu+ssh://a868801@155.210.154.201/system define ipa2111.xml
virsh -c qemu+ssh://a868801@155.210.154.201/system define ipa2112.xml
virsh -c qemu+ssh://a868801@155.210.154.201/system define nfs2111.xml

virsh -c qemu+ssh://a868801@155.210.154.203/system define cliente2121.xml
virsh -c qemu+ssh://a868801@155.210.154.203/system define cliente2122.xml

### En ipa2111
sudo nmcli connection add type vlan con-name vlan211 ifname vlan211 vlan.parent ens3 vlan.id 211

sudo nmcli connection modify vlan211 \
  ipv6.addresses '2001:470:736b:211::2/64' \
  ipv6.gateway '2001:470:736b:211::1' \
  ipv6.dns '2001:470:736b:2ff::2' \
  ipv6.method manual

sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan211

sudo hostnamectl set-hostname ipa1

ip a 
sudo reboot

//no estaba la ipv6 desconectada de ens3, hacer
sudo nmcli connection modify "Wired connection 1" ipv6.method "disabled"
sudo nmcli connection up "Wired connection 1"

### En IPA2112
sudo nmcli connection add type vlan con-name vlan211 ifname vlan211 vlan.parent ens3 vlan.id 211

sudo nmcli connection modify vlan211 \
  ipv6.addresses '2001:470:736b:211::3/64' \
  ipv6.gateway '2001:470:736b:211::1' \
  ipv6.dns '2001:470:736b:2ff::2' \
  ipv6.method manual

sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan211

sudo hostnamectl set-hostname ipa2

ip a

//no estaba la ipv6 desconectada de ens3, hacer
sudo nmcli connection modify "Wired connection 1" ipv6.method "disabled"
sudo nmcli connection up "Wired connection 1"

### En NFS2111
sudo nmcli connection add type vlan con-name vlan211 ifname vlan211 vlan.parent ens3 vlan.id 211

sudo nmcli connection modify vlan211 \
  ipv6.addresses '2001:470:736b:211::4/64' \
  ipv6.gateway '2001:470:736b:211::1' \
  ipv6.dns '2001:470:736b:2ff::2' \
  ipv6.method manual

sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan211

sudo hostnamectl set-hostname nfs1

ip a

//no estaba la ipv6 desconectada de ens3, hacer
sudo nmcli connection modify "Wired connection 1" ipv6.method "disabled"
sudo nmcli connection up "Wired connection 1"

### En Cliente2111
sudo nmcli connection add type vlan con-name vlan212 ifname vlan212 vlan.parent ens3 vlan.id 212

//se tiene que hacer ip dinamica, por lo tanto
sudo nmcli connection modify vlan212 ipv6.method auto

sudo nmcli connection modify vlan212 ipv6.addr-gen-mode eui64

sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan212

sudo hostnamectl set-hostname cliente1

ip a

//no estaba la ipv6 desconectada de ens3, hacer
sudo nmcli connection modify "Wired connection 1" ipv6.method "disabled"
sudo nmcli connection up "Wired connection 1"

ssh a868801@2001:470:736b:212:5054:ff:fe02:1205

### En Cliente2112
sudo nmcli connection add type vlan con-name vlan212 ifname vlan212 vlan.parent ens3 vlan.id 212

//se tiene que hacer ip dinamica, por lo tanto
sudo nmcli connection modify vlan212 ipv6.method auto

sudo nmcli connection modify vlan212 ipv6.addr-gen-mode eui64

sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan212

sudo hostnamectl set-hostname cliente2

ip a

//no estaba la ipv6 desconectada de ens3, hacer
sudo nmcli connection modify "Wired connection 1" ipv6.method "disabled"
sudo nmcli connection up "Wired connection 1"

ssh a868801@2001:470:736b:212:5054:ff:fe02:1206

# Parte 2, Todo lo demas

### Servidor freeipa

#### ipa2111
//instalar servidor ipa
sudo dnf install freeipa-server
sudo dnf install freeipa-server-dns
sudo dnf clean all

//definimos los dominios
2001:470:736b:211::2 ipa1.1.2.ff.es.eu.org ipa1
2001:470:736b:211::3 ipa2.1.2.ff.es.eu.org ipa2
2001:470:736b:211::4 nfs1.1.2.ff.es.eu.org nfs1
2001:470:736b:212:5054:ff:fe02:1205 cliente1.1.2.ff.es.eu.org cliente1
2001:470:736b:212:5054:ff:fe02:1206 cliente2.1.2.ff.es.eu.org cliente2

sudo hostnamectl set-hostname ipa1.1.2.ff.es.eu.org

sudo ipa-server-install --setup-dns 

sudo kinit admin //introducimos la contraseña anteriormente puesta
sudo klist
sudo ipa config-mod --defaultshell=/bin/bash //establecemos la shell a bash
sudo firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,dns,ntp}
sudo firewall-cmd --runtime-to-permanent

#### Añadir a o2ff2
```
stub-zone:
	name: "1.2.ff.es.eu.org"
	stub-addr: 2001:470:736b:211::2
	stub-addr: 2001:470:736b:211::3
stub-zone:
	name: "1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa."
	stub-addr: 2001:470:736b:211::2
	stub-addr: 2001:470:736b:211::3
```

#### En ipa2111

sudo ipa dnszone-add 1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa.
sudo ipa dnsrecord-add 1.2.ff.es.eu.org ipa1 --aaaa-ip-address=2001:470:736b:211::2
sudo ipa dnsrecord-add 1.2.ff.es.eu.org ipa2 --aaaa-ip-address=2001:470:736b:211::3

sudo ipa dnsrecord-add 1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa. 2.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.1 --ptr-rec ipa1.1.2.ff.es.eu.org.

sudo ipa dnsrecord-add 1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa. 3.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.1 --ptr-rec ipa2.1.2.ff.es.eu.org.

sudo systemctl restart named

#### En ipa2112

sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan211

sudo dnf -y install freeipa-client

sudo dnf clean all

sudo hostnamectl set-hostname ipa2.1.2.ff.es.eu.org

sudo reboot

sudo nmcli connection modify vlan211 ipv6.dns ""
sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:211::2"
sudo nmcli connection up vlan211

//configurarlo como cliente
sudo ipa-client-install --server=ipa1.1.2.ff.es.eu.org --domain=1.2.ff.es.eu.org --realm=1.2.FF.ES.EU.ORG
-> yes
-> no o enter
-> yes 
-> admin
-> contraseña admin

sudo kinit admin
sudo ipa user-add usuarioPruebas --first Prueba1 --last Usuario --password
sudo klist

//pruebas desde ipa2111
	sudo ipa hostgroup-add-member ipaservers --hosts ipa2.1.2.ff.es.eu.org
	sudo firewall-cmd --add-service=freeipa-replication
	sudo firewall-cmd --runtime-to-permanent
//fin pruebas ipa2111

sudo firewall-cmd --add-service={freeipa-ldap,freeipa-ldaps,dns,ntp,freeipa-replication}
sudo firewall-cmd --runtime-to-permanent
sudo dnf -y install freeipa-server freeipa-server-dns
sudo dnf clean all

	sudo ipa-replica-install --setup-ca --setup-dns --no-forwarders

sudo kinit admin
sudo klist
sudo ipa user-find

Pruebas realizadas
sudo ipa-replica-manage list //en ambas maquinas ipa para verificar que ambas replicas estan conectadas

### Hacer los clientes
#### En ipa2111
//añadir cliente1 directo
sudo ipa dnsrecord-add 1.2.ff.es.eu.org cliente1 --aaaa-ip-address=2001:470:736b:212:5054:ff:fe02:1205

//añadir cliente2 directo
sudo ipa dnsrecord-add 1.2.ff.es.eu.org cliente2 --aaaa-ip-address=2001:470:736b:212:5054:ff:fe02:1206

//añadir cliente1 inverso
sudo ipa dnsrecord-add 1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa. 5.0.2.1.2.0.e.f.f.f.0.0.4.5.0.5.2 --ptr-rec cliente1.1.2.ff.es.eu.org.

//añadir cliente2 inverso
sudo ipa dnsrecord-add 1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa. 6.0.2.1.2.0.e.f.f.f.0.0.4.5.0.5.2 --ptr-rec cliente2.1.2.ff.es.eu.org.


//por si nos equivocamos eliminar
sudo ipa dnsrecord-del 1.2.ff.es.eu.org cliente2
sudo ipa dnsrecord-del 1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa. 5.0.2.1.2.0.e.f.f.f.0.0.4.5.0.5.2

//probamos los cambios realizados haciendo restart en el unbound y comprobando con dig desde alguna maquina
dig AAAA cliente2.1.2.ff.es.eu.org
dig -x 2001:470:736b:212:5054:ff:fe02:1205

#### En cliente2121
sudo hostnamectl set-hostname cliente1.1.2.ff.es.eu.org

sudo nmcli connection modify vlan212 ipv6.dns ""
sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan212

sudo dnf install -y freeipa-client nfs-utils
sudo dnf clean all

//falta de tamaño por lo que lo aumentamos en los xml

sudo nmcli connection modify vlan212 ipv6.dns ""
sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:211::2"
sudo nmcli connection up vlan212

sudo ipa-client-install --server=ipa1.1.2.ff.es.eu.org --domain=1.2.ff.es.eu.org --realm=1.2.FF.ES.EU.ORG
-> yes
-> no o enter
-> yes
-> admin
-> contraseña admin

sudo nmcli connection modify vlan212 ipv6.dns ""
sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan212

//comprobaciones
sudo kinit admin  
sudo klist

#### En cliente2122
sudo hostnamectl set-hostname cliente2.1.2.ff.es.eu.org

sudo nmcli connection modify vlan212 ipv6.dns ""
sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan212

sudo dnf install -y freeipa-client nfs-utils
sudo dnf clean all

sudo nmcli connection modify vlan212 ipv6.dns ""
sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:211::2"
sudo nmcli connection up vlan212

sudo ipa-client-install --server=ipa1.1.2.ff.es.eu.org --domain=1.2.ff.es.eu.org --realm=1.2.FF.ES.EU.ORG
-> yes
-> no o enter
-> yes
-> admin
-> contraseña admin

sudo nmcli connection modify vlan212 ipv6.dns ""
sudo nmcli connection modify vlan212 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan212

//comprobaciones
sudo kinit admin  
sudo klist

### Cliente en nfs1
#### En ipa2111
//añadimos el dominio del nfs2111
sudo ipa dnsrecord-add 1.2.ff.es.eu.org nfs1 --aaaa-ip-address=2001:470:736b:211::4
sudo ipa dnsrecord-add 1.2.0.b.6.3.7.0.7.4.0.1.0.0.2.ip6.arpa. 4.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.1 --ptr-rec nfs1.1.2.ff.es.eu.org.

//probamos reiniciando el unbound
dig AAAA nfs1.1.1.ff.es.eu.org
dig -x 2001:470:736b:211::4

#### En nfs2111
sudo nmcli connection modify vlan211 ipv6.dns ""
sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:2ff::2"
sudo nmcli connection up vlan211

sudo hostnamectl set-hostname nfs1.1.2.ff.es.eu.org

sudo reboot

sudo dnf install -y nfs-utils freeipa-client
sudo dnf clean all

sudo nmcli connection modify vlan211 ipv6.dns ""
sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:211::2"
sudo nmcli connection up vlan211

sudo ipa-client-install --server=ipa1.1.2.ff.es.eu.org --domain=1.2.ff.es.eu.org --realm=1.2.FF.ES.EU.ORG
-> yes
-> no o enter
-> yes
-> admin
-> contraseña admin

sudo nmcli connection modify vlan211 ipv6.dns ""
sudo nmcli connection modify vlan211 ipv6.dns "2001:470:736b:211::2"
sudo nmcli connection up vlan211

### Automontado

#### En nsf2111
//descomentamos las siguientes lineas en el fichero /etc/nfs.conf
```
[nfs]
vers3=y
vers4=y
vers4.0=y
vers4.1=y
vers4.2=y
```

sudo systemctl daemon-reload
sudo systemctl restart nfs-mountd

sudo firewall-cmd --permanent --add-service nfs
sudo firewall-cmd --reload

sudo systemctl enable --now nfs-server
sudo cat /proc/fs/nfsd/versions

#### En ipa2111

//creamos un nuevo ticket
sudo kdestroy
sudo kinit admin

sudo ipa service-add nfs/nfs1.1.2.ff.es.eu.org
sudo ipa service-add nfs/cliente1.1.2.ff.es.eu.org
sudo ipa service-add nfs/cliente2.1.2.ff.es.eu.org

sudo ipa user-add prueba2 --first prueba --last usuario --password

#### En nfs2111

//creamos un nuevo ticket
sudo kdestroy
sudo kinit admin

sudo ipa-getkeytab -s ipa1.1.2.ff.es.eu.org -p nfs/nfs1.1.2.ff.es.eu.org -k /etc/krb5.keytab
sudo klist -k /etc/krb5.keytab

sudo mkdir -p /export/home
sudo chmod 777 /export/home

// sudo mkdir -p /export/home/prueba1
// sudo chown usuarioPruebas:usuarioPruebas /export/home/prueba1
// sudo chmod 777 /export/home/prueba1

sudo mkdir -p /export/home/prueba2
sudo chown prueba2:prueba2 /export/home/prueba2
sudo chmod 777 /export/home/prueba2

//modificar el fichero /etc/exports con 
```
/export/home *(rw,sec=krb5:krb5i:krb5p,sync)
```

sudo systemctl restart --now nfs-server
sudo exportfs -rv
sudo showmount -e nfs1.1.2.ff.es.eu.org

#### En cliente2121
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload
sudo systemctl restart firewalld

sudo kdestroy
sudo kinit prueba2
sudo klist

sudo showmount -e nfs1.1.2.ff.es.eu.org

sudo mkdir -p /mnt
sudo chmod 777 /mnt

sudo mount -t nfs4 -o sec=krb5p nfs1.1.2.ff.es.eu.org:/export/home /mnt
cd /mnt/prueba2

#### En ipa2111

sudo ipa automountmap-add default auto.home

sudo ipa automountkey-del default auto.home --key="/home"

sudo ipa automountkey-add default auto.home --key='*' --info='-fstype=nfs4,rw,sec=krb5p nfs1.1.2.ff.es.eu.org:/export/home/&'

sudo ipa automountkey-add default auto.master --key=/home --info=auto.home

#### En cliente2122

sudo kdestroy
sudo kinit admin

sudo ipa-getkeytab -s ipa1.1.2.ff.es.eu.org -p nfs/cliente2.1.2.ff.es.eu.org -k /etc/krb5.keytab

### Pruebas automontaje
#### En cliente1
ssh  prueba2@cliente2.1.2.ff.es.eu.org
touch fichero_pruebas.txt
//añadir contenido al fichero

#### En nfs2111
sudo ls /exports/home/prueba2
sudo cat /export/home/prueba2/fichero_pruebas.txt

#### De nuevo en cliente1
ls /home
su prueba2 
cd home/prueba2
exit
ls /home

Ahora aparece prueba2
