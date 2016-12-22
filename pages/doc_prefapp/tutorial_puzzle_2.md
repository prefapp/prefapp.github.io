---
title: Tutorial de Puzzle (II)
keywords: puzzle, docker, stack
tags:
    - puzzle
    - docker
    - stack
layout: page
toc: false
permalink: tutorial_puzzle_2
---

# Sobre este tutorial

Este tutorial es parte de una [serie de tutoriales](/tutorial_puzzle_1) relativos a Puzzle y su forma de uso. 

Se presupone que se ha realizado la [primera parte](/tutorial_puzzle_1) y que se ha conseguido lanzar un servicio simple con un container de aplicación y uno de mysql. 


# Tutorial II: Comandos administrativos

## 1. Gestión de backups en el container de Mysql

### A) El problema

Nuestro container de Mysql gestiona una bbdd de la que se deberían hacer copias de seguridad. Una herramienta adecuada para trabajar sería mysqldump con la que podríamos hacer respaldos periódicos de nuestra bbdd en archivos que podrían volcarse de nuevo a la bbdd en caso de ser necesario. 

Lo ideal, si se trabaja con containerización, es ejecutar el comando de dump de la bbdd desde un **container auxiliar** que comparta la configuración del de bbdd y que desaparezca una vez terminado el proceso de creación de respaldo.

Esto es, debemos evitar, en la medida de lo posible, la utilización de **docker exec** para la realización de actividades administrativas. 

Puzzle nos ayuda a ello. 

### B) El container auxiliar

Para la creación de un container auxiliar, lo primero será alterar un poco la estructura de nuestro compose de base, así editaremos el docker-compose (~/puzzle_tutorial/compose/mi_app.yml) y declararemos la estructura del mismo

```yaml
# ~/puzzle_tutorial/compose/mi_app.yml


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

#
# creamos un container auxiliar
#
db_aux:
    image: mysql
    links:
        - "db:db"
    volumes:
        - ${RUTA_DB_AUX}:/home/aux

```

La estructura declarada establece que:

1. El container parte de la imagen mysql con lo que dispone de las utilidades mysql y mysqldump
1. Tiene un volumen que comunica con una carpeta del host, así los backups se podrán copiar, mover... desde el propio host
1. Tiene acceso mediante socket INET a la bbdd






