---
title: "Pickle Rick - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "assets/images/machines/THM/PickleRick/data/pickleRick.png"
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

La máquina **Pickle Rick** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **medio**  desarrollada por **tryhackme** y publicada **24 de Septiembre del 2020** con un sistema operativo **Linux**.

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

  Ya que el escaneo de versiones y servicios no mostró demasiado empezamos a enumerar un poco mas a nivel web así que entre a la pagina ya que cuenta con un servicio **http** por el puerto 80, al entrar pude ver lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/web.png">
  </p>

  Curios curioso vemos que esta hablando que necesita conseguir 3 ingredientes los cuales son las **flags** que buscamos y menciona que perdió una contraseña asi que puede ser una pista para nosotros, enumerando el código fuente de la pagina se puede ver a modo comentado lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/pass.png">
  </p>

  Ahora que tenemos un usuario el cual es **R1ckRul3s** podemos pensar que puede existir un loggin así que si hacemos enumeración web básica antes de emplear un fuzzing a nivel de rutas podemos probar con el recurso **Robots.txt**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/robots.png">
  </p>

  En este caso el recurso **Robots.txt** si estaba disponible, recordar que el recursos **Robots.txt** es un recursos que nos muestra rutas existentes a nivel de directorios en las paginas web, la mayoría de las paginas lo deshabilitado ya que es una buena practica en este caso no se deshabilitado y lo podemos usar para encontrar directorios o recursos sin necesidad de hacer fuerza bruta.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/wubba.png">
  </p>

  Vemos que tiene un recurso llamado **"Wubbalubbadubdub"** pero vemos que si entramos por la pagina web a este supuesto subdirectorio nos marca como error 404 el cual indica que no existe, ya que vimos esto intentamos buscar otras formas de encontrar rutas asi que si buscamos algo clasico como "login.php" puede y encontremos algo:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/log.png">
  </p>

  Excelente ahora que tenemos un login podemos usar la credencial de usuario que encontramos en el codigo fuente y ya que lo que encontramos en Robots no fue un directorio lo usaremos como contraseña:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/webcmd.png">
  </p>

  Super bien vemos que tenemos un panel el cual se llama "Command Panel" lo que es mas que obvio que es una gran pista, ingresamos un **"whoami"** y vemos que nos responde la maquina como el usuario **"www-data"**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/PickleRick/scan/whoami.png">
  </p>

  Asi que si vemos que tenemos ejecucion de comando pues entonces nos daremos una shell:
