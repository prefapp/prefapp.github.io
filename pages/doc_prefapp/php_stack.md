---
title: Stack PHP
keywords: php, stack
tags:
  - php
  - stack
layout: page
toc: false
permalink: php_stack
---  

## Creación de nueva aplicación PHP

Desde el [panel de prefapp](http://panel.prefapp.es){:target=>"_blank"} nos vamos a *aplicaciones* y *nueva aplicacion*

Dentro de la sección **Stacks** escogemos la stack de PHP/MySQL

![stack_php](/images/php_stack.png)

En la pantalla resultante, vamos a configurar nuestra aplicación:

1. Para que podamos construir las imágenes necesarias para tu aplicación, primero es necesario que elegir las construcciones que necesita y configurarlas.
Para la stack php disponemos actualmente de 3 combinaciones de construcciones típicas (planos):
- php
- php y mysql
- php, redis y mysql

2. Introducimos un alias que nos resulte cómodo para identificarla y, si lo deseamos, una descripción.   
![alias_descripción](/images/alias_descripcion.png "Introducimos el alias y la descripción")

3. En la construcción **php** tenemos los siguientes parámetros configurables:   
![parametros_php](/images/parametros_php_stack.png)    
- **app:repo_url**: url del repositorio donde descargar el código de la aplicación
- **app:repo_type**: tipos de repositorios válidos: **git, svn, remote_file** (por_defecto: git)
- **app:revision**: rama, tag o commit a desplegar (por defecto: master)
- **app:credential**: (opcional) key ssh para acceso al repo en caso de ser privado
- **app:data_dir**: (opcional) path absoluto dentro del container donde se almacenarían datos que no queremos perder entre deploy y deploy de la aplicacion
- **app:extra_packages**: (opcional) lista de paquetes adicionales a instalar dentro de la imagen (paquetes de ubuntu)
- **app:extra_modules**: (opcional) lista de modulos php a instalar (pear)
- **app:migration_command**: (opcional) comando para realizar la migración de la bbdd (debe ser idempotente porque se ejecutara cada vez que se arranque con container)
- **app:postdeploy_script**: (opcional) nombre del script bash a ejecutar una vez completado el deploy
- **app:timeout**: max\_execution\_time de la aplicacion (por defecto: 60)
- **app:php_version**: version de php a usar por la aplicacion (valores válidos: 5.5, 5.6, 7.0, 7.1)
- **app:php_ini_admin_values**: hash de opciones de configuración de php necesarios para la aplicación

4. Una vez que tenemos configurada nuestra aplicación le damos a **Preparar Aplicación** y el sistema construirá las imágenes asociadas a las construcciones de la aplicación,
 y las almacenará en el registry privado.
    <aside class="notice"> 
    _OJO: El proceso de compilación de la imagen puede llevar varios minutos_
    </aside>

5. El sistema construye y almacena las imágenes necesarias para la aplicación, y una vez finalizado el proceso ya la podemos ver en la lista de aplicaciones:   
![administrar_aplicacion](/images/php_administrar_aplicacion.png "Administrando la aplicación")

6. y podemos descargarnos el [docker-compose.yml](https://docs.docker.com/compose/compose-file/) con el que desplegarla:   
![descargar_compose](/images/php_descargar_compose.png "Descargar el compose de la aplicación")   

    Con él podemos lanzar localmente la aplicación que hemos preparado mediante el siguiente comando (ejecutado en el directorio donde hemos guardado el fichero)    
```
docker-compose up
```

7. O también, podemos desplegarla en un servidor previamente registrado en el panel.
