---
title: "Por qué Docker & R?"
author: "Alberto Almuiña"
date: '2020-11-07T21:13:14-05:00'
description: Crea un contenedor Docker para encapsular tus proyectos en R.
slug: docker_r_es
tags:
- contenedores
- Docker
categories:
- Despliegue
- Produccion
---

## Qué es Docker?

Docker es un proyecto de código abierto que automatiza la implementación de aplicaciones dentro de contenedores de software, proporcionando una capa adicional de abstracción y automatización de la virtualización de aplicaciones en múltiples sistemas operativos.

La principal diferencia con una máquina virtual es que los contenedores son mucho más ligeros y tienen un consumo mucho más eficiente de los recursos disponibles.

## Por qué Docker & R?

Docker es una gran herramienta para la implementación de modelos y aplicaciones. También evita grandes conflictos de dependencias. Por ejemplo, imagine la siguiente situación: está creando un modelo con R 3.5 y la biblioteca 'A' 1.0 y necesita compartir su modelo con un compañero. Su compañero tiene R 3.6 y la biblioteca 'A' 2.0 instalada y si actualiza su versión a la de su modelo, otros proyectos se rompen. Aquí es donde entra en juego Docker.

### ¿Cuál es la diferencia entre una imagen y un contenedor?

Las imágenes son aquellas que se crean a partir de un conjunto de instrucciones e indican la configuración del sistema operativo (aquí es donde se instalan todas las dependencias y paquetes necesarios y las versiones de los programas que necesita). Los contenedores son instancias de esa imagen en ejecución. ¿Qué significa esto? Creas tu imagen una vez con la configuración necesaria y cada vez que se corra, se creará un contenedor.

## Dockerfile y Crear una Imagen

Las imágenes de Docker se crean a partir de un archivo llamado Dockerfile. Este archivo indica la imagen a partir de la cual se creará nuestra imagen, cómo configurar el sistema operativo, los programas / paquetes necesarios y qué hará el contenedor cuando se inicie.

Vamos a crear una imagen en Docker que hará lo siguiente (en este ejemplo, creamos la imagen en el sistema operativo Debian 10):

1. Instalar la versión de R requerida.

2. Instalar paquetes 'magick' y 'DSpoty' (con su versión correspondiente).

3. Establecer una variable de entorno (será un nombre de artista, 'Arctic Monkeys' por defecto).

4. Usar la librería Dspoty para obtener un enlace a la foto del artista que se pasa por parámetro y guardarla con la librería 'magick' dentro del contenedor.

5. Exportar esta imagen desde el contenedor a nuestro host.


Afortunadamente, **rocker** ha subido varias imágenes que tienen R instalado en el repositorio de DockerHub, por lo que esta será nuestra imagen inicial. En este caso, vamos a elegir una imagen que tenga R 3.6.3 instalado como punto de partida. Esto es lo que se especifica en la primera línea del Dockerfile (siempre debe comenzar con un 'FROM' que indica la imagen desde la que se inicia).

En segundo lugar, definimos una variable de entorno. Esta variable será el nombre del artista del que queremos obtener una foto. Cada vez que ejecutamos una instancia de la imagen Docker (es decir, creamos un contenedor), debemos especificar este valor si queremos que sea diferente del valor predeterminado.

Posteriormente, tenemos tres comandos RUN (estos comandos solo se ejecutan una vez cuando se construye la imagen). En este caso, se crea un directorio dentro de la imagen para trabajar en él y se instalan los paquetes necesarios para poder instalar las tres librerías de R posteriores.

**Nota:** La librería 'remotes' nos permite instalar las versiones deseadas del resto de librerías de R. Debemos tener en cuenta que si por algún motivo este paquete se eliminara del repositorio de CRAN, todas nuestras librerías restantes no podrían instalarse y fallarían. (Esto también se aplica a otras librerías, pero a menos que sean librerías raras, no deberíamos tener problemas). En cualquier caso, si son de vital importancia para nuestro negocio / tarea, una buena opción sería tener nuestro propio repositorio central con todas las librerías que necesitamos para que siempre estén ahí.

Los dos últimos pasos son la copia de nuestro script R dentro de la imagen y finalmente usamos la instrucción CMD (esto se iniciará cada vez que ejecutemos un contenedor de la imagen) en el que se especifica el lanzamiento del script R y movemos el imagen a otro directorio.

Este es el Dockerfile utilizado:

![Dockerfile](/img/docker_r.es_files/docker_r_dockerfile.jpg)

Este el script de R:

![R Script](/img/docker_r.es_files/rscript.jpg)

Finalmente, se necesita construir la imagen con el comando 'build' y luego ejecutar el contenedor con el comando 'run'.Ten en cuenta que la opción -v se usa para especificar la equivalencia entre la carpeta del host y la del contenedor, de modo que la imagen guardada dentro del contenedor no se pierda:

![Comandos Docker](/img/docker_r.es_files/commands.jpg)


Se pueden encontrar las imágenes en la carpeta esperada:

![Resultados](/img/docker_r.es_files/results.jpg)


**Disfruta!**
