---
title: Stack IIS - SQL Server
keywords: stack, iis, windows, sql server
tags:
    - stack
    - iis
    - windows
    - sql-server
layout: page
toc: false
permalink: iis_sql_server_stack
---

## Requisitos de sistema

El despliegue de una stack de IIS y SQL Server requiere: 

- Windows Server 2016 o 
- Windows 10

### Software necesario

Necesitamos tener instalado *Docker* y *docker-compose* en nuestra máquina:

- [Windows Server 2016](/windows_server_2016_docker)
- Windows 10 


## Creación de nuestro plano

Nos vamos al panel y nos logueamos

![loguear en panel](/images/login_panel.png)

En el panel nos vamos *aplicaciones* y *nueva aplicacion*

En el panel de selección de aplicación, escogemos *stacks* 

![selección_stack](/images/seleccion_stack.png)

Escogemos ahora la stack de Aspnet y Sql Server


![stack_iis](/images/iis_stack.png)

En la pantalla resultante, vamos a configurar nuestra aplicación:

1) Introducimos un alias que nos resulte cómodo para identificarla y, si lo deseamos, una descripción. 

![alias_descripción](/images/alias_descripcion.png "Introducimos el alias y la descripción")

2) En nuestro container de IIS podemos establecer el puerto donde se va a conectar nuestra aplicación (el puerto de la máquina de Windows), por defecto está el 80. 

Necesitamos también establecer la ruta absoluta donde vamos a colocar nuestro proyecto de asp.net, en nuestro servidor o en nuestro PC Windows. 

![puerto_codigo](/images/iis_puerto_ruta_iis.png "Colocamos el puerto y la ruta del código de nuestra aplicación")

3) En el container de SQL Server, podemos cambiar el nombre de la bbdd que vamos a crear, además deberemos establecer:

  - El nombre del fichero .mdf de la base de datos
  - El nombre del fichero .ldf de la base de datos

Ambos ficheros estarán en una carpeta de nuestro servidor o nuestro PC Windows, la ruta de dicho directorio también lo estableceremos. 


![container_bbdd](/images/iis_bbdd.png "Configurando la bbdd")


La password de administrador tiene que cumplir una serie de __requisitos__

- Tamaño de entre 8 y 15 caracteres. 
- Incluir al menos un caracter de a-z
- Incluir al menos un caracter de A-Z
- Incluir al menos un caracter de 0-9
- Incluir al menos un caracter no alfanumérico (,%.)


4) Una vez que tenemos configurada nuestra aplicación le damos a 'Preparar Aplicación'

![preparar_aplicacion](/images/preparar_aplicacion.png "Preparando la aplicación")

5) Nuestra aplicación se prepara, y ya la podemos ver en la lista de aplicaciones:

![administrar_aplicacion](/images/iis_administrar_aplicacion.png "Administrando la aplicación")

Clickamos en ella para administrarla. 

6) Nos descargamos el docker-compose.yml para poder desplegarlo:

![administrar_aplicacion](/images/iis_descargar_compose.png "Descargar el compose de la aplicación")

Tenemos ahora un [docker-compose.yml](https://docs.docker.com/compose/compose-file/) que colocamos en cualquier punto de nuestro sistema de ficheros de la máquina Windows.

7) Lanzamos la aplicación que hemos preparado mediante el siguiente comando (ejecutado en el directorio donde hemos guardado el fichero)

```powershell
docker-compose -f .\docker-compose.yml up 
```










