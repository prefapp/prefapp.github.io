---
title: Instalación de Docker y docker-compose en MS Server 2016
keywords: instalación, docker, docker-compose, server 2016, windows
tags:
    - instalación
    - docker
    - windows
    - docker-compose
layout: page
toc: false
permalink: windows_server_2016_docker
---

## Instalación de Docker

Se emplea el módulo [OneGet provider PowerShell Module](https://github.com/oneget/oneget) para realizar la instalación. 

Instalamos el OneGet:

```bash
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force

```

Mediante el OneGet instalamos Docker

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider

```

Tenemos que reiniciar el sistema para poder utilizar el software instalado:

```
Restart-Computer -Force
```

### Actualizar el sistema 

Para asegurarnos que el sistema está actualizado una vez instalado el software de Docker, ejecutamos:

```powershell
sconfig
```
Ahora, elegimos:

1. Opción 6) 'Download and Install Updates' *y*
1. Opción A) 'dowload All updates'


##### Información adicional

[Windows Containers on Windows Server](https://msdn.microsoft.com/virtualization/windowscontainers/quick_start/quick_start_windows_server)

## Instalación de docker-compose

Para instalar *docker-compose* utilizamos [Chocolatey](https://chocolatey.org/) que es un proyecto open source para manejar paquetería en Windows.

Instalamos chocolatey (Desde Cmd.exe):

```powershell
@powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
    
```

Si experimentáis algún problema con la instalación, id a esta [página](https://chocolatey.org/install)

Ahora podemos instalar docker-compose:

```powershell
choco install -y docker-compose  
``` 

##### Información adicional
- [Getting started with Windows and Docker](https://stefanscherer.github.io/get-started-with-docker-on-windows-using-chocolatey/)
- [Información sobre docker-compose](https://docs.docker.com/compose/)






