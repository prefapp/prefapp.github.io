---
title: Tutorial de Puzzle
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

### Instalamos puzzle


### 
