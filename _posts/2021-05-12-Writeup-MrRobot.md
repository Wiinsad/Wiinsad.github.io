---
title: "Mr Robot - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "assets/images/machines/THM/MrRobot/data/mrrobot.png"
  teaser_home_page: true
  icon: "assets/images/icons/TryHackMe.png"
categories:
  - Writeup
  - TryHackMe
tags:
  - SUID
  - Wordpress
  - Python
  - BruteForce
---

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/data/MR-ROBOT.jpg">
  </p>

  La máquina **MrRobot** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **medio**  desarrollada por **DarkStar7471** y publicada **24 de Septiembre del 2020** con un sistema operativo **Linux**.

## Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.197.4]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **80 y 443**:  

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/PortServ.png">
  </p>

  Lo que **nmap** nos muestra es que hay un servicio **HTTP** en el puerto **80** y un servicio **HTTPS** en el puerto **443**.

  Cuando entre al servicio **HTTP** desde mi navegador pude ver lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/443.png">
  </p>

  Haciendo **fuzzing** podemos ver que cuenta con un recurso **robots.txt** y un **wp-login**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/fuzzing.png">
  </p>

  Primero iremos por el robots.txt, al entrar se pudieron ver el directorio **key-1-of-3.txt** y un recurso **fsocity.dic** el cual puedo descargar a mi máquina:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/robots.png">
  </p>

  Viendo el contenido de **fsocity.dic** puedo ver que es un diccionario con **858160** lineas:

  <div align="center">
  <table class="center"><tr>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/dic1.png">
  <div class="caption" ></div></center></td>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/dic2.png">
  <div class="caption"></div></center></td>
  </tr></table>
  </div>

## Lateral Movement
  Ya sabiendo esto me dirijo al la ruta **wp-login** ahí veo que es un login de wordpress, en algunas ocasiones los logins de wordpress te permiten enumerar usuarios ya que si ponemos uno valido nos saldría un mensaje diciendo '**ERROR: The password you entered for the username Winsad is incorrect. Lost your password?**' por ejemplo.

  Para esto y ya que no conocemos los usuarios usaremos el diccionario para hacer un ataque de fuerza bruta, para este ataque hare un script personalizado en python el cual es el siguiente:

  ```python
#!/usr/bin/python

import requests, time
from pwn import *


url = 'http://[IP Machine]/wp-login.php'

s = requests.Session()
print ''
p1 = log.progress("Fuzzeando usuario")
time.sleep(3)
p1.status("Iniciando Fuzzing")
time.sleep(3)

with open('fsocity.dic', 'r') as f:
	for i in f:

		data = {'log' : i,
			'pwd' : 'contra'
			}

		r = s.post(url, data)
		p1.status('Probando %s' % i.strip('\n'))

		if "Invalid username." not in r.text:
			p1.success('Usuario el usuario %s valido ' % i.strip('\n'))
			break
  ```

  Con el script que me acabo de desarrollar lo que hace es hacer un loggeo y comparar la respuesta por parte del servidor enviando un usuario y una contraseña cualquiera, si en ella se encuentra la cadena **"Invalid username."** significa que el usuario es valido gracias a esto pude descubrir que el usuario **Elliot** es un usuario valido:

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/fuzzU.png?raw=true">
  </p>

  Ahora que tenemos el usuario haremos una fuerza bruta a la password del usuario **Elliot**, para esto adaptaremos un poco el script para que ahora itere sobre la password y no el usuario:

  ```python
#!/usr/bin/python

import requests, time
from pwn import *

url = 'http://[Ip Machine]/wp-login.php'
user = 'Elliot'
s = requests.Session()

print ''

p1 = log.progress('Fuzzeando password')
time.sleep(2)
p1.status('Iniciando Fuzzing')
time.sleep(3)

with open('smallwordlist', 'r') as f:

	for i in f:

		data  = {
			'log' : user,
			'pwd' : i
 		}

		r = s.post(url, data)
		p1.status('Probando password %s' % i.strip('\n'))

		if 'The password you entered for the username ' not in r.text:

			p1.success('La passwor de Elliot es %s' % i.strip('\n'))
			break

  ```
  Yo acote el diccionario ya que eran demasiadas lineas, la contraseña se encontraba en la linea **5627:ER28-0652**, seria un gran punto si le metiéramos hilos pero en esta ocasión no lo hare yo.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/fuzzP.png?raw=true">
  </p>

  Ya que encontramos el usuario y la passwod vamos al login de wordpress y ingresamos con las credenciales encontradas.

  Dentro de una cuenta con permisos de administrador en wordpress es muy fácil darnos una shell los pasos serian:
  - Entrar a Appearence y en edit.
  - Editar el archivo **archive.php** ingresando la revershell en php.
  - Ponernos en escuchar por el puerto especificado en el **archive.php**.
  - Entrar a la url **https://[IP Machine]/wp-content/themes/twentyfifteen/archive.php**.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/paginaApparence.png?raw=true">
  </p>

  Lo que hacemos aquí es editar una plantilla de la web, en este caso es **archive.php** y una vez que la editamos entramos mediante el navegador y lo que pasa es que la pagina nos interpreta el contenido en php que en este caso es una revershell la cual en este caso conseguí de esta pagina en **[github](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)**.

  Ya una vez que hicimos los pasos y entramos a la url especificada podemos ver que nos da la conexión en nuestra máquina.

  Una vez dentro nos dirigimos a la rutar **/home/robot** y vemos que se encuentra la segunda flag y aparte un archivo **password.raw-md5**

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/shell.png?raw=true">
  </p>


  Si en la máquina victima hacemos un **cat password.raw-md5 \| base64** al archivo y el output que nos sale le hacemos un **echo '[output]' \| base64 -d >> hash** en nuestra máquina tenemos el mismo archivo, ya en nuestra máquina.

  Esto sirve para que a la hora de tranferirlo no se dañe el archivo se puede hacer también para binarios compilados o otros archivos ya que al codificarlo en **base64** es como si hiciéramos un **ctrl+c** y **ctrl+v** pero un poco hacker, ahora con el hash en nuestra podemos usar la herramienta de john para poder romper el hash:

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/hash1.png?raw=true">
  <div class="caption" align="center" >  <strong>Máquina victima. </strong></div>
  </p>

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/hash2.png?raw=true">
  <div class="caption" align="center"> <strong>Máquina del atacante</strong></div>
  </p>  
  Con el hash en nuestra máquina usamos la herramienta de john con el diccionario de **Rockyou** para ver si podemos romper el hash que esta en **md5**

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/crack.png?raw=true">
  </p>

  Ahora que tenemos el password lo usamos para entrar como el usuario **robot**:

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/robot.png?raw=true">
  </p>

## Privilege escalation

  Para poder convertirnos en **root** enumeramos el sistema para ver como poder hacer la escalación de privilegios.

  Usamos el comando ```find / -perm -4000 2>/dev/null``` para poder encontrar algún binario con permisos de **SUID**.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/nmap.png?raw=true">
  </p>

  Ejecutando este comando vemos que hay un uno el cual es **/usr/local/bin/nmap**, si buscamos en la pagina **[GTFOBINS](https://gtfobins.github.io)** el binario nmap podemos ver que cuenta con una forma para spawnearnos la shell si es **SUID**

  ```bash
  nmap --interactive
  nmap> !sh
  ```
  Haciendo esto podemos nos otorgaría una shell como root y ya entrar al directorio root y visualizar la ultima flag:

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/MrRobot/instrusion/root.png?raw=true">
  </p>
