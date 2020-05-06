# Instalacion-DockerSwarm
Implementar un cluster de alta disponibilidad con Docker Swarm
Creación del Clúster 
Paso 1
Creamos cinco maquinar virtuales con las siguientes especificaciones:
 
Nos servirán para los Masters 01, 02, 03 y los Workers 01 y 02.
Paso 2.
Establecemos el hostname para cada máquina virtual
-	hostnamectl set-hostname master01
-	hostnamectl set-hostname master02
-	hostnamectl set-hostname master03
-	hostnamectl set-hostname worker01
-	hostnamectl set-hostname master02
Paso 3
Al momento de crear las maquinas virtuales se realiza un cambio en el archivo /etc/sysconfig/network-scripts/ifcfg-enp0s3 este puede realizarse con un nano para cambiar ONBOOT a yes, para tener conexión a internet.
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=3c21a88b-fbf3-419e-a4d5-495d0ec72c7f
DEVICE=enp0s3
ONBOOT=yes
Paso no. 4
Instalamos los paquetes necesarios para conectividad a cada máquina virtual:
-	Net-tools para funciones red.
-	Htop para poder verificar los estados de las maquinas.
Paso no. 5
Agregamos las funcionalidades de yum a cada máquina virtual.
-	sudo yum install -y yum-utils device-mapper-persistent-data lvm2
Paso no. 6
Agregamos el repositorio de Docker a cada máquina virtual.
-	sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Paso no. 7
Instalamos Docker en cada máquina virtual y lo habilitamos.
-	sudo yum install -y docker-ce
-	systemctl enable docker
-	systemctl start dock
Paso no. 8
Habilitamos los puertos necesarios para poder acceder por medio de Docker Swarm a cada máquina virtual.
-	sudo firewall-cmd --add-port=2376/tcp --permanent
-	sudo firewall-cmd --add-port=2377/tcp –permanent
-	 sudo firewall-cmd --add-port=7946/tcp --permanent
-	 sudo firewall-cmd --add-port=7946/udp --permanent
-	 sudo firewall-cmd --add-port=4789/udp –permanent
Paso no. 9
Reiniciamos el firewall en cada máquina virtual:
-	systemctl restart firewalld
Paso no. 10
Iniciamos el clúster de Docker Swarm en nuestro master01
-	docker swarm init --advertise-addr 192.168.0.9
Paso no. 11
Solicitar los tokens de seguridad para añadir los otros nodos al clúster
-	docker swarm join-token manager para los Master 02 y 03
-	docker swarm join-token para los worker 01 y 02
Paso no. 12
Verificamos que todos nuestros nodos hayan sido añadidos al clúster
-	docker node ls

Creación del Dashboard
Paso no. 1
Generar un archivo yml con docker compose con el siguiente contenido 
# compose.yml
version: "3"

services:
  dashboard:
    image: charypar/swarm-dashboard //imagen que utilizaremos para el dashboard
    volumes:
    - "/var/run/docker.sock:/var/run/docker.sock" // compartiremos los volumenes
    ports:
    - 8081:8080 //exponemos los puertos
    environment:
      PORT: 8080 //puerto en donde se creará la variable de entorno
    deploy:
      replicas: 1 //número de replicas que queremos del dashboard
      placement:
        constraints:
          - node.role == manager //se utilizara un master para establecer el Dashboard


Paso no. 2
Levantamos el servicio del Dashboard con la siguiente línea:
-	docker stack deploy -c compose.yml svc
 
Creación de un servicio para los nodos
Ejecutamos un servicio básico con el que podemos crear varias replicas que serán distribuidas y balanceadas por el servicio de Docker Swarm
-	docker service create --replicas 1 --name helloworld -p 8080:8080 alpine ping docker.com
 

