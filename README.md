# Proyecto Grafana

## A) Máquinas en Vagrant
Lo primero que debemos tener listo son las máquinas virtuales, en este caso tendremos 2 para hacer la simulación de un clúster. Para este caso en particular lo ideal sería tener las máquinas aprovisionadas, por lo que en la creación de las mismas se instalaran los paquetes necesarios, más específicamente Docker y docker-compose, que son las herramientas que utilizaremos. En este repositorio adjuntamos el archivo Vagrant con la configuración de las máquinas y su respectivo aprovisionamiento. Recordemos que para levantar las máquinas bastará con el siguiente comando:
```bash
vagrant up
```
Esto nos levantará una máquina **server** y una máquina **client**

> Asumiremos que todos los comandos son ejecutados como root

## B) Creación de Clúster
Una vez levantadas las máquinas lo primero que debemos hacer crear el clúster y unir un worker. Se crea el clúster [1] en la máquina server y el output de este lo ejecutaremos en la máquina client [2] para unirlo al clúster.
```bash
[1] docker swarm init --adverise=192.168.57.2
#Nos arrojará un output qu utilizaremos en client
[2] docker swarm join --token [Token Generado] 192.168.57.2:2377
```

## C) Levantamiento de los servicios
El servicio principal, el cual vamos a monitorear será Redis (Base de datos NoSQL Clave-Valor)
### 1. Creación de la red en Docker
Es necesario tener una red general para que todos nuestros servicios en contenedores se puedan comunicar entre sí, por lo que crearemos una con el siguiente comando:
```bash
docker network create --driver=overlay cluster-net
```
### 2. Desplegar servicios
En este repositorio hay una carpeta llamada testCluster la cual contiene los archivos necesarios para desplegar los servicios que necesitamos. Ingresamos a la carpeta y desde allí bastará con correr el siguiente comando:
```bash
docker stack deploy -c docker-compose.yml monitoring
```
Esto crea todos los servicios bajo el dominio de un stack de servicios que lo hemos llamado *monitoring*.

## D)  Prometheus y Grafana, configuración
### 1. Verificar Prometheus
Es necesario verificar que el servicio de Prometheus esté funcionando correctamente además de que se estén procesando correctamente la data que el servicio en este caso Redis por medio de Redis-Exporter funcione bien. Para ello uno de los servicios que se levantó ha habilitado el puerto 9090 para poder observar la configuración y estado de Promeheus.
Ingresamos por la web a 192.168.57.2:9090.  Estando ahí seleccionamos el menú "*Status*" y luego el submenú "*targets*". Allí el estado de los endPoint debe ser **up** y en color verde.

![image](https://user-images.githubusercontent.com/19672268/169128671-1afe1bc8-1271-483f-adb1-3a60ee1dbb3a.png)

### 3. Conectar Grafana con Prometheus
El servicio que corre grafana nos ha habilitado el puerto 3000 para acceder a su interfaz gráfica en la web. Una vez ingresamos nos solicitara usuario y contraseña que por defecto son admin ambos. Cuando recién entramos nos aparecerá un paso a paso, que es el de configurar el dashboard. Vamos a seleccionar el que dice *Add Data Source*, esto nos llevará a una lista de diferentes opciones, seleccionamos Prometheus. Se abrirá una pantalla para configurar. Bastará con colocar la ip del cliente y su puerto (9090) de Prometheus y le damos en guardar. La del cliente ya que allá es donde está el exportador de la data de métricas.

![image](https://user-images.githubusercontent.com/19672268/169129306-e1092ce0-6a42-4639-8342-1f2b3a2bc5dd.png)

### 4. Configurar Dashboard
Cuando hemos configurado la conexión entre grafana y prometheus. Procedemos a configurar un dashboard, dentro de este repositorio tenemos un archivo .json que contiene una configuración para uno. Bastará entonces con dirigirnos al segundo menú lateral izquierdo que es el de Dashboards y en el submenú *manage* en donde podremos seleccionar la opción de importar dicha configuración. Con esto tendremos un dashboard predefinido, que nos mostrará métricas de nuestro servicio de Redis. Sin embargo, podremos personalizar dichas métricas según la necesidad.

![image](https://user-images.githubusercontent.com/19672268/169129994-39111660-63a8-4ca7-8b00-2c0e7f3c4fb8.png)
