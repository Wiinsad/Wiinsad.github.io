---
title: "Spectra - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/spectra/data/Spectra.jpg?raw=true"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  -
  -
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/data/SpectraHTB.png">
</p>

La máquina **Spectra** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de3 dificultad **Easy** una calificación en el rank de 3.5, **6899 USER OWNS** y **6626 SYSTEM OWNS** con la ip **10.10.10.229** y una clasificación de sistema operativo **Other**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.229]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/spectra/scan/ScanPorts.png?raw=true">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80 y 3306**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/PortsServ.png">
  </p>

  Viendo que namp nos mostró que cuenta con un servicio **http** en el puerto **80** hice un pequeño reconocimiento a nivel web y veo que la pagina tiene este aspecto como incompleto, al inspeccionar el código fuente puedo ver que hace mención al host **"spectra.htb/"** en varios links.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/web1.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/web2.png">
  </p>

  Con base a esto fui a mi **/etc/hosts** y asocie este dominio con la ip de la maquina.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/etecejost.png">
  </p>

  Al volver a entrar a la pagina y entrar a esos host se puede ver como pude ver que la pagina con el url **http://spectra.htb/main/** se puede ver la siguiente interfaz.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/web3.png">
  </p>

  Con la herramienta de **Wappalyzer** la cual nos enumera la pagina llegandonos a mostra gestores de contenido y lenguajes en los que estan las paginas se puede ver que la pagina esta montada sobre un **wordpress 5.4.2**, usa el lenguaje de programacion **php** y esta usando la base de datos de **MySql**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/wappa.png">
  </p>
