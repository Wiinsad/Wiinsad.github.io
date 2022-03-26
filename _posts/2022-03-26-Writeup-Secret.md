---
title: "Secret - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/secret/data/secret.png?raw=true"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Deserialization
  - Command injection
  - SUID
  - Memory Dump
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/data/secretHTB.png">
</p>

La máquina **Secret** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **facil** una puntuación de 4.2, **9384 USER OWNS** y **8159 SYSTEM OWNS** con la ip **10.10.11.120** y un sistema operativo **Linux**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.11.120]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/scanPort.png">
  </p>

  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80,3000**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/Ports.png">
  </p>

  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/PortServ.png">
  </p>

  Lo que **nmap** nos muestra es existe un servicio ***HTTP*** en el puerto **80** como en el puerto **3000**, así que para examinar la webs entrare a cada uno mediante una navegador web.

  Al entrar a las dos se puede ver que son similares en contenido visual.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/web.png">
  </p>

  Enumerando las webs se puede ver un apartado donde nos permite descargar un archivo llamado **files.zip**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/web1.png">
  </p>

  Al descargar el archivo se puede ver una estructura de archivos de una web.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/files.png">
  </p>

  Viendo que tengo el código fuente de la web inspeccionare los archivos para ver el funcionamiento de la web.
  En la carpeta de **routes** se encuentran unos archivos **js** interesantes.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/files1.png">
  </p>


  Lo que se puede ver específicamente en el archivo **private.js** es que hay un ruta llamada **"priv"** en ella se hace un chequeo del nombre el cual debe ser **theadmin** en un token y para verificar esto va al apartado archivo **verifytoken.js**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/file1.png">
  </p>

  Al ver el archivo **verifytoken** se ve que en los headers de la petición que se hace a **priv** es necesario ingresar un token el cual contiene nuestra información serializada.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/file2.png">
  </p>

  Sabiendo esto es necesario ver como se puede conseguir este token para esto enumerare la web en busca de posibles rutas que no se encuentren en los archivos que nos brindaron.

  Para eso usare la herramienta de **wfuzz** para enumerar directorios.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/web2.png">
  </p>

  Se puede ver que existe el directorio **api** y al enumerar la ruta es donde se encuentra **priv**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/web3.png">
  </p>

  A volver a ver los archivos que nos proporciono la web el archivo **auth.js** no dice que existe una ruta llamada **user** y en ella existe el apartado **register** en el cual podemos crear un usuario esto mediante **POST** el cual es necesario pasarle los parámetros **name,email,password**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/web4.png">
  </p>

  También nos muestra que existe una ruta **login** la cual nos pide como parámetro **email y password** ya que ingresamos esos datos se logea en la api y la misma genera un **auth-token** el cual es el que en archivos anteriores veíamos que se necesitaba para entrar a la ruta **priv**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/web5.png">
  </p>

  Así que para esto creare un usuario con nombre **winsad**, email **winsad@secret.com** y password **winsad123#**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/reg.png">
  </p>

  Por lo que se puede ver la api contesto y no nos marco ningún mensaje de error, visto eso me logeare en el apartado **/login** que para que la web me de el token que se genera a la hora de logearme.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/log.png">
  </p>

  Como se puede ver la api nos responde con el token que se genera pero si utilizo este token para entrar a la ruta **priv** no me permite ingresar mostrando el siguiente mensaje.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/priv.png">
  </p>

  Si trato de modificar el archivo con una herramienta de decoder como **[jwt.io](jwt.io)** en ella al ingresar el token puedo ver la información que contiene y modificarlo pero para esto necesito la **secret key* que se usa para serializar los tokens.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/token.png">
  </p>

  Así que investigando en los archivos que nos proporciono la web se puede ver que es una estructura en **git**, aprovechando esto usare una herramienta para dumpear los commits que se hicieron en el proyecto.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/git.png">
  </p>

  La herramienta que usare es **[GitTools](https://github.com/internetwache/GitTools)**, esta herramienta tiene una funcionalidad para dumpear los commits de un proyecto git, este proceso lo hice la herramienta **extractor.sh** de que viene el el repositorio antes mencionado.

  ´´´
  # ./extractor.sh local-web dump
  ´´´

  Ya que la herramienta termino de extraer todo el proyecto y los commits al entrar puedo ver que existen varios commits que se hicieron en el proyecto.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/git2.png">
  </p>

  Al ver la diferencia de los commits se puede ver que en uno de ellos hicieron un cambio en una variable llamada **"TOKEN_SECRET"** en la cual anteriormente existía una key que estaba establecida pero lo cambiaron y al usar ese secret en la herramienta para decodear y modificar el token nos permite modificarlo ya que es el token con el que se serializan los tokens en la api.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/token1.png">
  </p>

  Ya teniendo este token volveré a ingresar el token nuevo a la ruta **/priv/** para verificar que el token es correcto ahora.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/priv1.png">
  </p>

  Y como se puede ver la api responde diciéndonos **"Welcome back admin"**.

## Lateral Movement

Ahora que tenemos un token el cual la api lo interpreta como administrador gracias a sus deficit en la validación de usuario y volviendo al archivo **"private.js"** podemos ver que en la ruta **/logs** recibe como parámetro **GET** una variable llamada **file** el cual se ve que su contenido se ejecuta en un comando a nivel de sistema y analizando el comando se ve que la api no está sanitizando variable.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/priv1.png">
</p>

Sabiendo esto si yo le inyecto comando en la variable **file** puedo ejecutar comandos.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/cmd.png">
</p>

Viendo que podemos ejecutar comando en la maquina me enviare una shell a mi maquina para obtener acceso a la maquina.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/scan/shell.png">
</p>

## Privilege escalation

Ahora que tenemos acceso al sistema es la hora de buscar la forma de conseguir root en el sistema para esto enumerare archivos con permisos **SUID**:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/intrusion/uid.png">
</p>

Y se puede ver que existe un archivo en la ruta **/opt** llamado **count** el cual espera la ruta de un archivo y aplica un conteo de palabras y como es **suid** puedo leer cualquier archivo del sistema en este caso es el archivo id_rsa de **root** ell binario lo lee sin problemas.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/intrusion/uid2.png">
</p>

Examinando el archivo ya que está su version no compilada llamada **"code.c"** podemos ver que hay un apartado donde el **coredump generation** esta habilitado y seteado en **1**

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/intrusion/uid3.png">
</p>

Investigando sobre como **['trabajan los core dumps'](https://linux-audit.com/understand-and-configure-core-dumps-work-on-linux/)** de linux con archivos **suid** se puede ver que al momento de crashear el proceso en ejecución del programa es posible dumpear el contenido del archivo que estaba en uso al momento de que ocurriera el crash.

Esto es posible ya que esta seteado **prctl(PR_SET_DUMPABLE, 1);** en **1** y esto permite que los volcados del núcleo puedan ser leídos por usuarios normales y no solo root como pasaría si estuviera seteado en **2**.

Sabiendo esto hare el creasheo del proceso para poder ver el contenido de la **id_rsa** de root.

Primero crasheare el proceso.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/intrusion/crash.png">
</p>

Una vez que se crasheo el proceso ire a la ruta **/var/crash** ahí se encuentra el archivo de reporte para descomprimir el archivo de usa la herramienta **apport-unpack** para ver el contenido del reporte.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/intrusion/crash2.png">
</p>

Y efectivamente usando **Strings** en **CoreDump** se puede apreciar la **id_rsa** de root.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/intrusion/ssh.png">
</p>

Usando esta id_rsa que no esta protegida con contraseña puedo acceder a la maquina como root sin proporcionar contraseña y así visualizar la flag de root y rooteando la maquina.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/secret/intrusion/root.png">
</p>
