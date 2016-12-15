---
title: Tutorial de Puzzle (I)
keywords: puzzle, docker, stack
tags:
    - puzzle
    - docker
    - stack
layout: page
toc: false
permalink: puzzle_tut
---

# Sobre puzzle

Puzzle es una herramienta administrativa que facilita la gestión de proyectos basados en Docker mediante la creación dinámica de ficheros docker-compose.yml

# Sobre esta tutorial

Este tutorial ilustra algunas de las funcionalidades básicas de puzzle, para ello:

* Se montará un proyecto con cuatro containers:
    * Uno de php basado en la imagen oficial de [Docker Hub](https://hub.docker.com/_/php/)
    * Un container de datos para php
    * Un container de Mysql basado en la imagen oficial de [Docker Hub](https://hub.docker.com/_/mysql/)
    * Un container de datos para mysql
* Se creará una pequeña aplicación web que se comunicará con la bbdd y que, además, creará sesiones y permitirá la descarga de una serie de archivos. 

* Se mostrará la forma de administrar la aplicación en su conjunto así como los comandos básicos de administración: arranque, parado, actualización, recompilación de una parte....

* Se crearán comandos administrativos específicos y se demostrará su integración y uso mediante puzzle

## Tutorial

### Instalar puzzle

### Creación de la estructura del proyecto

Vamos a crear una serie de directorios para nuestro proyecto

```bash
mkdir ~/puzzle_tutorial; 
mkdir ~/puzzle_tutorial/composes; 
mkdir ~/puzzle_tutorial/box_dev;
 
```

### El docker-compose de base

Para poder orquestar containers, uno de los métodos básicos es el docker-compose. 

En nuestra aplicación, vamos a emplear el siguiente:

```yaml

# docker compose

app:
    image: php
    ports:
        "80:80"
    volumes:
        - ${RUTA_CODIGO}:/var/www/app
    volumes_from:
        - app_data
    links:
        - "db:db"
       
app_data:
    image: busybox
    volumes:
        - /var/www/data
     
db:
    image: mysql
    volumes_from:
        - db_data       

db_data:
    image: busybox
    volumes:
        - ${RUTA_DATOS}:/var/lib/mysql
```

Este compose básico se controla desde dos variables de entorno donde podremos establecer la ruta (del HOST) donde está nuestro código y otra donde ruta (también del HOST) donde almacenar/leer los datos de la bbdd. 

Volcamos esta información en un fichero: **~/puzzle_tutorial/compose/docker-compose.yml**


### El montaje de las piezas

Puzzle establece el concepto de **pieza** como una colección de datos (meta-datos) que permiten controlar a alto nivel uno o varios docker-compose. 

#### Generación de la pieza

Ejecutamos el siguiente comando:

```bash
puzzle generate piece ~/puzzle_tutorial/box_dev/app.yml
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
Vamos a configurarla

#### Configuración de la pieza

Nuestra pieza tiene que referirse a un docker-compose.yml para poder manejarlo. 

```yaml
origin: compose/docker-compose.yml
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

origin: compose/docker-compose.yml

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

### Arranque del proyecto

Nos colocamos en la carpeta raíz del proyecto:

```bash
cd ~/puzzle_tutorial; puzzle up --save ./run --only-build 
```

Este comando va a crear un docker compose **específico** a partir del compose de base reescrito con nuestras preferencias. 

Si hacemos

```bash
ls ~/puzzle_tutorial/run
```

Veremos que se ha creado un directorio app con el docker-compose necesario para lanzar nuestra aplicación. 

Además tenemos un puzzle.db que es una bbdd con lo necesario para REPLICAR este proyecto allí donde queramos.

#### Lanzamiento de la aplicación

Para poner la aplicación en marcha, basta con ejecutar:

```bash
puzzle up
```









