Pruebas
===================

A continuación se describe de manera somera y sin correcciones posteriores el proceso llevado a cabo para configurar la topologia de test.
> **Nota:**
> Aca el test solo se preocupa por la conectividad entre los diferentes elementos de la red. Todavia no se hace enfasís en el montaje del test.

Prueba 1
--------------
La siguiente figura muestra la topología a montar:

![Montaje 1](test1.png?raw=true "Experimento 1")

####  Caso 1: Topologia empleando driver bridge

#####  Descripción de la topologia

| Host     | Interfaz | IP   | Imagen |
| :------- | ----: | :---: | :--: |
| h1 | ? |  ? | kalilinux/kali-linux-docker| 
| h2 | ? |  ? | ubuntu |

#####  Comandos
Solo basta con aplicar los comandos mas basicos de docker. Para el caso, los dos contenedores quedan automaticamente conectados.
```
# Creando los contenedores
# Container h1
docker run -it --name=h1 kalilinux/kali-linux-docker /bin/bash 
# Container h2
docker run -it --name=h2 ubuntu /bin/bash
```
#####  Información de la topología
A continuacion se muestran algunos comandos empleados para obtener informacion de los contenedores asociados a la red
```
docker ps
docker inspect --format='' h1
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' h1
docker exec h1 cat /etc/hosts
docker inspect --format='' h2
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' h2
docker exec h2 cat /etc/hosts
```
A continuacion se muestran los comandos para obtener información asociada a la red:
```
docker network ls
docker network inspect bridge
```
Mas comandos pueden ser consultados en:
- http://networkstatic.net/10-examples-of-how-to-get-docker-container-ip-address/
- https://github.com/ajohnstone/dot-files/blob/master/bash.d/bash/docker 
- http://networkstatic.net/docker-one-liners/

####  Caso 2: Topologia empleando driver bridge (Las IPs de los containers se definen manualmente)

#####  Descripción de la topologia

####  Descripción de la topologia

| Host     | Interfaz | IP   | Imagen |
| :------- | ----: | :---: | :--: |
| h1 | ? |  10.0.0.1/8 | kalilinux/kali-linux-docker| 
| h2 | ? |  10.0.0.2/8 | ubuntu |

Inicialmente no la conectividad no sera empleando OpenFlow.

####  Comandos de montaje de la topología
A continuación se muestran los montajes para llevar a cabo la topología:

#####  Comandos

Inicialmente se creo y testeo la red:
```
# Creando la red
docker network create \
  --driver=bridge \
  --subnet=10.0.0.0/8 \
  --gateway=10.0.0.254 \
  mynet
# Listando las redes disponibles
docker network ls
# Inspeccionando las propiedades de la red creada
docker network inspect mynet
```
Posteriormente se crearon los contenedores
```
# Creando los contenedores
# Container h1
docker run --network=mynet --ip 10.0.0.1 -it --name=h1 kalilinux/kali-linux-docker /bin/bash
# Container h2
docker run --network=mynet --ip 10.0.0.2 -it --name=h2 ubuntu /bin/bash
```
#####  Información de la topología
A continuacion se muestran algunos comandos empleados para obtener informacion de los contenedores asociados a la red
```
docker ps
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' h1
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' h2
```
El siguiente enlace puede ser de utilidad para ayudar a comprender el procedimiento para profundizar mas al respecto: https://stackoverflow.com/questions/27937185/assign-static-ip-to-docker-container 

####  Caso 3: Topologia empleando driver bridge pero definiendo la topologia en un archivo docker-compose

#####  Descripción de la topologia

| Host     | Interfaz | IP   | Imagen |
| :------- | ----: | :---: | :--: |
| h1 | ? |  ? | kalilinux/kali-linux-docker| 
| h2 | ? |  ? | ubuntu |

Inicialmente no la conectividad no sera empleando OpenFlow. Asi mismo la definicion de la topologia no sera empleando
comandos en consola sino haciendo uso de docker-compose (https://docs.docker.com/compose/).

####  Comandos de montaje de la topología
El proceso para montar y hacer pruebas sobre la topología topología se muestra a continuación:
1. Codificar el archivo .yml en el editor de texto preferido. El nombre de este por default será docker-compose.yml
2. Ejecutar el comando para procesar el archivo de docker compose y poner en marcha la topología
3. Verficar mediante los comandos de información (inspect por ejemplo) que la topologia creada sea consistente con lo definido en el archivo de docker-compose.
4. Hacer las respectivas pruebas y analizar los resultados.
5. Una vez se hagan las pruebas bajar la topologia.

#####  Comandos
####  Codificando el archivo docker-compose.yml
A continuación se muestra el archivo codificado:
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
      - mynet2      
  ubuntu:
    image: "ubuntu"
    container_name: h2  
    entrypoint: /bin/bash
    stdin_open: true
    tty: true  
    networks:
      - mynet2
      
networks:
  mynet2:
    driver: bridge
```

#####  Puesta en marcha de la topologia
En el directorio donde se encuentra el archivo docker-compose.yml recien editado, se ejecuta el comando de docker-compose para lanzar la red:
```
# Lanzando la red
docker-compose up -d
# Verificando 
docker-compose ps
docker-compose logs
```
-> **Nota:**
-> Para mas información consultar: https://docs.docker.com/compose/reference/overview/#command-options-overview-and-help.

#####  Comandos de verificacion
A continuación se muestran algunos comandos de verificación empleados para verificar que la topología lanzada este de acuerdo a lo deseado:

```
# Verificando la red
docker network ls
docker network inspect mynet2  # Tambien puede ser el ID(mynet2)
                               # Tambien en el peor de los casos el nombre que aparece con el ls
# Verificando los containers
docker ps
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' h1
docker inspect -f '{{.NetworkSettings.Networks.bridge.IPAddress}}' h2
```
#####  Pruebas en la red
Para el caso se hizo una prueba de conectividad sencilla haciendo un ping desde el container de kali (pues es el unico con 
esta utilidad instalada). Si todo sale bien, habra conectividad indicando que vamos por buen camino:

```
ping -c 4 IP(h2)
```
> **Nota:**
> - El comando anterior se ejecutó dentro del container h1 (el de kali).
> - Se pueden hacer las pruebas que se quieran (de acuerdo a las necesidades).

#####  Bajando la red
Para ello se usa el comando down de docker-compose en el directorio en el que se encuentra el archivo docker-compose.yml, asi:
```
docker-compose down
```


####  Referencias
- http://blog.terranillius.com/post/everyday_hacks/
- https://www.ctl.io/developers/blog/post/what-to-inspect-when-youre-inspecting
- https://container42.com/2016/03/27/docker-quicktip-7-psformat/
- http://container-solutions.com/docker-inspect-template-magic/
- https://www.katacoda.com/courses/docker/formatting-ps-output 
- https://coderwall.com/p/2rpbba/docker-create-a-bridge-and-shared-network
- http://www.dasblinkenlichten.com/docker-networking-101-user-defined-networks/
- http://blog.arungupta.me/docker-bridge-overlay-network-compose-variable-substitution/
- http://docker-k8s-lab.readthedocs.io/en/latest/docker/docker-compose.html
- https://runnable.com/docker/docker-compose-networking
- https://runnable.com/docker/basic-docker-networking
- http://www.summa.com/blog/docker-for-developers-composing-multi-container-networks
