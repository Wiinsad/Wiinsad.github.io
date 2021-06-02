---
title: "Unbaked Pie - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/data/image.png?raw=true"
  teaser_home_page: true
  icon: "assets/images/icons/TryHackMe.png"
categories:
  - Writeup
  - TryHackMe
tags:
  - Pivot
  - Python
  - Pickle
  - Django
---

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/PickleRick/data/pickleRickTHMpng.png?raw=true">
</p>

La máquina **Pickle Rick** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **facil**  desarrollada por **tryhackme** y publicada **10 de marzo del 2019** con un sistema operativo **Linux**.

# Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.189.24]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-Pn:**     .
  - **--min-rate 5000:** .
  - **-sS:**
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/UnbakedPie/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **5003**:  

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/UnbakedPie/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/UnbakedPie/scan/PortServ.png">
  </p>

  Realmete de este scaneno no logramos sacar mucha informacion asi que como veo que en el resultado se alcanza a ver que tiene etiquetas en **html** entro via web a la ip para ver si tiene alguna servicio http montado.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/UnbakedPie/scan/web.png">
  </p>

  Efectivamente tiene un servicio **http**, la herramieta de wappalyzer no muestra minformacion acerca de la web por que intuyo que es un servicio web simple sin gestores de contenido.

  Investigando en la pagina vemos que podemos generar un error si entramos directamente a la url **"http://[IP]:5003/search"**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/UnbakedPie/scan/error.png">
  </p>

  Entre toda la informacion se alcanza a ver que unas **cookies** se serializan con pickle ne base64, la cookie es **search_cookie**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/UnbakedPie/scan/cookie.png">
  </p>
