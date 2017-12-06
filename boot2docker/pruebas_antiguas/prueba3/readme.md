Pruebas
===================

A continuación se describe de manera somera y sin correcciones posteriores el proceso llevado a cabo para configurar la topologia de test. El objetivo de estos ensayos es realizar el montaje empleando como medio de conexion switch virtual con driver ovs (open-vswitch) y no con driver bridge (como se hizo en los casos anteriormente descritos). Una vez se tenga correctamente la topologia funcional, el siguiente paso consiste en agregar de forma manual reglas al swich ovs para permitir la conectividad entre los containers.

> **Nota:**
> Aca el test solo se preocupa por la conectividad entre los diferentes elementos de la red. Todavia no se hace enfasís en el montaje del test definitivo.
--------------
#####  Descripción de la topologia
La siguiente figura muestra la topología a montar:

![Montaje 1](test-con-switch-ovs.png?raw=true "Experimento empleando un swith ovs")


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
# 1. Se agrega el switch virtual y sus interfaces 
sudo ovs-vsctl add-br s1

# 2. Se verifica que el switch haya sido creado
sudo ovs-vsctl list-br                      # Listando los switchs existentes
sudo ovs-vsctl show                         # Mostrando informacion resumida de los switches existentes
sudo ovs-vsctl list-ifaces s1               # Listando las interfaces del switch 

# 3. Se crean los contenedores
# Poner a correr las 2 imagenes en dos consolas diferentes si no se coloca en modo 
# detach (ejecucion en background) 
docker run -it --name=h1 --net=none kalilinux/kali-linux-docker /bin/bash 
docker run -it --name=h2 --net=none ubuntu /bin/bash 

# 4. Se verifica que los contanedores esten corriendo 
docker ps
docker inspect --format '{{ .NetworkSettings.IPAddress }}' h1
docker inspect --format '{{ .NetworkSettings.IPAddress }}' h2

# 4. Se conectan los contenedores al switch ovs
ovs-docker add-port s1 h1-eth0 h1 --ipaddress=10.0.0.1/8 
ovs-docker add-port s1 h2-eth0 h2 --ipaddress=10.0.0.2/8 

# 4. Test the connection between two containers connected via OVS bridge using Ping command
en h1: ifconfig ---> Se puede ver la Ip es la que esperamis
```
Para verificar que la conexion entre los contenedores este bien, se ejecutan los comandos de conectividad (ping) desde cada una de las 
consolas de los contenedores si los contenedores lo tienen instalado. La idea es que por lo menos un contenedor lo tenga instalado.

```
# ping h1 -> h2 (Esto se hace en la consola de h1)
ping -c 4 IP(h2)

# ping h2 -> h1 (Esto se hace en la consola de h2)
ping -c 4 IP(h1)
```
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
