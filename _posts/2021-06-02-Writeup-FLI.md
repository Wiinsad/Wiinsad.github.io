---
title: "LFI - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "/assets/images/machines/THM/LFI/data/lfi.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/TryHackMe.png?raw=true"
categories:
  - Writeup
  - TryHackMe
tags:
  - LFI
  - Sudo
---

<p align="center">
<img src="/assets/images/machines/THM/LFI/data/LFI.png">
</p>

La máquina **LFI** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **medio**  desarrollada por **DarkStar7471** y publicada **24 de Septiembre del 2020** con un sistema operativo **Linux**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.197.4]** utilizando los parámetros:

- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-n:**     No hacen la resolución del DNS.
- **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
- **-v:**     Aumenta el nivel de los mensajes verbose.
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/scanPort.png">
</p>


Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22 y 80**:  
2
<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/Ports.png">
</p>


Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

- **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
- **-sV:** Detección de versiones en servicios alojados en puertos.
- **-p :** Puertos a inspeccionar.
- **-oN:** Formatos de Nmap en los que se guardará el archivo.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/PortServ.png">
</p>

Como vemos en el puerto **80** tiene montado un servico **http** y en el puerto **22**, entrando al servicio **80** por via web podemos ver la siguiente pagina.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/web.png">
</p>

Enumerando la web puedo ver que cuenta con articulos que al entrar a ellos te muestra lo siguiente.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/article.png">
</p>

Algo que llama mucho la atención es la **url** que hace mención a una variable **name** en el apartado de **article**, esta variable name llama **lfiname** y si cambiamos el valor por **hacking** que es otro articulo no lo muestra, algo que se puede intentar es hacer mención al **/etc/passwd** para ver si la web esta mal satinizada y podemos incluir archivos locales.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/lfi.png">
</p>
