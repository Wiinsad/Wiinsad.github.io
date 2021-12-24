---
title: "Watcher - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: ""
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/TryHackMe.png?raw=true"
categories:
  - Writeup
  - TryHackMe
tags:
  -
---


  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/Watcher/data/watcherTHM.png">
  </p>

  La máquina **Watcher** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **medio**  desarrollada por **USWCSS** y publicada **2 de diciembre de 2020** con un sistema operativo **Linux**.

## Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.18.102]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/Watcher/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **21,22 y 80**:  

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/Watcher/scan/Ports.png">
  </p>2021-09-24


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/Watcher/scan/PortServ.png">
  </p>
