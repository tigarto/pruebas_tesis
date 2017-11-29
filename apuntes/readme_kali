Configurando el entorno de desarrollo
===================

A continuación se describen las instrucciones necesarias para configurar el entorno de desarrollo antes de realizar las pruebas.

> **Note:**
> Las instrucciones llevadas a cabo para la configuración del entorno de desarrollo se llevaron a cabo mediante un procedimiento de ensayo y error. **No hay garantía** que su aplicación de la misma manera genere los resultados que se obtuvieron en el caso. Para ser sinceros, muchos de los resultados a los cuales se llego fuero por que mi Dios es muy bueno. 

----------
Entorno de desarrollo
-------------
La distribución linux empleada fue **Kali-Linux-2017.2**, para el caso se trabajo con una imagen de vmware descarda de: https://www.kali.org/downloads/

Instalación de las herramientas
-------------
A continuacion se muestran los comandos de instalacion que se ejecutaron en la consola de Kali. Las herramientas instaladas para el caso fueron:
- docker-ce
- docker-compose
- openvswitch-switch

```
# Instalando docker-ce
apt-get install docker-ce
# Instalando docker-compose
apt-get install docker-compose
# Instalando openvswitch
apt-get install openvswitch-switch
```

Verificación de la instalacion
-------------
Basicamente lo que hacemos e verificar que los comandos informacion (ayuda, version) sean correctamente mostrados.
```
# Verificando docker-ce
docker info
# Verificando docker-compose
docker-compose -h
# Verificando openvswitch
ovs-ofctl --version
```

Instalación del docker-ovs-plugin
-------------
Para la instalacion de este plugin se siguieron las instrucciones de: https://github.com/gopher-net/docker-ovs-plugin. 
El archivo **docker-compose.yml** se modifico levemente con respecto al nombre del original quedando tal y como se muestra a continuacion:
```
plugin:
  image: gophernet/ovs-plugin
  volumes:
    - /run/docker/plugins:/run/docker/plugins
    - /var/run/docker.sock:/var/run/docker.sock
  container_name: ovs-plugin
  net: host
  stdin_open: true
  tty: true
  privileged: true
  command: -d

ovs:
  image: socketplane/openvswitch:2.3.2
  cap_add:
    - NET_ADMIN
  net: host
  container_name: ovs
```
> **Nota:**
> El test no salio exactamente como se esperaba (queda pendiente ver si esto es problema en general o sera necesario hacer una mejora (Puede que toque ver: https://github.com/tigarto/pruebas_tesis/tree/master/boot2docker/ejemplos_docker_networking/ovs_dock)
