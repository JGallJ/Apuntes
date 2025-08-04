## Etapa 1 : Preparación y despliegue de una VM Openstack

#### Preparación previa a la instalación de Devstack

Modificar el fichero *debian12controlador/cloud-config.yaml* con la clave publica
```
hostname: controlador
users:
- name: stack
ssh_pwauth: false
ssh_authorized_keys:
- ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIM50yjHFHa5Y5czPDV97yehmG1cZzhNiuR6ycIwx6/pj jogue@jogue
sudo: ['ALL=(ALL) NOPASSWD:ALL']
shell: /bin/bash
homedir: /opt/stack
```

modificar el valor 149 por W=2 (en mi caso) para los ficheros:
*debian12controlador/local.conf*
```
[[local|localrc]]
HOST_IP=192.168.2.11
FIXED_RANGE=10.4.128.0/20
FLOATING_RANGE=192.168.2.128/26
LOGFILE=/opt/stack/logs/stack.sh.log
ADMIN_PASSWORD=labstack
DATABASE_PASSWORD=supersecret
RABBIT_PASSWORD=supersecret
SERVICE_PASSWORD=supersecret
```

*debian12controlador/lvariables.tf*
```
variable "host_ip" {
  type = string
  default = "192.168.2.11"
}

variable "private_key" {
  type = string
  default = "/home/jogue/.ssh/id_ed25519"
}
```

*redlibvirt/main.tf*
```
terraform {
  required_providers {
    libvirt = {
      source = "dmacvicar/libvirt"
    }
    cloudinit = {
      source ="hashicorp/cloudinit"
    }
  }
}

provider "libvirt" {
  ## Configuration options
  uri = "qemu:///system"
  #alias = "server2"
  #uri   = "qemu+ssh://root@192.168.100.10/system"
}


resource "libvirt_network" "openstack_networkW" {
  name = "openstackW"
  mode = "nat"
  addresses = ["192.168.2.0/24"]
}
```

*debian12controlador/libvirt.tf*
```
# Define own storage pool
resource "libvirt_pool" "tofu_pool" {
  name = "tofu_pool"
  type = "dir"
  target {
    # modificar con PATH /misc/alumnos/as2/as22024/aXXXXXX/images/
    # creado previamente ?
	path = "/home/jogue/Escritorio/as-2/images/"
  }
}

# Defining VM Volume
resource "libvirt_volume" "dev_base_devstack" {
  name = "dev_base_devstack"
  pool = libvirt_pool.tofu_pool.name # List storage pools using virsh pool-list
  source = "/home/jogue/Escritorio/as-2/ProyectoAS2Parte1/debian_base_devstack_2025.1.qcow2
  #source = "/home/tmp/images/WithCloudinit/ubuntu22.04-base.qcow2"
  format = "qcow2"
}

...

network_interface {
	network_name   = "openstack2" # List networks with virsh net-list
	hostname       = "controlador"
	addresses      = [ "${var.host_ip}" ] # var.host
	mac            = "52:54:00:ff:ab:cd"
	wait_for_lease = false
}
```

damos permisos a debian_base_devstack_2025.1.qcow2
```
chmod g+w debian_base_devstack_2025.1.qcow2
```

#### Una vez completada la instalación
Nos metemos en el directorio *redlibvirt*
dentro de este
```
../../tofu init

../../tofu apply -auto-approve
```

Nos metemos en el directorio *debian12controlador*
dentro de este
```
../../tofu init

../../tofu apply -auto-approve
```

se produce un error
```
│ Error: error creating libvirt domain: Cannot access storage file '/home/jogue/Escritorio/as-2/images/server-disk' (as uid:64055, gid:993): Permiso denegado
│ 
│   with libvirt_domain.controlador,
│   on libvirt.tf line 46, in resource "libvirt_domain" "controlador":
│   46: resource "libvirt_domain" "controlador" {
│ 
```
que para solucionarlo necesitamos ejecutar lo siguiente

```
sudo vi /etc/apparmor.d/libvirt/TEMPLATE.qemu

sudo apparmor_parser -r /etc/apparmor.d/libvirt/TEMPLATE.qemu
sudo systemctl reload apparmor

sudo systemctl restart libvirtd
```

tambien hacia falta dar permisos al /home/jogue
chmod 755 /home/jogue

accedemos al controlador mediante
```
ssh stack@192.168.2.11
```

metemos los usuarios en /etc/hosts

luego hacemos la instalación
```
cd ~/devstack
./stack.sh
```
IMPORTANTE GUARDAR LO SIGUIENTE
```
=========================
DevStack Component Timing
 (times are in seconds)  
=========================
wait_for_service      12
async_wait            46
osc                  125
apt-get              271
test_with_retry        4
dbsync                 2
pip_install          187
apt-get-update         2
run_process           28
git_timed            216
-------------------------
Unaccounted time     248
=========================
Total runtime        1141

=================
 Async summary
=================
 Time spent in the background minus waits: 217 sec
 Elapsed time: 1141 sec
 Time if we did everything serially: 1358 sec
 Speedup:  1.19018


Post-stack database query stats:
+------------+-----------+-------+
| db         | op        | count |
+------------+-----------+-------+
| keystone   | SELECT    | 19947 |
| keystone   | INSERT    |    91 |
| keystone   | UPDATE    |     7 |
| neutron    | DESCRIBE  |     2 |
| neutron    | CREATE    |     1 |
| neutron    | SHOW      |     4 |
| neutron    | SELECT    |  5639 |
| neutron    | INSERT    |  4127 |
| neutron    | DELETE    |    32 |
| placement  | SELECT    |    19 |
| placement  | INSERT    |    62 |
| placement  | SET       |     1 |
| neutron    | UPDATE    |   244 |
| nova_api   | SELECT    |    50 |
| nova_cell0 | SELECT    |    28 |
| nova_cell1 | SELECT    |    60 |
| nova_cell0 | INSERT    |     6 |
| nova_cell0 | UPDATE    |     3 |
| nova_cell1 | INSERT    |     4 |
| cinder     | SELECT    |    55 |
| cinder     | INSERT    |     5 |
| cinder     | UPDATE    |     3 |
| nova_api   | INSERT    |    20 |
| nova_api   | SAVEPOINT |    10 |
| nova_api   | RELEASE   |    10 |
| glance     | INSERT    |    14 |
| glance     | SELECT    |    30 |
| glance     | UPDATE    |     2 |
| placement  | UPDATE    |     3 |
| nova_cell1 | UPDATE    |    13 |
| cinder     | DELETE    |     1 |
+------------+-----------+-------+



This is your host IP address: 192.168.2.11
This is your host IPv6 address: ::1
Horizon is now available at http://192.168.2.11/dashboard
Keystone is serving at http://192.168.2.11/identity/
The default users are: admin and demo
The password: labstack

Services are running under systemd unit files.
For more information see: 
https://docs.openstack.org/devstack/latest/systemd.html

DevStack Version: 2025.1
Change: 030324a0a5f405be40728dc08e9c96fba5c1310b Cap stable/2025.1 network, swift, volume api_extensions for tempest 2025-03-21 11:18:45 -0700
OS Version: Debian 12 bookworm
```

para encender y apagar la maquina usaremos
```
virsh start controlador
sudo shutdown -h now
```

## Etapa 2 : Utilización de Openstack y Opentofu

Nos metemos a la maquina de la siguiente manera
```
http://192.168.2.11/dashboard
```

Una vez dentro vamos a poner en marcha una instancia "cirros"
Nos metemos en Proyecto -> Computación -> Instancias -> Lanzar instancia
- Detalles
	- le asignamos un nombre
- Origen
	- escogemos la imagen cirros ya disponible
- Sabor
	- escogemos uno de los sabores que hay, en este caso ge cogido el m1.tiny
- Redes
	- escogemos la red, en este caso he cogido privada

Robado de emilliano:
	Con demo:
		- Creación de instancia desde imagen cirros
		- Necesidad de crear router propio (router-demo) para usar IP flotante
		- Acceso limitado a recursos
	Con admin:
		- Acceso a todos los proyectos
		- Visibilidad total de routers, redes, instancias
		- Creación de recursos compartidos (como redes externas)
		- Capacidad de revisar/quitar recursos en todos los proyecto

Una vez probado todo lo que tiene openstack (o mirado) procedemos a crear una nueva imagen que cargue la imagen ubuntu18.04

Subimos la imagen a openstack con el usuario **admin**
Computación -> Imagenes -> Crear imagen
Hay le damos las siguientes especificaciones
- Nombre: ubuntu18.04
- Formato: QCOW2
- Fuente: ubuntu-bionic etc etc
- DIsco minimo de 3GB
- Visibilidad: Publico

Creamos un nuevo flavour con usuario **admin**
Administrador -> Computación -> Sabores -> Crear sabor
Especificaciones (se pone 3GB de disco por que sino pasan cosas raras)
1. Nombre: 2Gcon256M
2. VCPUs: 1
3. RAM: 256 MB
4. Disco raíz: 3 GB
5. Disco efímero: 0 GB

Ahora modificaremos los ficheros de la carpeta ***openstackvmconipflotante***
*openstackvmconipflotante/main.tf*
```
terraform {
  required_providers {
    openstack = {
      source = "terraform-provider-openstack/openstack"
      version = "3.0.0"
      # version = "~> 3.0.0"
    }
  }
}

provider "openstack" {
  user_name   = "demo"
  tenant_name = "demo"
  password    = "labstack"
  #auth_url    = "http://192.168.2.11/v3"
  auth_url    = "http://192.168.2.11/identity/"
  region      = "RegionOne"
}
```

*openstackvmconipflotante/variables.tf*
```
variable "image" {
  default = "ubuntu18.04"
}

variable "flavor" {
  default = "2Gcon256M"
}

variable "ssh_key_file" {
  default = "~/.ssh/id_ed25519"
}

variable "ssh_user_name" {
  default = "ubuntu"
}

variable "pool" {
  default = "public"
}

```

Copiamos la carpeta del proyecto dentro del controlador con usuario stack y nos metemos dentro
```
scp -r ProyectoAS2Parte1 stack@192.168.2.11:~
```

Posteriormente nos metemos a la carpeta openstackvmconipflotante y hacemos lo siguiente
```
../../tofu init
../../tofu apply -auto-approve
```

Ocurre un error con la clave pública ed25519 asi que la creamos y volvemos a ejecutar lo de antes
```
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
```

El output es el siguiente
address = "192.168.2.185"

Copiado de emilliano
- Si te fijas para permitir la comunicación, br-ex (el bridge de OpenStack) debe tener una IP en la misma subred que las instancias de OpenStack. Si haces ip a en controlador lo verás que br-ex está vacío. Sin una IP en br-ex, no hay ruta entre las redes. OpenStack espera que br-ex sea la interfaz que enrute el tráfico entre el mundo exterior y sus instancias. La IP de la puerta de enlace (192.168.31.129) es el punto de conexión obligatorio para que las instancias de OpenStack (192.168.31.130-190) se comuniquen con redes externas (incluyendo tu VM controlador en libvirt).

Para solucionar esto necesitamos añadir la IP de la puerta de enlace al br-ex, de la siguiente manera
```
sudo ip addr add 192.168.2.129/26 dev br-ex
sudo ip link set br-ex up
```

## Comandos terminal

	Cargar todas las variables de entorno necesarias antes para trabajar mediante linea de comandos
```
source /opt/stack/devstack/openrc
```

