---
title: "Mr Robot - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "assets/images/machines/THM/MrRobot/data/mrrobot.png"
  teaser_home_page: true
  icon: "assets/images/icons/TryHackMe.png"
categories:
  - Writeup
  - TryHackMe
tags:
  -
---

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/data/MR-ROBOT.jpg">
  </p>

  La máquina **Delivery** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.6, **12347 USER OWNS** y **10865 SYSTEM OWNS** con la ip **10.10.10.222** y un sistema operativo **Linux**.

## Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.214]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22 y 443**:  

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/PortServ.png">
  </p>

  Lo que **nmap** nos muestra es que hay un servicio **ssh** en el puerto **22** y un servicio **HTTP** en el puerto **443**.

  Cuando entre al servicio **HTTP** desde mi navegador pude ver lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/443.png">
  </p>

  Haciendo **fuzzing** podemos ver que cuenta con un recurso **robots.txt** al entrar se pudieron ver el directorio **key-1-of-3.txt** y un recurso **fsocity.dic** el cual puedo descargar a mi maquina:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/robots.png">
  </p>

  Viendo el contenido de **fsocity.dic** puedo ver que es un diccionario con muchos strings variados, y con **858160** lineas:
  <div align="center">
  <table class="center"><tr>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/dic1.png">
  <div class="caption" ></div></center></td>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/dic2.png">
  <div class="caption"></div></center></td>
  </tr></table>
  </div>

  
