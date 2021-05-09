---
title: "Ready - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "assets/images/machines/ready/data/ready.png"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  - Doker
  - RCE (Remote Code Execution)
  - GitLab
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/data/readyHTB.png">
</p>

La máquina **Ready** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una puntuación de 4.2, **6646 USER OWNS** y **5768 SYSTEM OWNS** con la ip **10.10.10.220** y un sistema operativo **Linux**.

### Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.220]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/scan/scanPort.png">
  </p>

  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,5080**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/scan/Ports.png">
  </p>

  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

    - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
    - **-sV:** Detección de versiones en servicios alojados en puertos.
    - **-p :** Puertos a inspeccionar.
    - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/scan/PortServ.png">
  </p>

  Lo que **nmap** nos muestra es exite un servicio ***HTTP*** en el puerto 5080 con un **robots.txt** habilitado.
  Si entramos a la pagina web vemos que es tiene como **cms** un GitLab.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/scan/page.png">
  </p>

  Ya que nos permite crear una cuenta procedo a crear la cuenta:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/scan/pageLogin.png">
  </p>


## Movimiento Lateral

  Ya dentro de la cuenta que acabo de crear enumerando un pude encontrar la versión de **GitLab** la cual es **11.4.7**, buscando con la herramienta de **searchsploit** la versión del CMS encontre que tiene varios exploits que nos permiten hacen un ataque de Remote Code Ejecution:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/intrusion/version.png">
  </p>
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/intrusion/searchsploit.png">
  </p>

  Examinando el exploit con el **id 49334** ya que pide autenticación usamos ese con las credenciales que acabamos de crear, examinando el exploit se modifico unas lineas ya que tenia algunos errores, se modifico la linea **29** ya que la variable **local_port** estaba tomando el argumento de la variable **password** y a al momento de ejecutar el esploit las credenciales se enviaban mal, tambien se modifico la linea **59** que esa linea corresponde a el payload, se le agrego el argumento **-e /bin/bash** ya que a la hora de ejecutar el exploit nos da la conexion pero no nos permite ejcutar ningun comando y a veces da error para asegurar la shell le agregamos estos parametros;

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/intrusion/expModpa.png">
  <figcaption>After.</figcaption>
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/intrusion/expModpb.png">
  <figcaption>Before.</figcaption>
  </p>
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/ready/intrusion/expModB.png">
  </p>
