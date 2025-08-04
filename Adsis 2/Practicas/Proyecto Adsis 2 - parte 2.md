## Etapa 1 : Diseño y puesta en marcha de un cluster Kubernetes mediante Opentofu

Eliminar todo lo anterior
```
// eliminar la anterior pool 
virsh pool-undefine tofu_pool

// eliminar el commoninit de la carpeta images dnd estaba definada de la parte 1
virsh vol-delete --pool tofu_pool_master commoninit.iso

//eliminar todo lo de images/
sudo rm images/...
```

Primero, intentar poner lo ficheros .tf en orden

### Para master

fichero **master/variables.tf**
```
variable "host_ip" {
  type = string
  default = "192.168.2.6" # IP de la maquina master o host
}

variable "private_key" {
  type = string
  default = "/home/jogue/.ssh/id_ed25519"
}
```

fichero **master/libvirt.tf**
```
# Define own storage pool
resource "libvirt_pool" "tofu_pool_master" {
  name = "tofu_pool_master"
  type = "dir"
  target {
  # definimos el path al directorio de imagenes no definido de base
	path = "/home/jogue/Escritorio/as-2/images/"
  }
}

# Defining VM Volume
# Crea un volumen en el pool de tofu
resource "libvirt_volume" "imagen_ubuntu" {
  name = "imagen_ubuntu"
  pool = libvirt_pool.tofu_pool_master.name # List storage pools using virsh pool-list
  source = "/home/jogue/Escritorio/as-2/ProyectoAS2Parte2/ubuntu-bionic-server-cloudimg-amd64.img"
  #source = "/home/tmp/images/WithCloudinit/ubuntu22.04-base.qcow2"
  format = "qcow2"
}

# Crea un disco adicional para la VM basado en el volumen base
# Cometido: no modificar la imagen base. Cada VM tiene su disco independiente
resource "libvirt_volume" "master_disk" {
  name           = "master_disk"
  size           = 20000000000
  pool           = libvirt_pool.tofu_pool_master.name # Asociamos con el pool de antes
  base_volume_id = libvirt_volume.imagen_ubuntu.id
}

# Cargamos el archivo cloud-config.yaml para ser usado en la VM
data "cloudinit_config" "user_data" {
  gzip = false
  base64_encode = false

  part {
    filename = "cloud-config.yaml"
    content_type = "text/cloud-config"
    content = file("${path.module}/cloud-config.yaml")
  }
}

# Crea un iso con la configuración de cloud-init
resource "libvirt_cloudinit_disk" "commoninit" {
  name           = "commoninit.iso"
  pool           = libvirt_pool.tofu_pool_master.name # List storage pools with virsh pool-list
  user_data      = "${data.cloudinit_config.user_data.rendered}"
}

# Define KVM domain to create
resource "libvirt_domain" "master" {
  name   = "master"
  memory = "1024" # 1 GB de RAM
  vcpu   = 1
  # qemu_agent = true

  boot_device {
    dev = [ "hd" ]
  }

  # Copiamos el fichero local.conf en la VM en el directorio especificado
  provisioner "file" {
    source      = "k3s"
    destination = "/tmp/k3s" # Primero en tmp, se puede en otro lado?

    connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = "${var.host_ip}"
      private_key = "${file("${var.private_key}")}" # host key validation disabled by default
    }
  }

  # Copiamos el fichero local.sh en la VM en el directorio especificado
  provisioner "file" {
    source      = "install.sh"
    destination = "/home/ubuntu/install.sh"

    connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = "${var.host_ip}"
      private_key = "${file("${var.private_key}")}" # host key validation disabled by default
    }
  }

  provisioner "remote-exec" {
    inline = [
      "sudo mv /tmp/k3s /usr/local/bin/k3s",
      # Dar permisos de ejecucion
      "sudo chmod +x /usr/local/bin/k3s",
      "chmod +x /home/ubuntu/install.sh",
      # Nombre master
      "sudo hostnamectl set-hostname master",
      # Aniadir al fichero hosts 
      "echo '192.168.2.6 master' | sudo tee -a /etc/hosts",
      "echo '192.168.2.7 worker1' | sudo tee -a /etc/hosts",
      "echo '192.168.2.8 worker2' | sudo tee -a /etc/hosts",
      "echo '192.168.2.9 worker3' | sudo tee -a /etc/hosts",
      # Instalacion master kubernetes k3s
      "INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh server \\",
     "--token \"wCdC16AIP8qpqq153DM6ujtrfZ7qsEM7PHLxD+Sw+RNK2d1oDJQQOsBkIwy5OZ/5\" \\",
     "--flannel-iface ens3 \\",
     "--bind-address ${var.host_ip} \\",
     "--node-ip ${var.host_ip} --node-name master \\",
     "--disable traefik \\",
     "--disable servicelb \\",
     "--node-taint k3s-controlplane=true:NoExecute"
    ]

    connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = "${var.host_ip}"
      private_key = "${file("${var.private_key}")}" # host key validation disabled by default
    }
  }

  provisioner "local-exec" {
    command = <<EOT
    # Copiar kubectl (no necesita sudo)
     scp -i ${var.private_key} \
         -o StrictHostKeyChecking=no \
         ubuntu@${var.host_ip}:/usr/local/bin/kubectl ./kubectl && \
     chmod +x ./kubectl && \
     sudo mv ./kubectl /usr/local/bin/


     # Configurar kubeconfig (usando sudo para copiar)
     mkdir -p ~/.kube && \
     ssh -i ${var.private_key} ubuntu@${var.host_ip} "sudo cat /etc/rancher/k3s/k3s.yaml" > ~/.kube/config && \
     sed -i.bak "s/127.0.0.1/${var.host_ip}/" ~/.kube/config && \
     chmod 600 ~/.kube/config


     # Verificar instalación
     export KUBECONFIG=~/.kube/config
     kubectl cluster-info


    EOT
  }

  cpu {
    mode = "host-passthrough" # Utiliza las capacidades del procesador fisico
  }

  network_interface {
    network_name   = "openstack2" # List networks with virsh net-list
    hostname       = "master"
    addresses      = [ "${var.host_ip}" ] # var.host
    mac            = "52:54:00:ff:ab:cd"
    wait_for_lease = false
  }

  disk {
    volume_id = "${libvirt_volume.master_disk.id}"
  }

  cloudinit = "${libvirt_cloudinit_disk.commoninit.id}"

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}

# Output Server IP
#output "ip" {
#  value = "${libvirt_domain.controladorceph.0.addresses.0}"
#}
```

hacer lo siguiente antes de tofu apply
```
mkdir -p ~/.kube
// Listar los workers y master con IPs y macs
virsh net-dhcp-leases openstack2
```

Hacemos esto para el master
```
../tofu destroy -auto-approve
../tofu init
../tofu apply -auto-approve
```

Para solucionar el error de los known hosts
```
ssh-keygen -f ~/.ssh/known_hosts -R '192.168.2.6'
```

https://192.168.2.6:6443

### Para los workers

fichero **workers/variables.tf**
```
variable "workers" {
  default = [
    {
      name      = "worker1"
      ip        = "192.168.2.7"
      mac       = "52:54:00:ff:ab:ce"
      ram       = 1639
      cpus      = 1
      disk_size = 20000000000
      extra_disk_size = 30000000000
    },
    {
      name      = "worker2"
      ip        = "192.168.2.8"
      mac       = "52:54:00:ff:ab:cf"
      ram       = 1639
      cpus      = 1
      disk_size = 20000000000
      extra_disk_size = 30000000000
    },
    {
      name      = "worker3"
      ip        = "192.168.2.9"
      mac       = "52:54:00:ff:ab:d0"
      ram       = 1639
      cpus      = 1
      disk_size = 20000000000
      extra_disk_size = 30000000000
    }
  ]
}

variable "private_key" {
  type = string
  default = "/home/jogue/.ssh/id_ed25519"
}
```

fichero **workers/libvirt.tf**
```
# Defining VM Volume
# Crea un volumen en el pool de tofu
resource "libvirt_volume" "imagen_ubuntu" {
  name = "imagen_ubuntu"
  pool = "tofu_pool_master" # List storage pools using virsh pool-list
  source = "/home/jogue/Escritorio/as-2/ProyectoAS2Parte2/ubuntu-bionic-server-cloudimg-amd64.img"
  #source = "/home/tmp/images/WithCloudinit/ubuntu22.04-base.qcow2"
  format = "qcow2"
}

# Crea un disco adicional para la VM basado en el volumen base
# Cometido: no modificar la imagen base. Cada VM tiene su disco independiente
resource "libvirt_volume" "worker_disk" {
  count          = length(var.workers)
  name           = "${var.workers[count.index].name}_disk"
  size           = var.workers[count.index].disk_size
  pool           = "tofu_pool_master" # Asociamos con el pool de antes
  base_volume_id = libvirt_volume.imagen_ubuntu.id
}

resource "libvirt_volume" "worker_extra_disk" {
  count  = length(var.workers)
  name   = "${var.workers[count.index].name}_extra_disk"
  size   = var.workers[count.index].extra_disk_size
  pool   = "tofu_pool_master"
}

# Cargamos el archivo cloud-config.yaml para ser usado en la VM
data "cloudinit_config" "user_data" {
  gzip = false
  base64_encode = false

  part {
    filename = "cloud-config.yaml"
    content_type = "text/cloud-config"
    content = file("${path.module}/cloud-config.yaml")
  }
}

# Crea un iso con la configuración de cloud-init
resource "libvirt_cloudinit_disk" "commoninit" {
  name           = "commoninit.iso"
  pool           = "tofu_pool_master" # List storage pools with virsh pool-list
  user_data      = "${data.cloudinit_config.user_data.rendered}"
}

# Define KVM domain to create
resource "libvirt_domain" "workers" {
  count  = length(var.workers)
  name   = var.workers[count.index].name
  memory = var.workers[count.index].ram
  vcpu   = var.workers[count.index].cpus
  # qemu_agent = true

  boot_device {
    dev = [ "hd" ]
  }

  # Copiamos el fichero local.conf en la VM en el directorio especificado
  provisioner "file" {
    source      = "k3s"
    destination = "/tmp/k3s"

    connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = var.workers[count.index].ip
      private_key = "${file("${var.private_key}")}" # host key validation disabled by default
    }
  }

  provisioner "remote-exec" {
    inline = [
      "sudo mv /tmp/k3s /usr/local/bin/k3s",
      "chmod +x /usr/local/bin/k3s",
    ]

    connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = var.workers[count.index].ip
      private_key = "${file("${var.private_key}")}" # host key validation disabled by default
    }
  }

  # Copiamos el fichero local.sh en la VM en el directorio especificado
  provisioner "file" {
    source      = "install.sh"
    destination = "/home/ubuntu/install.sh"

    connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = var.workers[count.index].ip
      private_key = "${file("${var.private_key}")}" # host key validation disabled by default
    }
  }

  provisioner "remote-exec" {
    inline = [
      # Dar permisos de ejecucion
      "chmod +x /home/ubuntu/install.sh",
      # Nombre master
      "sudo hostnamectl set-hostname ${var.workers[count.index].name}",
      # Aniadir al fichero hosts 
      "echo '192.168.2.6 master' | sudo tee -a /etc/hosts",
      "echo '192.168.2.7 worker1' | sudo tee -a /etc/hosts",
      "echo '192.168.2.8 worker2' | sudo tee -a /etc/hosts",
      "echo '192.168.2.9 worker3' | sudo tee -a /etc/hosts",
      # Instalacion master kubernetes k3s
      "INSTALL_K3S_SKIP_DOWNLOAD=true \\", 
      "./install.sh agent --server https://192.168.2.6:6443 --token \"wCdC16AIP8qpqq153DM6ujtrfZ7qsEM7PHLxD+Sw+RNK2d1oDJQQOsBkIwy5OZ/5\" \\",
      "--flannel-iface ens3 \\",
      "--node-ip ${var.workers[count.index].ip} --node-name ${var.workers[count.index].name} ",
    ]

    connection {
      type     = "ssh"
      user     = "ubuntu"
      host     = var.workers[count.index].ip
      private_key = "${file("${var.private_key}")}" # host key validation disabled by default
    }
  }

  cpu {
    mode = "host-passthrough" # Utiliza las capacidades del procesador fisico
  }

  network_interface {
    network_name   = "openstack2" # List networks with virsh net-list
    hostname       = "${var.workers[count.index].name}"
    addresses      = [ "${var.workers[count.index].ip}" ]
    mac            = "${var.workers[count.index].mac}"
    wait_for_lease = false
  }

  disk {
    volume_id = libvirt_volume.worker_disk[count.index].id
  }

  disk {
    volume_id = libvirt_volume.worker_extra_disk[count.index].id
  }

  cloudinit = "${libvirt_cloudinit_disk.commoninit.id}"

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}

# Output Server IP
#output "ip" {
#  value = "${libvirt_domain.controladorceph.0.addresses.0}"
#}
```

## Etapa 2 : Análisis y comprehensión de despliegue de una aplicación con Ceph

Descargamos el zip mediante
```
wget [https://github.com/rook/rook/archive/refs/tags/v1.8.10.tar.gz](https://github.com/rook/rook/archive/refs/tags/v1.8.10.tar.gz)
```

Y extraemos los ficheros de la carpeta *rook/deply/examples* que concuerden con los de  *aplicacionesCephRook/ceph/* y los metemos en la ultima

luego hacemos
```
kubectl create -f aplicacionesCephRook/ceph/crds.yaml
kubectl create -f aplicacionesCephRook/ceph/common.yaml
kubectl create -f aplicacionesCephRook/ceph/operator.yaml

//con esto comprobamos que esten ya listos
kubectl -n rook-ceph get pod
```

Esperamos a que esten todos en running (tarda 2-3 min aprox)

Una vez este running hacemos lo siguiente 
```
kubectl create -f aplicacionesCephRook/ceph/cluster.yaml

//con esto comprobamos que esten ya listos
kubectl -n rook-ceph get pod
```

Este tarda bastante mas (6-7 min aprox)

Ahora ejecutamos lo siguiente para comprobar el estado de funcionamiento de Ceph a traves del despliegue de un pod de herramientas toolbox
```
kubectl -n rook-ceph create -f aplicacionesCephRook/ceph/toolbox.yaml

// comprobar funcionamiento con 
kubectl -n rook-ceph get pod -l "app=rook-ceph-tools"
```

Luego acceder a la caja de herramientas Ceph
```
kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -- bash
```
Dentro podemos ejecutar comandos como
```
ceph status
ceph df
rados df
```


### Aplicación web, almacenamiento distribuidos de dispositivos bloques y de sistema de ficheros

Ya que tenemos a Ceph el administrador del Cluster vamos a poner en marcha un pool de bloques en Ceph (nivel de replicación 3) mediante un recurso Kubernetes
personalizado CephBlockPool, y un StorageClass asociado a este pool de bloques mediante el aprovisionador ligado al operador rook-ceph puesto en marcha en la etapa inicial
```
kubectl apply -f aplicacionesCephRook/ceph/storageclassRbdBlock.yaml
```

Ahora editaremos los yaml *mysql.yaml* y *wordpress.yaml*
**mysql.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: rook-ceph-block # el que ya tenemos desplegado mediante el storageclassRbdBlock.yaml
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
    tier: mysql
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: changeme
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage # Conectamos el volumen definido mas abajo con el punto de montaje del contenedor
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim # Enlazamos el volumen pvc que gestionará la persistencia de datos
```

**wordpress.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
  - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: rook-cephfs # si solo un pod -> rook-ceph-block, varios pods mismo volumen -> rook-cephfs
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
    tier: frontend
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.6.1-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          value: changeme
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim # Coinciendo con el nombre del pvc definido antes
```

Aplicamos los ficheros para poner en funcionamiento la aplicacion de wordpress
```
kubectl apply -f aplicacionesCephRook/mysql.yaml
//Con lo siguiente comprobamos que se haga puesto en running el pod
kubectl get pods

kubectl apply -f aplicacionesCephRook/wordpress.yaml
// lo mismo de antes, esperar
```

Luego de tener esto, comprobamos que podemos acceder al servicio wordpress desplegado mediante wget
```
kubectl exec -ti wordpress-b98c66fff-f5zzb /bin/bash

// Ya dentro del pod
curl -v http://wordpress
```

```
// Para ver donde se ubica cada servicio
kubectl get pod -o=wide
// Para ver el puerto donde se puede acceder al servicio
kubectl get svc wordpress -o=wide
```

Mirando lo anterior podemos ver que el servicio se encuentra activo en el worker1 y que podemos acceder mediante la siguiente url local en mi caso

http://192.168.2.7:31507/wp-admin/

### Comprobación de la Utilización del Pool de Dispositivos de Bloques Ceph (RBD) Mediante un Pod de Creación y Uso Manual de Dispositivos de Bloque

Ejecutamos un nuevo pod
```
kubectl apply -f aplicacionesCephRook/ceph/direct-mount.yaml
// Comprobamos que este en funcionamiento
kubectl -n rook-ceph get pod -l app=rook-direct-mount

// Para meternos dentro del pod
kubectl -n rook-ceph exec -it rook-direct-mount-84fdf795b9-4ftnd bash
```

Una vez dentro podemos hacer cosas como
```
// Crear un dispositivo RBD
rbd create replicapool/test --size 10

// Ver informacion del RDB
rbd info replicapool/test

// Deshabilitar algunas características de RBD que no están soportadas por el kernel
rbd feature disable replicapool/test fast-diff deep-flatten object-map

// Mapear el dispositivo RBD
rbd map replicapool/test

// Ver los dispositivos mapeados
lsblk | grep rbd

// Formatear el volumen (SOLO HACERLO LA PRIMERA VEZ O SE PERDERAN DATOS)
mkfs.ext4 -m0 /dev/rbd1
```


Ahora vamos a crear un directorio y a montar el dispositivo en ese directorio
```
mkdir tmp/pruebas-montaje
mount /dev/rbd1  /tmp/pruebas-montaje

echo "Prueba de rook ceph" > /tmp/pruebas-montaje/texto-generico.txt
cat tmp/pruebas-montaje/texto-generico.txt
```

Ahora vamos a probar la toleracia de fallos, parando un nodo el cual no tenga ni al pod DNS ni el de metricas ni ceph-mgr. Luego comprobaremos que siga funcionando el fichero creado anteriormente
```
// Para comprobar las condiciones anteriores
kubectl get pods --all-namespaces -o wide
DNS se encuentra en worker2
metricas en worker2 as well
ceph-mgr en worker2 as well

// Decido apagar desde dentro el worker 1
ssh ubuntu@192.168.2.7
sudo shutdown -h now
```

Sistema de ficheros Ceph

```
kubectl apply -f aplicacionesCephRook/ceph/storageclassCephFS.yaml
kubectl apply -f aplicacionesCephRook/ceph/filesystem.yaml

mon_endpoints=$(grep mon_host /etc/ceph/ceph.conf | awk '{print $3}')
my_secret=$(grep key /etc/ceph/keyring | awk '{print $3}')

mkdir /tmp/file_system
mount -t ceph -o mds_namespace=myfs,name=admin,secret=$my_secret $mon_endpoints:/ /tmp/file_system

echo "Fichero pruebas anti montaje" > /tmp/file_system/fichero.txt
cat tmp/file_system/fichero.txt

// desmontamos y comprobamos que funciona entre montajes
cat tmp/file_system/fichero.txt
```


## Problemas encontrados
No se me conectaban los nodos worker con el master y al hacer ```kubectl get nodes``` estos no salian como nodos mientras las maquinas si que estaban activas y funcionando
Mucho rato perdido para un simple cambio de linea en un fichero

No funciona la tolerancia a fallos cuando se supone que tienen que migrar

Error con uno de los OSD
``` 
rook-ceph-osd-1-6dfc4c5447-cz4qv                    0/1     CrashLoopBackOff   26 (22s ago)   3d3h
```

Para solucionarlo:
```
// meterse en la toolbox de Rook-ceph
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash

// Comprobar que el OSD realmente esta fallando
ceph osd tree

// Eliminaer el OSD del CRUSH map y de Ceph
ceph osd crush remove osd.1
ceph auth del osd.1
ceph osd rm 1

// Borrar el pod roto y deja que Rook lo reprovisione
kubectl delete pod rook-ceph-osd-1-6dfc4c5447-cz4qv -n rook-ceph

```