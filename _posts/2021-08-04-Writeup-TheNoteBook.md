---
title: "TheNoteBook - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/TheNoteBook/data/TheNoteBook.png?raw=true"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - JSON
  -
  -
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/data/TheNoteBookHTB.png">
</p>

La máquina **TheNoteBook** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.6, **12347 USER OWNS** y **10865 SYSTEM OWNS** con la ip **10.10.10.222** y un sistema operativo **Linux**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.230]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/scanPorts.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **22 y 80**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/Ports.png">
  </p>

  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/PortServ.png">
  </p>

  Como solo hay dos puertos abierto yo enumere el servicio **http** ya que no contaba con credenciales para acceder por el puerto **22**. Entrando a la web puedo ver que existe una apartado de **login y register**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/web.png">
  </p>


  Al entrar al apartado de **register** y crear una cuenta no se puede apreciar mucho a la hora de entrar a la web, podemos crear notas y ya es todo lo que se puede hacer con la web por lo tanto intercepte una sesión con burpsuite y puede ver que la **cookie** de sesión que se genera es un formato **JSON Web Token**

  <div align="center">
  <table class="center"><tr>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/web1.png">
  <div class="caption" >Panel de Registro.</div></center></td>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/web2.png">
  <div class="caption">Web por Defecto.</div></center></td>
  </tr></table>
  </div>
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/token.png">
  </p>

  Este token lo podemos decodear en la pagina [jwp.io](https://jwt.io/).

  Al decodear el token se puede ver un apartado muy interesante el cual es **header** el cual lo que hace es llamar a una url en localhost por el puerto **7070** a un archivo **privatekey.key**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/tokenA.png">
  </p>

  Y también se puede ver que en el apartado de **PAYLOAD** del mismo token existe un apartado **"admin_cap"** el cual esta seteado a **0**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/tokenB.png">
  </p>

  Teniendo estos datos en mente podemos generar nuestro token el cual tenga instrucciones para crear una cuenta con capacidad de admin.

  Para hacer esto hay que crear unas llaves publica y privadas en **RSA** para esto se pueden hacer con la herramienta **ssh-keygen** y **openssl** esto se haria ingresando la siguiente sentencia de comandos:

  ```bash
  openssl genrsa -out privateKey.key 2048
  ```

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/tokenG.png">
  </p>

  Una vez que se tenemos las keys modificare el header del token para que apunte a un server http que levantare con python para que tome la private key desde mi equipo y no el del server.

  También seteare la variable "**admin_cap**" a 1 e ingresando las private key y public key en la web.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/tokenM.png">
  </p>

  Ya teniendo el nuevo token desde el mismo navegador podemos modificar el token yéndonos a **inpeccionar** y en el apartado de **almacenamiento** se puede ver el token en el apartado de **auth**.

  Este campo lo modificare con el nuevo token y al recargar la pagina se puede ver que se agrega a la side bar el campo **admin panel**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/1.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/2.png">
  </p>

  Se puede ver que a la hora se acceder a cualquier apartado en el servidor que monte con python la pagina hace una petición a **privKey.key** por lo que es importe mantener este servicio activo para que nuestro token siga siendo funcional.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/3.png">
  </p>


  Ahora que tenemos la capacidad de ser admin en la pagina podemos enumerar mas y si vamos al **Admin Panel** tenemos la capacidad de subir archivos.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/upload.png">
  </p>

  Si entramos a esté apartado podemos ver la siguiente interfaz.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/upload2.png">
  </p>
