---
title: "Active - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/Active/data/active.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Smb
  - Windows
  - Kerberoasting
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/Active/data/activeHTB.png">
</p>

La máquina **Active** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **Medium** una calificación en el rank de 5.0, **10316 USER OWNS** y **9254 SYSTEM OWNS** con la ip **10.10.10.100** y un sistema operativo **Windows**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.100]** utilizando los parámetros:
- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-n:**     No hacen la resolución del DNS.
- **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,47001,49152,49153,49154,49155,49157,49158,49169,49171,49182**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/Ports.png">
  </p>


  Ya identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/PortServ.png">
  </p>

  Con este escaneo se puede ver que la maquina cuenta con una gran cantidad de puertos abiertos lo que haré primero sera examinar el puerto **445** que pertenece a **smb** para esto usare la herramienta de **smbcliet** entrando con una **null** sesión.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/smb.png">
  </p>

  Usando la herramienta **smbmap** para verificar los permisos que tenemos sobre los recursos compartidos a nivel de **smb** se puede ver que tenemos permisos **READ** en el recurso **Replication**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/smbp.png">
  </p>

  Ya sabiendo esto haciendo una búsqueda recursiva se puede ver que existe un recurso **Groups** y en el se encuentra un archivo **Groups.xml**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/grups.png">
  </p>

  Al descargarlo y ver el contenido se puede ver una contraseña cifrada y un usuario llamado **SVC_TGS**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/cpasswd.png">
  </p>

  Si investigamos podemos llegar con un articulo el cual nos cuenta la historia de como **Microsoft** publico la key privada con la cual se cifran las contraseñas cifradas en **AES-256** lo cual nos permite descifrar estas contraseñas para verla en **plain text**(Articulo [aqui](https://adsecurity.org/?p=2288)).

  Sabiendo esto podemos usar la herramienta de **gpp-decrypt** para descifrar esta password.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/passwd.png">
  </p>

  Ya teniendo la contraseña en texto plano puedo usarla para efectuar un ataque **Kerberoasting** el cual consiste en

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/kerbe.png">
  </p>

 Gracias al ataque de **Kerberoasting** podemos ver que la cuenta del el usuario administrador es vulnerable a este ataque por lo que logramos obtener el **hash SPN** del usuario administrador.

  Para conseguir descifrar la contraseña del usuario **administrador** haré uso de la herramienta **john** para crackear el hash y obtener la contraseña en texto plano de el ticket **SPN.**

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/scan/kerbe2.png">
 </p>

 Como se puede ver la contraseña que logramos crackear es **Ticketmaster1968** y sabiendo que es del usuario **administrador** usare la herramienta de **crackmapexec** para verificar si tengo capacidad de **psexec** la cual nos permitirá acceder a la maquina como el usuario administrador.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/intrusion/crack.png">
 </p>

 Como se observa la herramienta me muestra un **"PWn3d!"** lo que significa que puedo acceder a la maquina mediante la herramienta **psexec**.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Active/intrusion/pwn.png">
 </p>
