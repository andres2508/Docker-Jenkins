#Proyecto Final: Integración Continua
<b><i>Universidad Icesi</i></b><br>
<li>https://github.com/andres2508/Docker-Jenkins </li>


Integrantes: 
    Jaime Andres Aristizabal 11104055                  
    Diego Esteban Delgado  12107019


#Objetivos:
<ul>Comprender los conceptos de integración continua (continuous integration) y aplicarlo en un caso real </ul>
<ul>Realizar el aprovisionamiento automático de un maestro y nodos de trabajos para dar soporte a procesos de integración continua</ul>


El siguiente proyecto pretende realizar el aprovisionamiento de un ambiente que permite la integración continua, para su  desarrollo se propone el uso de herramientas como Jenkins con la tecnología de virtualización docker y máquinas virtuales. El esquema a utilizar será el siguiente:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/esquema.png)


#Procedimiento:
## 1. Clonar Repositorio
```
$ git clone https://github.com/andres2508/Docker-Jenkins
```
## 2. Configuración del API Docker
Para este proceso se configura el Api de Docker se la siguiente forma:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/config-docker-api.png)

## 3. Crear imagen master
Como se puede observar en el directorio Docker-Jenkins/master/files se encuentra el archivo Dockerfile y adicional plugins.txt, se describe el contenido de estos archivos a continuación:
Dockerfile
![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/Dockerfile.PNG)


Este archivo define inicialmente el usuario con el que se realizarán las configuraciones, posterior a esto se procede a instalar los plugins necesarios para un correcto despliegue, estos son:
Docker-Plugin: Nos permite ejecutar los contenedores de los esclavos, se dirige al API de la máquina donde están las imágenes de los contenedores y levanta los esclavos. 
saferestart: Permite un Reinicio seguro de nuestra máquina.
git/github: Permite la conexión con Github.
Estos plugins se encuentran ubicados en el archivo plugins.txt el cual es copiado dentro el Dockerfile al contenedor buscando que sea una referencia local para la máquina y de esta forma realizar su instalación.
Finalmente se expone el puerto 8080 por el cual se va a comunicar nuestro nodo maestro.
Plugins.txt

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/plugins.PNG)

Este archivo contiene los nombres de los plugins a instalar y su correspondiente versión.
Una vez definidos estos parámetros usamos el siguiente comando para terminar la construcción del nodo maestro:
```
$ cd master
$ docker build -t jenkins_master . 
```
## 4. Crear imagen slave
Para la creación del nodo esclavo se procede a configurar el archivo Dockerfile que se muestra a continuación:
  Dockerfile
  ![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/Dockerfileslave.PNG)

Como se observa en la anterior imagen en este caso se instalan las librerías necesarias para ejecutar una prueba de su funcionamiento que se realizará posteriormente. Se instala la librería de Python y Virtualenv para tener un entorno virtual aislado del sistema central, se realizan las configuraciones de los directorios a usarse, posterior a esto se configura la sección de Prueba del Proyecto usando la librería de virtualenv y finalmente se instalan los paquetes necesarios para las pruebas.


Finalmente se usa el siguiente comando para construir la imagen:
```
$ cd slave
$ docker build -t jenkins_slave . 
```
## 5. Ejecutar Contenedor Maestro
Una vez construidas las imágenes se procede a ejecutarlas:
$ docker run -p 8080:8080 -d jenkins_master
Para comprobar que se ha ejecutado correctamente vamos al navegador y cargamos la dirección 192.168.1.58 ó127.0.0.1:8080, se debe visualizar algo similar a este dashboard:
  ![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/dashboard.png)


## 6. Configuración del Docker Cloud 
Con el fin de que Jenkins cree nodos esclavos para la ejecución de las pruebas usamos el Docker-plugin que se instaló previamente y se debe configurar los parámetros del servidor donde se encuentran las imágenes de los contenedores.
Para realizar esta configuración nos dirigimos a la sección de Manage Jenkins → Sytem Configuraton → Add Cloud → Docker y se registra la información:

  ![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/manage-jenkins.png)

  
Estando en esta sección oprimimos en la pestaña de Cloud - Add new Cloud como se observa en esta imagen:

 ![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/addcloud.png)

Una vez se le da click se despliega un menú como el siguiente, es importante tener en cuenta que la URL de la API de docker debe pasarse como protocolo tcp y no como http:

 ![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/addcloud2.png)


Se deben llenar estos campos como se muestra en la anterior imagen,  adicional a esto adicionar la imagen que se desea usar:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/addtemplate.png)


Al determinar que se desea adicionar la imagen Docker template se despliega un menú adicional que debe ser llenado de la siguiente forma. En esta parte es importante tener en cuenta que se configura un label en el template de docker, esto con el objetivo de definir sobre qué contenedor se van a ejecutar los jobs:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/addcloud3.png)


Se deben determinar los parámetros necesarios o credenciales para esta conexión, esto se realiza al presionar Add credencial, los parámetros necesarios son los siguientes:  
```
$ usuario = jenkins, contraseña= jenkins.
```
![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/addcloud4.png)


Como esto se termina la configuración necesaria del Docker Cloud con la que el Jenkins Master se conectará con el fin de observar las imágenes que se van a levantar para los esclavos identificadas con un label que permite decirle que esa es la imagen que desean ejecutar. 

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/addcloud5.png)

En la anterior imagen se puede observar la conexión realizada con el Docker Cloud y una lista de las imágenes obtenidas .


## 7. Configuración de un Job
Ahora bien una vez configurado lo anterior se procede a la configuración de un Jobs, para esto estando en la página principal de Jenkins creamos un nuevo Job, para nuestro caso se llamará Test-Project:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/job.png)


Posterior a esto se procede a realizar la configuración como se muestra en las siguientes imágenes:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/job1.png)

Pasamos a la pestaña de Source Code y configuramos como se observa a continuación. Es importante que es la zona de “restrict where this project can be run” se coloque el label que se le dio cuando se crea el docker cloud:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/job2.png)

Luego nos dirigimos a la pestaña de Build Environment y configuramos como se observa en la imagen, esto nos permitirá ejecutar la prueba: 

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/job3.png)

Como se observó en la anterior configuración se especifica que el trabajo se realice dentro de un contenedor usando Docker, se especifica que se debe correr usando solo un nodo específico, se establece que la fuente del código estará en un repositorio de GitHub, finalmente se establece que se ejecute un comando de Shell en el momento de la construcción del trabajo, ejecutando así la aplicación disponible en el repositorio.

#Test
## 1. Resultado de las pruebas del Job
Una vez terminada la configuración se da en save para guardarla y se redirecciona a la página principal del job, se hace click en Build Now lo que iniciará el proceso de construcción de la prueba.
Finalmente los resultados de esta prueba es vista en la salida de la consola de la siguiente forma, donde se puede ver que se hizo el pull del repositorio y se probó el proyecto:

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/console1.png)

![alt tag] (https://github.com/andres2508/Docker-Jenkins/blob/master/images/console2.png)

## Referencias:
<ul>
<li>https://github.com/d4n13lbc/testproject</li>  
<li>http://devopscube.com/docker-containers-as-build-slaves-jenkins/
</li>
<li>https://engineering.riotgames.com/news/jenkins-ephemeral-docker-tutorial
</li>
<li>
https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin
</li>
<li>http://www.virtuallyghetto.com/2014/07/quick-tip-how-to-enable-docker-remote-api.html
</li>
</ul>
 
