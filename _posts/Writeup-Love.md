---
title: "Love - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/Love/data/love.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Ssrf
  -
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/Love/data/loveHTB.png">
</p>

La máquina **love** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **EASY** una calificación en el rank de 4.4, **9403 USER OWNS** y **8307 SYSTEM OWNS** con la ip **10.10.10.239** y un sistema operativo **Windows**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.239]** utilizando los parámetros:
- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-n:**     No hacen la resolución del DNS.
- **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.


  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **80,135,139,443,445,3306,5000,5040,5985,5986,7680,47001,49664,49665,49666,49667,49668,49669,49670**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/Ports.png">
  </p>


  Ya identificados los puertos abiertos realizare un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/PortServ.png">
  </p>

  Con este escaneo a versiones y servicio no pude sacar demasiada información así que empezare a enumerar manualmente los puertos, como veo que existe el puerto **80** entrare vía web para ver el contenido de la pagina.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/web.png">
  </p>

Al entrar se puede observar que existe un panel de login pero al no conocer las credenciales no puedo hacer gran cosa y ya que las cuentas por defecto no funciona como **"admin:admin, admin:'null',administrator:administrator, etc."** lo que haré sera con la herramienta de **openssl** conectarme al puerto 443 para ver si me brinda alguna información útil.


  Para hacer esto use el siguiente comando:

```bash
openssl s_client -connect 10.10.10.239:443
```

  Al hacer esto puedo ver que entre la información que me otorga se puede ver que existe un dominio **love.htb** y un subdominio **staging.love.htb**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/sub.png">
  </p>

  Estos dominios los agregare a la ruta de **/etc/hosts**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/hosts.png">
  </p>

  Ya implementado en mi **hosts** veo si entrando a estos dominios por web puedo ver algo diferente a la web que anteriormente me mostró el puerto **80**
