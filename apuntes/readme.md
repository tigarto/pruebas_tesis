Configurando el entorno de desarrollo
===================


A continuación se describe de manera somera y sin correcciones posteriores el proceso llevado a cabo para configurar el entorno de desarrollo.

> **Note:**

> Las instrucciones llevadas a cabo para la configuración del entorno de desarrollo se llevaron a cabo mediante un procedimiento de ensayo y error. **No hay garantía** que su aplicación de la misma manera genere los resultados que se obtuvieron en el caso. Para ser sinceros, muchos de los resultados a los cuales se llego fuero por que mi Dios es muy bueno. 

----------
Entorno de desarrollo
-------------
La distribución linux empleada fue **Ubuntu 14.04.5 (Trusty Tahr)**, para el caso se trabajo con una imagen de vmware descargada de: http://www.osboxes.org/ubuntu/

Instalación de docker
-------------
Para instalar Docker se siguieron las instrucciones de la siguiente pagina: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#trusty-1404
####  Comandos de instalacion en consola
```
# Comandos de instalacion
sudo apt-get update

sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt-get update

sudo apt-get install docker-ce

# Verificacion de la instalacion
sudo docker info
```
Instalación de docker-compose
-------------
Para instalar docker-compose se siguieron las instrucciones de la siguiente pagina: https://docs.docker.com/compose/install/#install-compose
```
# Comandos de instalacion
sudo curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# Verificacion de la instalacion
docker-compose --version
```

Documents
-------------
