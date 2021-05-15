---
title: "Tenet - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/data/Tenet.png?raw=true"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  - Bash
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/tenet/data/TenetHTB.png">
</p>

 La máquina **Tenet** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.6, **5039 USER OWNS** y **4475 SYSTEM OWNS** con la ip **10.10.10.223** y un sistema operativo **Linux**.


## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.214]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/scanPort.png?raw=true">
</p>


Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80**:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/Ports.png?raw=true">
</p>


Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - -sC: Script de enumeración básica de sondeo en puertos y servicios alojados.
  - -sV: Detección de versiones en servicios alojados en puertos.
  - -p : Puertos a inspeccionar.
  - -oN: Formatos de Nmap en los que se guardará el archivo.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/PortServ.png?raw=true">
</p>

De el escaneo de puertos y servicios no pudimos recolectar mucha informacion solo que en el puerto **22** esta alojado un servicio **ssh** y en el puerto **80** esta alojado un servicio **HTTP**, asi que el camino que seguiremos sera por este serviciom entrando al servicio por el navegador se logra ve lo siguiente:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web.png?raw=true">
</p>

Un camino para poder descubrir subdirectorios es haciendo un **fuzzing** en la web en este caso usare la herramienta **gobuster**.


Haciendo el fuzzing pude descubrir que hay un wordpress corriendo en la web:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/wordpress.png?raw=true">
</p>

Al entrar a la ruta veo que la pagina se ve como si estuviera mal montada lo que puede ser un indicio de que probablemente se esta haciendo virtual hosting y entrando al código fuente de la pagina de puedo ver efectivamente que en algunas parte del código fuente se hace mención a **tenet.htb** así que agrego este host al **/etc/hosts** y al entrar volver a entrar a la pagina ya carga correctamente.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web2.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web3.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web4.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web5.png?raw=true">
</p>
