Pruebas
===================

A continuación se describe de manera somera y sin correcciones posteriores el proceso llevado a cabo para configurar la topologia de test. El objetivo de estos ensayos es realizar el montaje empleando como medio de conexion switch virtual con driver ovs (open-vswitch) y no con driver bridge (como se hizo en los casos anteriormente descritos). Una vez se tenga correctamente la topologia funcional, el siguiente paso consiste en agregar de forma manual reglas al swich ovs para permitir la conectividad entre los containers.

> **Nota:**
> Aca el test solo se preocupa por la conectividad entre los diferentes elementos de la red. Todavia no se hace enfasís en el montaje del test definitivo.
--------------
#####  Descripción de la topologia
La siguiente figura muestra la topología a montar:

![Montaje 1](test-sin-controlador.png?raw=true "Experimento empleando un swith ovs usando el protocolo OpenFlow")


####  Caso 1: Topologia empleando driver ovs por medio de linea de comandos

La siguiente tabla detalla las caracteristicas de los containers que funcionan como hosts:

| Host     | Interfaz | IP   | Imagen |
| :-------: | ----: | :---: | :--: |
| h1 | ? |  10.0.0.1 | kalilinux/kali-linux-docker| 
| h2 | ? |  10.0.0.2 | ubuntu |

En lo que respecta al switch que conectará los containers la siguiente tabla define las caracteristicas:

| switch     | Driver | Uso de Openflow  | 
| :-------: | ----: | :---: |
| s1 | ovs |  No |

En este caso, el switch no necesitara del controlador para permitir la comunicación entre los containers por lo 
tanto, el funcionamiento de este ejemplo sera similar a los diferentes casos de test realizados en la prueba 1. La guia de los comandos 
aplicados para esta tarea se encuentra mas detalladamente en cualquiera de las siguientes 2 URL:
- http://containertutorials.com/network/ovs_docker.html
- https://developer.ibm.com/recipes/tutorials/using-ovs-bridge-for-docker-networking/

Asumiendo que ya se tiene instalado el ovs-docker, los comandos ejecutados para llevar a cabo el montaje de la topologia fueron
los siguientes:

```
# 0. Se arranca el openswitch (Se hace cuando el comando 1 no da)
sudo /etc/init.d/openvswitch-switch start
# 1. Se agrega el switch virtual y sus interfaces y se configura
sudo ovs-vsctl add-br s1
ifconfig s1 10.0.0.1 netmask 255.255.255.0 up

# 2. Se verifica que el switch haya sido creado
sudo ovs-vsctl list-br                      # Listando los switchs existentes
sudo ovs-vsctl show                         # Mostrando informacion resumida de los switches existentes
sudo ovs-vsctl list-ifaces s1               # Listando las interfaces del switch 

# 3. Se crean los contenedores
# Poner a correr las 2 imagenes en dos consolas diferentes si no se coloca en modo 
# detach (ejecucion en background) 
docker run --name h1 --hostname h1 --net=none --rm -ti kalilinux/kali-linux-docker /bin/bash 
docker run --name h2 --hostname h2 --net=none --rm -ti ubuntu /bin/bash 

# 4. Se verifica que los contanedores esten corriendo 
docker ps
docker inspect --format '{{ .NetworkSettings.IPAddress }}' h1
docker inspect --format '{{ .NetworkSettings.IPAddress }}' h2

# 4. Se conectan los contenedores al switch ovs
ovs-docker add-port s1 h1-eth0 h1 --ipaddress=10.0.0.2/8 
ovs-docker add-port s1 h2-eth0 h2 --ipaddress=10.0.0.3/8 

# 5. Se verifica nuevamente la informacion asociada al switch
sudo ovs-vsctl list-br                      # Listando los switchs existentes
sudo ovs-vsctl show                         # Mostrando informacion resumida de los switches existentes
sudo ovs-vsctl list-ifaces s1               # Listando las interfaces del switch 
```
Para el caso resaltamos la salida del comando **sudo ovs-vsctl show**
```
root@kali:~# ovs-vsctl show 
822290ff-8ad5-49c0-87d2-b06efc249250
    Bridge "s1"
        Port "s1"
            Interface "s1"
                type: internal
        Port "10d9d662d8404_l"
            Interface "10d9d662d8404_l"
        Port "feaeb61434ac4_l"
            Interface "feaeb61434ac4_l"
    ovs_version: "2.6.2"
```
Como se puede notar ya existen dos interfaces cada conectada a los containers previamente creados. 
> **Nota:**
> No se cual contenedor va a cual interface (Como solucion hubiera ejecutado el comando ovs-vsctl despues de crear
la conexion).
Ahora vamos a proceder a cambiar el modo de operacion del switch ovs para que se comporte como un switch openflow:
```
ovs-vsctl set-fail-mode s1 secure
```
Nuevamente miramos la salida del comando **sudo ovs-vsctl show**
```
root@kali:~# ovs-vsctl show 
822290ff-8ad5-49c0-87d2-b06efc249250
    Bridge "s1"
        fail_mode: secure
        Port "s1"
            Interface "s1"
                type: internal
        Port "10d9d662d8404_l"
            Interface "10d9d662d8404_l"
        Port "feaeb61434ac4_l"
            Interface "feaeb61434ac4_l"
    ovs_version: "2.6.2"
```
Vamos a verificar la conexion entre los containers. Tengase en cuenta que para que funcione el comando ping se deberan 
tener las netools instaladas
```
# ping h1 -> h2 (Esto se hace en la consola de h1)
ping -c 4 10.0.0.2   # IP(h2) = 10.0.0.2

# ping h2 -> h1 (Esto se hace en la consola de h2)
ping -c 4 10.0.0.1 # IP(h1) = 10.0.0.1
```
Vemos que no hay conectividad (haciendo el ping **h1 -> h2**):
```
root@fdeffa857174:/# ping -c 4 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
From 10.0.0.1 icmp_seq=1 Destination Host Unreachable
From 10.0.0.1 icmp_seq=2 Destination Host Unreachable
From 10.0.0.1 icmp_seq=3 Destination Host Unreachable
From 10.0.0.1 icmp_seq=4 Destination Host Unreachable

--- 10.0.0.2 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3070ms
pipe 4
root@fdeffa857174:/# 
```
Si se ejecuta el comando **ovs-ofctl dump-flows NOMBRE_SWITCH** se podra notar que no hay flujos instalados, por ello
no hay conexion:
```
root@kali:~# ovs-ofctl dump-flows s1
NXST_FLOW reply (xid=0x4):
```
Procediendo a instalar dos flujos empleamos los siguientes comandos
```
ovs-ofctl add-flow s1 in_port=1,actions=output:2
ovs-ofctl add-flow s1 in_port=2,actions=output:1
```
> **Nota:**
> Para mas informacion puede ver los siguientes enlaces:
> - http://www.pica8.com/document/v2.3/html/ovs-commands-reference/

Nuevamente si se lista la tabla de flujos el resultado sera el siguiente:
```
root@kali:~# ovs-ofctl dump-flows s1
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=4.808s, table=0, n_packets=0, n_bytes=0, idle_age=4, in_port=1 actions=output:2
 cookie=0x0, duration=1.871s, table=0, n_packets=0, n_bytes=0, idle_age=1, in_port=2 actions=output:1
```
Como ya existe en el switch los flujos que dicen que hacer con hacer con los paquetes que llegan
al switch ovs se puede notar que ya es posible el ping entre los contenedore tal y como se muestra a continuacion:
```
root@fdeffa857174:/# ping -c 4 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=1.70 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.110 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.090 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=0.121 ms

--- 10.0.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3052ms
rtt min/avg/max/mdev = 0.090/0.506/1.705/0.692 ms
```
Para culminar la prueba, nos salimos de los contenedores inicialmente con el comando **exit**. Luego, procedemos
a eliminar la instancia del switch ovs creado tal y como se muestran en los siguientes comandos:
```
# Eliminando los contenedores
docker rm h1
docker rm h2
# Eliminando el switch
ovs-vsctl del-br s1
# Verificando que el switch haya sido eliminado
ovs-vsctl show
```

**----------------------------------------> Aca vamos **

#### Caso 2: Topologia empleando driver switch ovs que funciona como container de docker
En este caso se hizo uso de la solución **docker-ovs-plugin** disponible en https://github.com/gopher-net/docker-ovs-plugin. 
Inicialmente se siguieron las instrucciones alli mostradas y se verificó el correcto funcionamiento del pluging una vez instalado. 
Posteriormente se procedio a realizar el montaje de la topologia de test. Las caracteristicas de la red a probar se muestran a continuacion:

| Host     | Interfaz | IP   | Imagen |
| :-------: | ----: | :---: | :--: |
| h1 | ? |  ? | kalilinux/kali-linux-docker| 
| h2 | ? |  ? | ubuntu |

En lo que respecta al switch que conectará los containers la siguiente tabla define las caracteristicas:

| switch    | Driver | Uso de Openflow  | 
| :-------: | ----: | :---: |
| s1 | ovs |  No |

Inicialmente se procede a codificar el archivo siguiente archivo en docker-compose, en el caso se llamara docker-compose-tb2.yml y su contenido sera el siguiente:

```
version: '3'

services:
  kali:
    image: "kalilinux/kali-linux-docker" 
    container_name: h1 
    entrypoint: /bin/bash
    stdin_open: true
    tty: true  
    networks:
      - mynet
        
  ubuntu:
    image: "ubuntu"
    container_name: h2  
    entrypoint: /bin/bash
    stdin_open: true
    tty: true  
    networks:
      - mynet
         
networks:
  mynet: 
    driver: ovs
```

Una vez codificado el archivo anterior se guarda. Antes de proceder a realizar el montaje se asumen los siguietes supuestos:
- El archivo **docker-compose.yml** asociado a la creacion del **docker-ovs-plugin** se encuentra en: OVS-PLUGING-DIR.
- El archivo **docker-compose-tb2.yml** se encuentra en TB2-DIR.

Los pasos de puesta en marcha son los siguientes:
1. Verifique que los contenedores asociados a **docker-ovs-plugin** esten en marcha. Si no lo estan corra el archivo docker-compose asociado a estos:

```
# Reiniciar el openvswitch
/etc/init.d/openvswitch-switch restart
# Verificando que los contenedores esten en marcha
docker ps
# Accediendo al directorio donde esta el archivo docker-compose del docker-ovs-plugin (Si no esta en marcha este plugin)
cd OVS-PLUGING-DIR
# Creando los contenedores asociados a docker-ovs-plugin
docker-compose up -d
# Verificando los conetenedores recien creados
docker ps


```
2. Crear una red asociada a este driver:
```
# Creacion de la red
docker network create -d ovs mynet
# Verificando la red recien creada
docker network ls
```

3. Ingresar al directorio TB2-DIR y arrancar la topologia:

```
# Accediendo al directorio donde esta el archivo docker-compose de la topologia
cd OVS-PLUGING-DIR
# Creando los contenedores asociados a docker-ovs-plugin
docker-compose -f docker-compose-tb2.yml up -d
# Verificando los conetenedores recien creados
docker ps
```
4. Inspeccionando los contenedores y la red recien creados:
```
# Inspeccionando los contenedores
docker ps
# Inspeccionando las redes (Nota: Para el caso se crea una red de nombre OVS-PLUGING-DIR_mynet)
docker network ls
# Verificando los conetenedores recien creados
docker network inspect
```
En el caso al realizar el inspect de la red se llego a que para el caso las IPs de los contenedores creados eran:

| Host     | Interfaz | 
| :-------: | ----: | 
| h1 | 172.20.0.2 | 
| h2 | 172.20.0.3| 

5. Verificando conectividad entre los contenedores: Para el caso solo se probo de con un ping de h1 a h2
```
# Inspeccionando los contenedores
docker exec h1 ping -c 2 172.20.0.3
```
Si todo esta bien debera haber respuesta. Si se quieren hacer diferente tipo de pruebas, es posible acceder a las consolas de los contenedores usando el comando **attach**, la idea es ejecutar este comando en dos consolas diferentes pues una vez este se pone en marcha, el control de la consola pasa al contenedor:
```
# Accediendo al contenedor h1 (Hacerlo en una consola nueva)
docker attach h1
# Accediendo al contenedor h2 (Hacerlo en una consola nueva - distinta a la anterior)
docker attach h2
```
Con el comando **exit** nos salimos de las consolas. Finalmente para dar de baja la topologia se emplea el siguiente comando 
en el directorio OVS-PLUGING-DIR:
```
# Dando de baja la topologia
docker-compose -f docker-compose-tb2.yml down
# Verificando que los conetenedores se hayan dado de baja
docker ps
# Verificando que la red se haya dado de baja
docker network ls
```

#### Caso 3: Topologia empleando driver switch ovs que funciona como container de docker

A diferencia del caso anterior, se agregaron unas cuanta lineas en el archivo docker-compose, para que las IPs de
los containers quedaran definidas, la siguiente tabla muestra el resultado:

| Host     | Interfaz | IP   | Imagen |
| :-------: | ----: | :---: | :--: |
| h1 | 10.0.0.2 |  ? | kalilinux/kali-linux-docker| 
| h2 | 10.0.0.3 |  ? | ubuntu |

En lo que respecta al switch que conectará los containers la siguiente tabla define las caracteristicas:

| switch    | Driver | Uso de Openflow  | 
| :-------: | ----: | :---: |
| s1 | ovs |  No |

El archivo docker-compose de este ejemplo se nombre **docker-compose-tb3.yml** y su contenido es el siguiente:
```
version: '3'

services:
  kali:
    image: "kalilinux/kali-linux-docker" 
    container_name: h1 
    entrypoint: /bin/bash
    stdin_open: true
    tty: true  
    networks:
      mynet:
        ipv4_address: 10.0.0.2    
        
  ubuntu:
    image: "ubuntu"
    container_name: h2  
    entrypoint: /bin/bash
    stdin_open: true
    tty: true  
    networks:
      mynet:
        ipv4_address: 10.0.0.3   
         
networks:
  mynet: 
    driver: ovs
    ipam:      
      config: 
        - subnet: 10.0.0.0/8
```
A continuacion se muestran los pasos para realizar la prueba, para este caso, ya no necesitaremos llevar a cabo los
pasos 1 y 2 del procedimiento anterior pues ya se tienen en funcionamiento los contenedores asociados al plugin docker 
de ovs, a continuacion se muestran los pasos:

1. Ingresar al directorio TB2-DIR y arrancar la topologia:

```
# Accediendo al directorio donde esta el archivo docker-compose de la topologia
cd OVS-PLUGING-DIR
# Creando los contenedores asociados a docker-ovs-plugin
docker-compose -f docker-compose-tb3.yml up -d
# Verificando los conetenedores recien creados
docker ps
```
2. Inspeccionando los contenedores y la red recien creados:
```
# Inspeccionando los contenedores
docker ps
# Inspeccionando las redes (Nota: Para el caso se crea una red de nombre OVS-PLUGING-DIR_mynet)
docker network ls
# Verificando los conetenedores recien creados
docker network inspect
```
3. Verificando conectividad entre los contenedores: Para el caso solo se probo de con un ping de h1 a h2
```
# Inspeccionando los contenedores
docker exec h1 ping -c 2 10.0.0.3
```
Si todo esta bien debera haber respuesta. Si se quieren hacer diferente tipo de pruebas, es posible acceder a las consolas de los contenedores usando el comando **attach** como ya se habia mencionado antes. 

4. Dando de baja el test: 
Finalmente para dar de baja la topologia se emplea el siguiente comando 
en el directorio OVS-PLUGING-DIR:
```
# Dando de baja la topologia
docker-compose -f docker-compose-tb2.yml down
# Verificando que los conetenedores se hayan dado de baja
docker ps
# Verificando que la red se haya dado de baja
docker network ls
```

![Montaje 2](test-sin-controlador.png?raw=true "Experimento sin controlador")
