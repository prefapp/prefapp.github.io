---
title: Tutorial de Puzzle (I)
keywords: puzzle, docker, stack
tags:
    - puzzle
    - docker
    - stack
layout: page
toc: false
permalink: tutorial_puzzle_1
---

# Sobre puzzle

Puzzle es una herramienta administrativa que facilita la gestión de proyectos basados en Docker mediante la creación dinámica de ficheros docker-compose.yml.

Con Puzzle se puede:

* Gestionar varios servicios (expresados en diferentes docker-compose) como si se tratase de uno. 
* Establecer de manera sencilla diversos entornos para uno o varios servicios. 
* Crear tareas para ejecutar en varios servicios desde un único comando. 
* Exportar/importar un proyecto a diferentes máquinas. 

Es **open source** y se encuentra [aquí](https://github.com/prefapp/puzzle).

# Sobre esta tutorial

Este tutorial es el primero de una serie que pretende mostrar los puntos claves de puzzle. 

Para ello, vamos a desplegar un único servicio que consta:

* Una pequeña aplicación escrita en php que puedes descargar de [aquí](http://localhost/meter_codigo)
* Una bbdd que necesita la aplicación para funcionar que mueve Mysql

# Tutorial I: Aplicación básica

## 1. Preparativos previos

### Cocinando una imagen de PHP

Vamos a partir, para nuestra aplicación, de una imagen oficial de PHP que encontramos en [dockerhub](https://hub.docker.com/_/php/).

La descargamos a nuestra máquina. 

```bash
docker pull php
```

La imagen nos viene sin la extensión de mysql para PHP que necesita nuestra aplicación. Así que vamos a cocinar una imagen con la extensión mysqli instalada. 

Tageamos la imagen a 'mi_app_php'

```bash
docker tag php mi_app_php
```

Arrancamos un container con esa imagen y empleamos el útil comando que nos ofrece la gente de PHP para instalar la extensión mysqli:

```bash
docker run --name mi_app_php -ti mi_app_php docker-php-ext-install mysqli 
```
El container está ahora apagado, pero ya tiene instalada la extensión que necesitamos. Nos basta ahora con volcar ese cambio a la imagen que estamos cocinando:

```bash
docker commit mi_app_php mi_app_php
```

Ahora la imagen está actualizada y tiene ya instalada la extensión mysqli.

No necesitamos más el container, así que lo borramos:

```bash
docker rm -v mi_app_php
```

Tenemos, por fin, una imagen preparada con las extensiones que tenemos para realizar el resto de este tutorial.

## 2. Iniciando el proyecto 

Puzzle trabaja mediante **proyectos**. Un proyecto agrupa la información, estructurada en ficheros, necesaria para crear, mantener y actualizar el despliegue de uno o varios servicios. 

Si tecleamos:

```bash
puzzle generate project puzzle_tutorial

# salida
mkdir : ~/puzzle_tutorial
mkdir : ~/puzzle_tutorial/run
mkdir : ~/puzzle_tutorial/compose 
mkdir : ~/puzzle_tutorial/dev_box
mkdir : ~/puzzle_tutorial/prod_box
``` 

Vemos como puzzle nos ha creado la siguiente estructura:

* **run**: en este directorio se produce la creación dinámica de ficheros de configuración que permite trabajar a puzzle. 
* **compose**: es el lugar donde, normalmente, colocaremos nuestros ficheros docker-compose que sirven de base para los servicios.
* **foo_box**: las boxes de piezas de puzzle. Es aquí donde se encuentra la verdadera configuración que emplea puzzle para funcionar. Por defecto, nos crea dos boxes: una de desarrollo y una de producción. Podremos, posteriormente, introducir nuevas boxes. 

## 3. El esqueleto de nuestro servicio

Todo servicio que pretenda funcionar a través de docker-compose, precisa la creación de un fichero donde se estableza la estructura del mismo, este fichero se conoce, tradicionalmente, como un fichero docker-compose. 

En este tutorial, como dijimos, vamos a desplegar un sencillo servicio con dos containers principales, uno de aplicación y uno de bbdd. Nuestro docker-compose inicial, podría quedar, entonces, como sigue:

```yaml
# docker compose

db:
    image: mysql
    volumes_from:
        - db_data       
db_data:
    image: busybox
    volumes:
        - ${RUTA_DATOS}:/var/lib/mysql

app:
    image: mi_app_php
    ports:
        - "80:80"
    volumes:
        - ${RUTA_CODIGO}:/var/www/app
    links:
        - "db:db"
```

Se articulan, como vemos, dos containers, y un tercero (un container data) para guardar los datos de mysql de forma externa y asegurar su persistencia. 

Este fichero lo almacenaremos en ~/puzzle_tutorial/compose/mi_app.yml

## 4. Controlando el servicio, nuestra primera pieza

Puzzle emplea **piezas** que son ficheros donde se establecen la configuración y las tareas a ejecutar en un determinado servicio. 
La pieza va a ser un fichero (expresado en yaml) que guardaremos en una box (en este caso, en la de desarrollo).

### A) Generación de la pieza

Ejecutamos el siguiente comando:

```bash
puzzle generate piece > ~/puzzle_tutorial/dev_box/mi_app.yml
```
Lo que nos generará un fichero con el siguiente contenido:

```yml

# compose base file
origin: ""

# interactive containers
application_containers: []

# boostrapping order [\d]
weight: 0

overrides:
    _self: {}

events: {}

tasks: {}
```
Partiendo de esta configuración base, vamos a modificarla para adaptarla a nuestras necesidades. 

### B) Configuración de la pieza

Nuestra pieza tiene que referirse a un docker-compose.yml para poder manejarlo. 

```yaml
origin: compose/mi_app.yml
```

Dentro de nuestra aplicación hay containers que son activos, se puede interactuar con ellos y deben arrancarse, y containers de datos que son pasivos. Marcamos los activos:

```yaml
application_containers: [app, db]
```

En el caso de que queramos controlar varias aplicaciones organizadas en varios containers, podemos establecerles un peso para saber su orden de arranque y de detención. 

Como en este tutorial, sólo queremos controlar una aplicación, le dejamos un peso 0.

```yaml
weight: 0
```
La sección *overrides*  es una de las secciones más importantes de la pieza, nos permite establecer variables de entorno que podrán ser compartidas por varios containers y, sobre todo, **exportadas** a containers de otra aplicación que gestionemos dentro de una box. 
En este caso, vamos a necesitar establecer como entorno, las siguientes variables:

* MYSQL_ROOT_PASSWORD: la password de administrador del sistema. 

* MYSQL_DATABASE: el nombre de la bbdd de la aplicación. 

* MYSQL_USER: el nombre del usuario empleado por la app para interactuar con el gestor de bbdd. 

* MYSQL_PASSWORD: la password de la bbdd. 

En nuestra sección overrides, esto quedará como sigue:

```yaml

overrides:
    _self: 

        db:
            MYSQL_ROOT_PASSWORD: "mi_password_secreta"

        "db + app":
            MYSQL_DATABASE: "mi_bbdd"
            MYSQL_USER: "app"
            MYSQL_PASSWORD: "app_password"
```

Estas variables han quedado establecidas en el environment de la siguiente forma:
    
* El container db tiene la variable MYSQL_ROOT_PASSWORD establecida. 

* El container db y el container app tienen las siguientes variables de entorno establecidas:
    * MYSQL_DATABASE
    * MYSQL_USER
    * MYSQL_PASSWORD

Nuestra pieza, debería quedar, entonces como sigue:

```yaml

origin: compose/mi_app.yml

application_containers: [app, db]

weight: 0

overrides:
    _self: 

        db:
            MYSQL_ROOT_PASSWORD: "mi_password_secreta"

        "db + app":
            MYSQL_DATABASE: "mi_bbdd"
            MYSQL_USER: "app"
            MYSQL_PASSWORD: "app_password"

events: {}

tasks: {}
```

Con esta estructura, ya podemos lanzar nuestro servicio. 

## 5. Gestión básica de nuestro servicio

Para arrancar nuestro servicio con puzzle, debemos definir las variables de entorno:
* RUTA_CODIGO
* RUTA_DATOS

Ahora, basta con teclear el siguiente comando:

```bash
cd ~/puzzle_tutorial; puzzle up
```

En poco tiempo, veremos como el sistema genera un docker-compose específico, que situará en la carpeta ~/puzzle_tutorial/run y lo ejecutará: se bajarán las imágenes necesarias, se arrancará el container de base de datos y, por último, nuestro container de aplicación. 

Si queremos comprobar que todo está bien, basta con teclear:

```bash
puzzle ps
```
Y veremos una lista de nuestros containers funcionando. 

Para detener nuestro servicio, bastaría con ejecutar:

```bash
puzzle down 
```

Si ahora lanzamos:

```bash
puzzle ps
```
Veremos que todos nuestros containers se han detenido. 

En la [siguiente parte](/tutorial_puzzle_2) de esta serie de tutoriales, veremos cómo crear comandos avanzados de administración de nuestro sistema. 

