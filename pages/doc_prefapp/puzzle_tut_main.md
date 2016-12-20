---
title: Tutorial de Puzzle (I)
keywords: puzzle, docker, stack
tags:
    - puzzle
    - docker
    - stack
layout: page
toc: false
permalink: puzzle_tut_main
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

puzzle generate project puzzle_tutorial

# salida
mkdir : ~/puzzle_tutorial
mkdir : ~/puzzle_tutorial/run
mkdir : ~/puzzle_tutorial/base_compose 
mkdir : ~/puzzle_tutorial/dev_box
mkdir : ~/puzzle_tutorial/dev_prod
 
```

### El docker-compose de base

Para poder orquestar containers, uno de los métodos básicos es el docker-compose. 

En nuestra aplicación, vamos a emplear el siguiente:

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
    image: php
    ports:
        - "80:80"
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
```

Este compose básico se controla desde dos variables de entorno donde podremos establecer la ruta (del HOST) donde está nuestro código y otra donde ruta (también del HOST) donde almacenar/leer los datos de la bbdd. 

Volcamos esta información en un fichero: **~/puzzle_tutorial/compose/docker-compose.yml**


### El montaje de las piezas

Puzzle establece el concepto de **pieza** como una colección de datos (meta-datos) que permiten controlar a alto nivel uno o varios docker-compose. 

#### Generación de la pieza

Ejecutamos el siguiente comando:

```bash
puzzle generate piece > ~/puzzle_tutorial/box_dev/app.yml
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

Antes de arrancar el proyecto, tenemos que establecer los valores de las dos variables de entorno (en el HOST)

- **RUTA_DATOS**: donde colocar los datos de la bbdd
- **RUTA_CODIGO**: el código de nuestra aplicación. 

Nos colocamos en la carpeta raíz del proyecto:

```bash
cd ~/puzzle_tutorial; puzzle up 
```

En este momento, han pasado muchas cosas:

1. Puzzle ha creado, en la carpeta run, una carpeta app (el nombre de la pieza)
1. En esa carpeta ha introducido un docker-compose generado a partir del compose de base (~/puzzle_tutorial/compose/docker-compose.yml) y las configuraciones de la pieza. 
1. A almacenado toda la información en una bbdd (~/puzzle_tutorial/run/puzzle.db)
1. Ha lanzado el proyecto.

Si hacemos

```bash
puzzle ps
```

Veremos la información que nos da de todos los containers y su estado. 

Además, el puzzle.db que es una bbdd con lo necesario para REPLICAR este proyecto allí donde queramos.

Si hacemos un cat del compose generado (~/puzzle_tutorial/run/app/docker-compose.yml) y nos fijamos en la sección environment:

```yaml

# seccion environment de app

app: 
  environment: 
    MYSQL_DATABASE: mi_bbdd
    MYSQL_PASSWORD: app_password
    MYSQL_USER: app

# seccion environment de db

db: 
  environment: 
    MYSQL_DATABASE: mi_bbdd
    MYSQL_PASSWORD: app_password
    MYSQL_ROOT_PASSWORD: mi_password_secreta
    MYSQL_USER: app

```

Vemos que los valores se han colocado correctamente en ambos containers de acuerdo a la distribución establecida en la pieza. En un solo punto de definición tenemos control de las variables de entorno de varios containers.


### Introducción a las tasks

Ahora tenemos dos containers relacionados entre sí (uno de bbdd y uno de aplicación), pero no tenemos código en nuestra aplicación. 

Como recordaremos, puzzle nos permite crear tareas que se ejecutan en nuestros containers para ayudar al desarrollo.

Vamos a crear un script en nuestra carpeta de código:

```php

<?php

echo("Hola \n");
echo("La base de datos se llama " . $_ENV["MYSQL_DATABASE"] . "\n");
echo("El usuario se llama " . $_ENV["MYSQL_USER"] . "\n");

```

Lo guardamos en la carpeta de código en nuestro HOST (a donde hayamos apuntado $RUTA_CODIGO).

Nos interesa ejecutarlo para probar si funciona, **pero en el contexto de nuestra aplicación**. 

Para ello, vamos a revisitar la pieza de dev_box e introducir una task:

```yaml

# ~/puzzle_tutorial/dev_box/app.yml

tasks:
    run_app:
        app:
            - php /var/www/app/dopple.php

```

Lo que acabamos de hacer es crear una **task**, que se ejecutará en el container app.

Al haber cambiado nuestra pieza, necesitamos recompilar nuestro proyecto:

```bash
puzzle up --rebuild
```

Ahora podemos lanzar la tarea

```bash
puzzle task app run_app
```

que nos dará el siguiente output:

```bash
# output

Running run_php for app
Hola mundo
La bbdd se llama 'mi_bbdd'
El usuario de acceso es 'app'
```

De esta forma, podemos desarrollar nuestra aplicación con un entorno creado para ella con todos los containers necesarios. 


#### Comandos administrativos en la bbdd

Para administrar la bbdd, lo ideal es crear un container auxiliar que realice conexiones contra nuestro container de bbdd y nos permita realizar tareas tales como:

* Importar una bbdd desde un fichero .sql
* Exportar una bbdd (backup)
* Reparar índices o tablas de bbdd


Este container auxiliar, puede estar basado en la misma imagen que la propia bbdd. 

Para crearlo, vamos a modificar nuestro docker-compose original:

```yaml

# ~/puzzle_tutorial/compose/docker-compose.

db_aux:
  image: mysql
  volumes:
    - ${RUTA_DB_AUX}:/home/aux
  links:
    - "db:db"

```

Le hemos agregado un container auxiliar que se encarga de trabajar contra la bbdd.

Ahora, podemos crear una tarea de importación de bbdd en nuestra pieza:

```yaml

# ~/puzzle_tutorial/dev_box/app.yml

tasks:
    migrar_bbdd:
        aux:
            - sh -c 'mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD --host=db $MYSQL_DATABASE < /var/lib/mysql/dopple.sql'

```

Pero, ¡cuidado! este container también necesita acceso a las credenciales de mysql. Para dárselo, basta con modificar la sección override:

```yaml


overrides:

        # ...

        "db + app + db_aux":
    
        # ...
```

Lo único que hemos hecho, es agregar ("+ db_aux") a la definición de variables de entorno, de tal manera que los containers de tipo db_aux también tengan acceso a las credenciales de mysql.

Definimos la ruta del volumen compartido con aux:

```bash
export RUTA_DB_AUX=~/db_aux
```

Y reconstruimos nuestro puzzle

```bash
cd ~/puzzle_tutorial; puzzle up --rebuild
```

Colocamos este [fichero](http://falta_fichero) en nuestra volumen compartido con db_aux y ejecutamos:

```bash
puzzle task app migrar_bbdd
```
Et voilà! Se ha creado un container de tipo db_aux (imagen mysql) que ha ejecutado una importación de la bbdd a nuestro container de mysql. 


#### Tarea de backup de bbdd

Si queremos crear una tarea de backup de bbdd, no nos será muy dificil con la estructura actual:


```yaml

# ~/puzzle_tutorial/dev_box/app.yml

tasks:

    # ...

    backup_bbdd:
        db_aux:
            - sh -c 'ahora=$(date +"%d_%m_%y"); mysqldump --user=$MYSQL_USER --password=$MYSQL_PASSWORD --host=db $MYSQL_DATABASE > "/home/aux/backup_$ahora.sql"'

```

Agregamos la nueva tarea a nuestra compilación:

```bash
puzzle up --rebuild
```

Para realizar un backup, bastaría con realizar:

```bash
puzzle task app backup_bbdd
```

Y tendríamos un backup en nuestro volumen compartido (el valor de $RUTA_DB_AUX)

Ni que decir tiene, que esto se puede meter en un cron:

```bash
05 00 * * * cd ~/puzzle_tutorial; puzzle task app backup_bbdd
```

Y tendríamos un backup de nuestra bbdd cada día a las 00:05




