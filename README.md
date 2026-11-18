# Lab Semana 12: Configuración de VMs

## Configuración inicial de VMs

1. Descargar la imagen ISO de Ubuntu Server v26.04.1

1. Crear una máquina virtual **base** en VirtualBox:

   - ISO: Ubuntu v26.04.1
   - RAM: 2 GB por nodo worker (*4 GB para nodo master*)
   - CPU: 2 núcleos por nodo
   - Almacenamiento: 20 GB Dinámicos
   - Adaptador 1: NAT (*predeterminado*)
   - Adaptador 2: Adaptador solo anfitrión (*Host-only*)

1. Instalar OpenSSH Server:

   ```bash
   sudo apt update && sudo apt install -y openssh-server
   sudo systemctl enable --now ssh
   ```

1. Habilitar sudo sin contrasenia para el usuario de Ansible
  
   ```bash
   echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
   sudo chmod 0440 /etc/sudoers.d/ansible
   ```

1. Añadir los *hostnames* en los nodos

   ```bash
   sudo vim /etc/hosts
   ```

   ```txt
   #127.0.1.1 hadoop-<name>
   192.168.56.10 hadoop-master
   192.168.56.11 hadoop-worker1
   192.168.56.12 hadoop-worker2
   192.168.56.13 hadoop-worker3
   ```

1. Configurar IPs estáticas

   - Editar el archivo de Configuración de red (editar despues para cada MV):

   ```bash
   # verificar el nombre del archivo
   ls /etc/netplan/
   # acceder a la edición del archivo
   sudo vim /etc/netplan/00-installer-config.yaml
   ```

   - Reemplaza el contenido del archivo por la siguiente configuración:

   ```yaml
   network:
     version: 2
     ethernets:
       enp0s3: # cambiar por el nombre de tu interfaz ('ip a')
         dhcp4: true
       enp0s8: # cambiar por el nombre de tu interfaz
         dhcp4: false
         addresses:
           - 192.168.56.10/24 # valor de la tabla (TODO: Modificar en cada MV)
   ```

   ```bash
   sudo netplan apply
   # verificar
   ip a
   ```

1. Clonar la maquina virtual base:

   | Hostname | IP estática |
   | --- | --- |
   | hadoop-master | 192.168.56.10 |
   | hadoop-worker1 | 192.168.56.11 |
   | hadoop-worker2 | 192.168.56.12 |
   | hadoop-worker3 | 192.168.56.13 |

   > Al clonar, debes marcar la opción "Generar nuevas direcciones MAC para todos los adaptadores de red".

   Al clonar la VM base, en cada una ejecutar:

   ```bash
   sudo hostnamectl set-hostname hadoop-master # en el master
   sudo hostnamectl set-hostname hadoop-worker1 # en los workers
   ```

1. Configurar acceso SSH por llave pública

   ```bash
   # Generar la llave SSH
   ssh-keygen -t ed25519 -C "ansible-hadoop"

   # Copiar la llave pública a cada nodo
   ssh-copy-id vboxuser@192.168.56.10
   ssh-copy-id vboxuser@192.168.56.11
   ssh-copy-id vboxuser@192.168.56.12
   ssh-copy-id vboxuser@192.168.56.13
   ```

1. Instalar ansible en tu maquina local (Ubuntu)

   ```bash
   sudo apt update
   sudo apt install software-properties-common
   sudo apt-add-repository --yes --update ppa:ansible/ansible
   sudo apt install ansible
   ```

1. Ejecutar ansible para configurar las MVs:

   ```bash
   ansible-playbook site.yml
   # verificar
   ansible all -m shell -a "sudo -u hadoop jps"
   ```
