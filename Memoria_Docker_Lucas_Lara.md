# Memoria Docker | Informática Como Servicio

## Autor - Lucas José Lara García
**Opción 1** - Desplegar un contenedor con wordpress:latest y un contenedor con mysql:5.7. El contenedor wordpress consume el puerto 3306 del contenedor mysql usando la url pública.

---

En esta memoria se describe como ha sido el proceso para realizar la práctica de PaaS de Azure paso a paso. Para poder empezar a trabajar se ha configurado el entorno descargando docker en el sistema y creando la imagen *bulletinboard:1.0* con la que se ha trabajado en la práctica. Además, se ha instalado la consola de azure en ubuntu con la orden:

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

Seguidamente se ha creado un nuevo recurso *"gruporecursoslucas"*. Este recurso es el registro de imágenes Docker de Azure:

```
az group create --name gruporecursoslucas --location "West Europe"
```

<image src="capturas/02-grupo de recursos.png">

Y a continuación se ha creado el *Container Registry* *"planserviciolucas"* sobre el grupo de recursos creado en el paso anterior:

```
az acr create --name planserviciolucas --resource-group gruporecursoslucas --sku Basic
```
<image src="capturas/03-plan-servicio.png">

---
## Push de la imagen al Container registry de Azure
---

Primeramente, para hacer el push de la imagen hay que crear un **tag** y además debe tener un formato específico, en concreto:

```
<nombreRegistro>.azurecr.io/<nombre>:<version>
```
A continuación creamos el tag para la imagen *bulletinboard* originada localmente al configurar el entorno.

```
docker tag bulletinboard:1.0 planserviciolucas.azurecr.io/bulletinboard:v1
```
Y seguidamente para comprobarlo listamos las imágenes Docker del sistema:

```
docker images
```
El resultado es el siguiente:

<image src="capturas/04-imagenes.png">

Seguidamente, al tratarse de un repositorio privado, hay que habilitar el login ejecutando el comando:

```
az acr update -n planserviciolucas --admin-enabled true
```
<image src="capturas/05-habilitar login.png">

Y si accedemos al portal de azure a la sección *registro de contenedor* se puede ver el contenedor creado:

<image src="capturas/portal-azure.png">

Posteriormente hacemos login por consola con el siguiente comando:

```
docker login planserviciolucas.azurecr.io
```

El resultado se muestra en la siguiente imagen:

<image src="capturas/06-login-servicio.png">

Finalmente ya podemos hacer el push de nuestra imagen Docker local para su despliegue en Azure con el siguiente comando:

```
docker push planserviciolucas.azurecr.io/bulletinboard:v1
```

El resultado se muestra en la siguiente imagen:

<image src="capturas/07-subir-imagen.png">

---
## Creación del contenedor con la imagen recién subida a Azure
---

Una vez que se ha subido la imagen, hay que crear el contenedor que contendrá la imagen junto a las opciones necesarias. Para ello utilizamos el siguiente comando:

```
az container create --resource-group gruporecursoslucas --name test-bulletinboard --image planserviciolucas.azurecr.io/bulletinboard:v1 --dns-name-label test-bulletinboard --ports 8080
```

El resultado es el siguiente:

<image src="capturas/08-container.png">

---
## Web con la imagen corporativa
---
Finalmente para comprobar que todo se ha realizado correctamente se puede visitar la imagen desplegada en en el siguiente enlace:
> http://lucascontainer.westeurope.azurecontainer.io:8080/

El resultado se muestra en la siguiente imagen:

<image src="capturas/09-imagen-Operativa.png">