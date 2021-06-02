---
title: "Spectra - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/spectra/data/Spectra.jpg?raw=true"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  - SUID
  - Wordpress
  - Virtual Hosting
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/data/SpectraHTB.png">
</p>

La máquina **Spectra** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de3 dificultad **Easy** una calificación en el rank de 3.5, **6899 USER OWNS** y **6626 SYSTEM OWNS** con la ip **10.10.10.229** y una clasificación de sistema operativo **Other**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.229]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/spectra/scan/ScanPorts.png?raw=true">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80 y 3306**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/PortsServ.png">
  </p>

  Viendo que namp nos mostró que cuenta con un servicio **http** en el puerto **80** hice un pequeño reconocimiento a nivel web y veo que la pagina tiene este aspecto como incompleto, al inspeccionar el código fuente puedo ver que hace mención al host **"spectra.htb/"** en varios links.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/web1.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/web2.png">
  </p>

  Con base a esto fui a mi **/etc/hosts** y asocie este dominio con la ip de la maquina.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/etecejost.png">
  </p>

  Al volver a entrar a la pagina y entrar a esos host se puede ver como pude ver que la pagina con el url **http://spectra.htb/main/** se puede ver la siguiente interfaz.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/web3.png">
  </p>

## Lateral Movement

  Con la herramienta de **Wappalyzer** la cual nos enumera la pagina llegándonos a mostrar gestores de contenido y lenguajes en los que estan las paginas se puede ver que la pagina esta montada sobre un **wordpress 5.4.2**, usa el lenguaje de programación **php** y tiene como base de datos de **MySql**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/wappa.png">
  </p>

  De parte de el apartado con **/main** en la url no se encontró mucho pero haciendo **fuzzing** y un reconocimiento básico en la misma web se puede llegar a que la web cuenta con un login.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/wp-l.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/wp-lo.png">
  </p>

  De la otra parte con la url **/testing** si entrabamos via web se podía ver un apartado con los archivos que conformaban la pagina y entre ellos se alcanza a ver un archivo **wp-config.php.save**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/scan/wp.png">
  </p>

  Al entrar al a ese recurso e inspeccionando el código se puede ver unas credenciales de un usuario **devteam01** y un password **"devteam01"**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/cred.png">
  </p>

  Teniendo estas credenciales y yendo al login usando el usuario que encontramos no se puede acceder pero probando varios se puede ver que el user **administrado** es valido con esa password dándome acceso al panel de administrator de wordpress.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/panel.png">
  </p>

  Una vez que ya se tiene acceso al panel de administrador de un gestor de contenido es fácil poder acceder a la maquina que esta montando la pagina web. En este caso para un wordpress los pasos son los siguientes:

  - Entrar **Appearence - Theme Editor**.
  - Seleccionar en el apartado de **Select theme to edit:** el tema **Twenty Seventeen**
  - Buscar y editar el archivo **404 Template** con un revershell en php (Puedes usar un revershell de php que esta en mi pagina de github la shell pertenece a **[pentestmonkey](https://github.com/Wiinsad/Pentest/blob/main/Shell/reverShell.php)**).
  - Agregar tu **IP** y el **PORT** en la shell y guardar el cambio dándole a **Update File**.
  - Dirigirte a la siguiente url ```http://spectra.htb/main/wp-content/themes/twentyseventeen/404.php``` (en este caso la url es así puede variar) y estar en paralelo ponerte en escucha por el puerto especificado.
  - Listo estas dentro.


  Ya dentro puedes hacer un tratamiento de la **tty** para poder moverte como en una shell interactiva pudiendo hacer **Ctrl+c**, regresar al comando anterior con las flechas, tabular, etc.

  Una de las formas para hacer el tratamiento de la **tty** se hace con los siguientes pasos:
  - **script /dev/null -c bash**
  - **Ctrl + z** (te regresara a tu shell)
  - **stty raw -echo; fg** (en tu shell)
  - **reset**
  - **xterm**
  - **export TERM=xterm**
  - **export SHELL=bash**
  - En tu maquina con otra consola aparte y la consola en pantalla completa escribe **stty -a**
  - En la maquina victima ingresas **stty rows [Tu numero de rows] columns [Tu numero de columns]**

  Y listo tendríamos una full **stty** interactiva.


  Continuando como vemos ya tengo acceso al sistema enumerando el sistema veo que estoy como el usuario **nginx** y que hay un total de 4 usuarios incluyendo root en el sistema.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/passwd.png">
  </p>

  Enumerando el sistema si vamos a la ruta **/opt** podemos ver un archivo llamado **"autologin.conf.orig"** vemos que el archivo lo que hace es inyectar unas keys en la ruta **/etc/autologin** y si vamos a esa ruta podemos ver las credenciales de el usuario **katie** en la maquina no se puede usar el comando **"su"** para poder ingresar como otro usuario asi que las credenciales las use en **ssh** aprovechando que estaba abierto.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/pass.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/ssh.png">
  </p>

## Privilege escalation
  Ya dentro y enumerando el sistema podemos ver que el usuario **katie** puede ejecutar como sudo sin otorgar password el **/sbin/initctl**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/sudo.png">
  </p>

  Ahora que vemos que tenemos este permiso podemos aprovecharnos injertando código malicioso usando el **initctl** esto se haría de la siguiente forma. Primer tendríamos que dirigirnos en la ruta **/etc/init** y verificar el estatus de los servicios que están corriendo, esto se haría usando:

  ```bash
  sudo -u root /sbin/initctl list | ls -la | grep test
  ```
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/perm.png">
  </p>

  Una vez que identificamos un archivo test que tengamos permisos para editar en este caso podemos editar el **test.conf**, usaremos este para inyectar un script malicioso el cual seria el siguiente:

  ```bash
  start on filesystem or runlevel [2345]
  stop on shutdown

  script
  chmod +s /bin/bash
  end script
  ```

  Lo que haría el siguiente script es otorgar permisos **SUID** a la **/bin/bash** y ya que nosotros estamos ejecutando el script con permisos sudo y el propietario de la **/bin/bash** es root entonces podríamos escalar privilegios de esta forma.
  Una vez que modificamos el archivo para ejecutar el script seria con el siguiente comando:

  ```bash
  sudo -u root /sbin/initctl start test
  ```
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/preroot.png">
  </p>
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/spectra/intrusion/root.png">
  </p>

  Como vemos el script otorgo el permiso **SUID** a la **/bin/bash** y usando el parametro **-p** nos otorga la shell como el usuario root.
