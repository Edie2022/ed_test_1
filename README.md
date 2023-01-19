# ed_test_1

Respuesta

Antes de comenzar, me gustaría realizar algunas observaciones:

+La construcción de APis y Microservicios, es una actividad que no he realizado, pero considero que con el tiempo suficiente podría aprender. 
Llevé a cabo una breve investigación y considero que podría utilizar Spring Boot para desarollar, ya que incluye starters con librerías predefinidas, por lo que contaría con las dependencias necesarias para trabajar. Además, cuenta con un servidor incorporado (Tomcat), con el cual podría llevar a cabo el despliegue con una ejecución más rápida.

+Para el entorno de desarrollo me siento más cómodo utilizando VS studio, sin embargo, Net Beans, presenta mayores ventajas a la hora de trabajar con Maven.

+Por último, mi experiencia principal es a través de Azure DevOps, por lo que el desarrollo de la actividad on prime, ha sido un excelente desafío. Lamentablemente no he podido desarrollarla completamente.

+Para el ejercicio utilicé una aplicación dockerizada que obtuve en un curso de Udemy Docker Hub : sotobotero/udemy-devops y un código fuente descargado desde un curso de Udemy.

De acuerdo a lo que he revisado, pude llegar a 2 potenciales soluciones para las cuales utilicé una MV (Ubuntu).

Solución 1:

1-Al ser una MV, me es posible trabajar con un solo nodo (en estos momentos no cuento con acceso a servicio cloud).

Para la primera parte, instalé:
1- Docker.
2- Kubectl.
3- Dockerizé Minikube.

2-Dockerizar Jenkins
-Busqué la imagen de Jenkins en el repositorio Docker Hub (jenkins/jenkins).
-Luego revisé la documentación la manera de extender la imagen para integrar maven. 
- Elaboré un Dockerfile para integrar ambos componentes.
- Este Dockerfile tiene por un lado la imagen de Jenkins y por otro lado la indicación de descargue MAVEN desde un repositorio con los RUN necesarios.
- Construí la imagen: docker build -t Jenkins-cicd —no-cache . 
- Construí el contenedor: docker run -d -p 8080:8080 -p 5000:5000 --name jenkins jenkins-cicd (los puertos los encontré en la documentación de Jenkins).
- Ingreso a interfaz de administración a través del localhost:8080
- El Password lo busqué en el log del contenedor de Jenkins.
- Luego de esto, ya es posible empezar a trabajar con Pipeline.

3-Conexión entre GitHub y Jenkins para trigger utilizando WEBHOOKS:

- Jenkins no está visible desde la web, por que lo estoy utilizando en mi MV, por lo anterior, necesito hacer un proxy, para lo que utilicé NGROK.
- Descargué código fuente de billing desde Udemy.
- Conecté el repositorio de GitHub en Jenkins (token y rama).
- Cree nueva Pipeline con estilo libre (Github proyect).
- Github trigger asociada a una rama feature.
- Ejecutar Maven a nivel superior (clean Install); añadí archivo Pom.xml
- Lo que espero, es que con lo anterior después de cualquier cambio en la rama feature/ep, sea trigger para la compilación con Jenkins.
- De esta manera comprobé que funcionara la compilación.

4-Deploy en k8s desde Jenkins
- La configuración anterior en Webhooks es necesaria para el deploy en k8s
- Start contenedores de Jenkins y Minikube.
- Me conecté al contenedor de Jenkins ( docker exec -it —user=root jenkins /bin/bash.
- Instalé el Kubectl en el contenedor.
- Obtuve los permisos y lo moví en donde se encuentran los binarios ( ./kubectl /usr/local/bin/kubectl).
- Finalmente, conecté el contenedor de Jenkins a la red de Minikube (Docker network connect minikube jenkins).

5-Pluggin Jenkins y k8s

- Descargué e instalé el plugin de Kubernetes desde página de Jenkins.
- Fue necesario agregar los siguientes archivos:
    - jenkins-account.yml : Es una cuenta para crear un token para poder acceder desde jenkins al cluster de Kubernetes. Contiene un role y otorga privilegios para poder trabajar en Kubernetes.
    - deployment-billing-app-back-jenkins.yaml : sirve para definir el Deployment para desplegar el servicio de la aplicación en backend billing y además expone el servicio.
    - jenkinsfile: es un Pipeline declarativo, se encargará de hacer el deployment en el cluster de Kubernetes. 
    - Subí estos archivos en un nuevo repositorio remoto, para de esta manera se puedan descargar en la Pipeline y generar así el despliegue en k8s.

6-Jenkins, Github y k8s
- Apliqué los fichero en minikube “jenkins account”, ahora ya puedo obtener la url y el token. (URL del certificado del cluster, URL del cluster y certificado del rol).
- Check de los “services account”  por defecto y el que cree con el fichero.
- Detalles service account (con esto obtengo el token y lo ingreso a and Credentials en Jenkins).
- Configuro el sistema, cloud y añado el cluster de Kubernetes (todo obtenido desde el config view: URL y ruta del certificado del cluster) e indico el token para conectarme a la api del cluster.

7-Creé una nueva Pipeline
- Lo configuro como Github proyect y lo asocio al repositorio (por ahora solo con rama master).
- Agrego Jenkins file (Pipeline declarativo que se encuentra en el repositorio).
- Posterior a esto, o probé y fue posible hacer correr el cluster luego de realizar un push en Git Hub

Opción 2

Esta opción consiste en desplegar un cluster de k8s que termine por disparar una Pipeline en Jenkins utilizando la imagen de Jenkins. Para ello definí los diferentes objetos de k8s de acuerdo a las características solicitadas. A este despliegue podría agregarle la imagen anterior para poder hacerla correr de manera automática.

1- Jenkins-configmap 
2- Jenkins-deployment
3- Jenkins-namespace
4- Jenkins-service
5- Jenkins-volumen
6- service account 
