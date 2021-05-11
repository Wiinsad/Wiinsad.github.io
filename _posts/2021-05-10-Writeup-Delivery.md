---
title: "Delivery - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "assets/images/machines/delivery/data/delivery.png"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  -
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/delivery/data/deliveryHTB.png">
</p>

La máquina **Delivery** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.6, **12347 USER OWNS** y **10865 SYSTEM OWNS** con la ip **10.10.10.222** y un sistema operativo **Linux**.

### Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.214]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80 y 8065**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/delivery/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/delivery/scan/PortServ.png">
  </p>


  Lo que nmap nos ha mostrado sobre los servicios es que tiene un servicio ssh y un servicio web corriendo tanto en el puerto 80 como el 8065, entrando a cada servicio web desde el navegador podemos ver los siguiete de cada uno:

  <div align="center">
  <table class="center"><tr>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/delivery/scan/web1.png">
  <div class="caption" >10.10.10.22.</div></center></td>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/delivery/scan/web2.png">
  <div class="caption">10.10.10.22:8065.</div></center></td>
  </tr></table>
  </div>
