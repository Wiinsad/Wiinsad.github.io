---
title: "Love - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/love/data/love.jpg"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Smb
  - Redis
  - Encryption
  - Windows
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/love/data/LoveHTB.png">
</p>

La máquina **love** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **Medium** una calificación en el rank de 2.8, **2131 USER OWNS** y **2010 SYSTEM OWNS** con la ip **10.10.10.237** y un sistema operativo **Windows**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.237]** utilizando los parámetros:
- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-Pn:**    Evita hacer ping a las maquinas a escanear.
- **--min-rate:** Envió de paquetes no mas lentos que el numero especificando.
- **-sS:**    Lanza un sondeo de tipo SYN sigiloso
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/love/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **80,135,443,445,5985,6379**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/love/scan/Ports.png">
  </p>


  Ya identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/love/scan/PortServ.png">
  </p>
