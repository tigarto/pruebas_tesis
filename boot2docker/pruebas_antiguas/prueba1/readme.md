Pruebas
===================

A continuación se describe de manera somera y sin correcciones posteriores el proceso llevado a cabo para configurar la topologia de test.
-> **Nota:**
-> Aca el test solo se preocupa por la conectividad entre los diferentes elementos de la red. Todavia no se hace enfasís en el montaje del test.

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

####  Comandos de instalacion en consola
```
# Aca van los comandos

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
