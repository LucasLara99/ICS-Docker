# Memoria Docker | Informática Como Servicio

## Autor - Lucas José Lara García
**Opción 2** - Desplegar, mediante docker-compose, un contenedor que contenga dos servicios con las dos imágenes anteriormente mencionadas (wordpress:latest y mysql:5.7). Unicamente existe una url pública. La comunicación entre los servicios se hace de forma interna dentro del contenedor desplegado. Por lo tanto, solamente está expuesto el puerto 80 del servicio wordpress al exterior.

---

En esta memoria se describe como ha sido el proceso para realizar la práctica de PaaS de Azure paso a paso. Para poder empezar a trabajar se ha configurado el entorno descargando docker en el sistema y se ha habilitado su uso sin *sudo* con las órdenes: 

```
sudo groupadd docker
sudo usermod -aG docker $USER
```

Para comprobar que todo funciona correctamente ejecutamos el **hola mundo** de docker 

```
docker run hello-world
```

Cuya salida es:

<image src="capturas/holamundo.png">

Además, se ha instalado la consola de azure en ubuntu con la orden:

```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
De igual manera es necesario instalar *Helm*, la dependencia para el despliegue de contenedores, ejecutando el comando:

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

---
## Login en azure
---

En primer lugar se ha creado una cuenta de Azure asociada a la de la udc para poder obtener una serie de servicios gratuitos. A continuación se ha iniciado sesión.

```
az login
```

<image src="capturas/01-azure-login.png">

---
## Creación del Container Registry
---

Seguidamente se ha creado un nuevo recurso *"recursos"*. Este recurso es el registro de imágenes Docker de Azure:

```
az group create --name recursos --location "West Europe"
```

<image src="capturas/02-grupo de recurso.png">

Y a continuación se ha creado el *Container Registry* *"serviciolucasics"* sobre el grupo de recursos creado en el paso anterior:

```
az acr create --name serviciolucasics --resource-group recursos --sku Basic
```
<image src="capturas/03-planservicio.png">

---
## Push de la imagen al Container registry de Azure
---
En primer lugar, hacemos login en nuestro plan de servicios con la orden:

```
az acr login --name serviciolucasics
```
<image src="capturas/loginservicio.png">

En segundo lugar, hacemos pull de las imágenes con las versiones correspondientes que se nos requerían en el enunciado:

```
docker image pull wordpress:latest
docker image pull mysql:5.7
```
El resultado es el siguiente:

<image src="capturas/pullwordpress.png">
<image src="capturas/pullsql.png">

Despúes, para hacer el push de las imágenes hay que crear un **tag** y además debe tener un formato específico, en concreto:

```
<nombreRegistro>.azurecr.io/<nombre>:<version>
```
A continuación creamos el tag para las imágenes previamente creadas.

```
docker tag wordpress:latest serviciolucasics.azurecr.io/wordpress:latest
docker tag mysql:5.7 serviciolucasics.azurecr.io/mysql:5.7
```
Y seguidamente para comprobarlo listamos las imágenes Docker del sistema:

```
docker images
```
El resultado es el siguiente:

<image src="capturas/04-tags.png">

Seguidamente, al tratarse de un repositorio privado, hay que habilitar el login ejecutando el comando:

```
az acr update -n serviciolucasics --admin-enabled true
```
<image src="capturas/05-habilitar-login.png">

Y si accedemos al portal de azure a la sección *registro de contenedor* se puede ver el contenedor creado:

<image src="capturas/portalazure.png">

Posteriormente hacemos login por consola con el siguiente comando:

```
docker login serviciolucasics.azurecr.io
```

El resultado se muestra en la siguiente imagen:

<image src="capturas/06-loginservicio.png">

Finalmente ya podemos hacer el push de las imagenes Docker para su despliegue en Azure con el siguiente comando:

```
docker push myserviceplan.azurecr.io/wordpress:latest
docker push myserviceplan.azurecr.io/mysql:5.7
```

El resultado se muestra en la siguiente imagen:

<image src="capturas/pushwordpress.png">
<image src="capturas/pushmysql.png">

---
## Creación del contenedor con la imagen recién subida a Azure
---

