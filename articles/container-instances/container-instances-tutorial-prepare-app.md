---
title: "Tutorial de Azure Container Instances: preparación de la aplicación"
description: "Preparación de una aplicación para la implementación en Azure Container Instances"
services: container-instances
author: seanmck
manager: timlt
ms.service: container-instances
ms.topic: tutorial
ms.date: 11/20/2017
ms.author: seanmck
ms.custom: mvc
ms.openlocfilehash: 6a168b600833c7e4544da7a5f5f4b7a0af73e0b5
ms.sourcegitcommit: a48e503fce6d51c7915dd23b4de14a91dd0337d8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/05/2017
---
# <a name="create-container-for-deployment-to-azure-container-instances"></a>Crear contenedor para su implementación en Azure Container Instances

Azure Container Instances permite la implementación de contenedores de Docker en la infraestructura de Azure sin necesidad de aprovisionar ninguna máquina virtual o adoptar algún servicio de nivel superior. En este tutorial, se compila una pequeña aplicación web en Node.js y se empaqueta en un contenedor que se puede ejecutar mediante Azure Container Instances. Trataremos los siguientes temas:

> [!div class="checklist"]
> * La clonación de origen de la aplicación desde GitHub
> * La creación de imágenes de contenedor desde el origen de la aplicación
> * La prueba las imágenes en un entorno local de Docker

En los tutoriales posteriores, se carga la imagen en una instancia de Azure Container Registry y, después, se implementa en Azure Container Instances.

## <a name="before-you-begin"></a>Antes de empezar

Para realizar este tutorial es necesario que ejecute la versión 2.0.21 o superior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure 2.0](/cli/azure/install-azure-cli).

En este tutorial se supone que el usuario tiene un conocimiento básico de los principales conceptos de Docker, como los contenedores, las imágenes de contenedor y los comandos básicos de `docker`. Si es necesario, consulte la [introducción a Docker]( https://docs.docker.com/get-started/), donde encontrará datos básicos acerca de los contenedores.

Para completar este tutorial, se necesita un entorno de desarrollo de Docker. Docker proporciona paquetes que permiten configurar Docker fácilmente en cualquier sistema [Mac](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) o [Linux](https://docs.docker.com/engine/installation/#supported-platforms).

Azure Cloud Shell no incluye los componentes de Docker necesarios para completar cada paso de este tutorial. Por lo tanto, le recomendamos que realice una instalación local del entorno de desarrollo de la CLI de Azure y Docker.

## <a name="get-application-code"></a>Obtención del código de la aplicación

El ejemplo de este tutorial incluye una aplicación web simple compilada en [Node.js](http://nodejs.org). La aplicación sirve una página HTML estática y tiene el siguiente aspecto:

![Aplicación del tutorial tal como se muestra en un explorador][aci-tutorial-app]

Use git para descargar el ejemplo:

```bash
git clone https://github.com/Azure-Samples/aci-helloworld.git
```

## <a name="build-the-container-image"></a>Compilación de la imagen del contenedor

El Dockerfile que se proporciona en el repositorio de ejemplo muestra cómo se compila el contenedor. Comienza con una [imagen oficial de Node.js][dockerhub-nodeimage] basada en [Alpine Linux](https://alpinelinux.org/), una pequeña distribución muy apropiada para usarla con contenedores. Luego copia los archivos de aplicación en el contenedor, instala las dependencias mediante Node Package Manager y, por último, inicia la aplicación.

```Dockerfile
FROM node:8.2.0-alpine
RUN mkdir -p /usr/src/app
COPY ./app/* /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js
```

Use el comando `docker build` para crear la imagen del contenedor y etiquétela como *aci-tutorial-app*:

```bash
docker build ./aci-helloworld -t aci-tutorial-app
```

La salida del comando `docker build` es similar a la siguiente (truncada para favorecer la legibilidad):

```bash
Sending build context to Docker daemon  119.3kB
Step 1/6 : FROM node:8.2.0-alpine
8.2.0-alpine: Pulling from library/node
88286f41530e: Pull complete
84f3a4bf8410: Pull complete
d0d9b2214720: Pull complete
Digest: sha256:c73277ccc763752b42bb2400d1aaecb4e3d32e3a9dbedd0e49885c71bea07354
Status: Downloaded newer image for node:8.2.0-alpine
 ---> 90f5ee24bee2
...
Step 6/6 : CMD node /usr/src/app/index.js
 ---> Running in f4a1ea099eec
 ---> 6edad76d09e9
Removing intermediate container f4a1ea099eec
Successfully built 6edad76d09e9
Successfully tagged aci-tutorial-app:latest
```

Use `docker images` para ver la imagen compilada:

```bash
docker images
```

Salida:

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
aci-tutorial-app             latest              5c745774dfa9        39 seconds ago       68.1 MB
```

## <a name="run-the-container-locally"></a>Ejecute el contenedor localmente

Antes de intentar implementar el contenedor en Azure Container Instances, ejecútelo localmente para confirmar que funciona. El modificador `-d` permite que el contenedor se ejecute en segundo plano, mientras que `-p` permite asignar un puerto arbitrario en el proceso al puerto 80 del contenedor.

```bash
docker run -d -p 8080:80 aci-tutorial-app
```

Abra el explorador en http://localhost: 8080 para confirmar que se está ejecutando el contenedor.

![Ejecución local de la aplicación en el explorador][aci-tutorial-app-local]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una imagen de contenedor que se puede implementar en Azure Container Instances. Se han completado los siguientes pasos:

> [!div class="checklist"]
> * Se ha clonado el origen de la aplicación desde GitHub
> * Se han creado imágenes de contenedor desde el origen de la aplicación
> * El contenedor se ha probado localmente

Pase al siguiente tutorial para aprender a almacenar imágenes de contenedor en Azure Container Registry.

> [!div class="nextstepaction"]
> [Insertar imágenes en Azure Container Registry](./container-instances-tutorial-prepare-acr.md)

<!-- LINKS -->
[dockerhub-nodeimage]: https://store.docker.com/images/node

<!--- IMAGES --->
[aci-tutorial-app]:./media/container-instances-quickstart/aci-app-browser.png
[aci-tutorial-app-local]: ./media/container-instances-tutorial-prepare-app/aci-app-browser-local.png