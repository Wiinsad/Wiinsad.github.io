---
title: "LFI - Try Hack Me"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "/assets/images/machines/THM/LFI/data/lfi.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/TryHackMe.png?raw=true"
categories:
  - Writeup
  - TryHackMe
tags:
  - LFI
  - Sudo
---

<p align="center">
<img src="/assets/images/machines/THM/LFI/data/LFI.png">
</p>

La máquina **LFI** es una máquina virtual vulnerable de la plataforma **Try Hack Me** con un nivel de dificultad **medio**  desarrollada por **DarkStar7471** y publicada **24 de Septiembre del 2020** con un sistema operativo **Linux**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.197.4]** utilizando los parámetros:

- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-n:**     No hacen la resolución del DNS.
- **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
- **-v:**     Aumenta el nivel de los mensajes verbose.
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/scanPort.png">
</p>


Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22 y 80**:  
2
<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/Ports.png">
</p>


Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

- **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
- **-sV:** Detección de versiones en servicios alojados en puertos.
- **-p :** Puertos a inspeccionar.
- **-oN:** Formatos de Nmap en los que se guardará el archivo.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/PortServ.png">
</p>

Como vemos en el puerto **80** tiene montado un servico **http** y en el puerto **22**, entrando al servicio **80** por via web podemos ver la siguiente pagina.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/web.png">
</p>

Enumerando la web puedo ver que cuenta con articulos que al entrar a ellos te muestra lo siguiente.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/scan/article.png">
</p>


# Lateral Movement

Algo que llama mucho la atención es la **url** que hace mención a una variable **name** en el apartado de **article**, esta variable name llama **lfiname** y si cambiamos el valor por **hacking** que es otro articulo no lo muestra, algo que se puede intentar es hacer mención al **/etc/passwd** para ver si la web esta mal satinizada y podemos incluir archivos locales.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/intrusion/lfi.png">
</p>

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/intrusion/lfi2.png">
</p>

Y efectivamente podemos ver archivos locales de la maquina, una vez que vemos que tenemos acceso a archivos locales podemos probar enumerando el sistema y podemos ver que existe un usuario llamad **falconfeast** algo que podemos probar es ver si en su directorio **.ssh** tiene algunas credenciales que podemos aprovechar para entrar por ssh sabiendo que el puerto 22 esta abierto.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/intrusion/noRsa.png">
</p>

En este caso no es así pero el **/etc/passwd** se alcanza a ver en formato comentado algo que parece una key.
<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/intrusion/really.png">
</p>

Usare esta key para ver si con ellas puedo acceder por ssh.

# Privilege escalation

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/intrusion/really2.png">
</p>


Y efectivamente estas eran las credenciales del usuario **falconfeast**, ahora que ya estamos dentro de la maquina empezamos a enumerar para encontrar la forma de escalar privilegios.

Una forma seria ver los comandos que tiene el usuario **falconfeast** para ejecutar a con **sudo** sin proporcionar contraseña.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/intrusion/sudo.png">
</p>

Se puede ver que puede ejecuar el binario **/usr/bin/socat** como el usuario **root** sin propocionar contraseña.

>**¿Qué es socat?**
>
>socat significa SOcket CAT. Es una utilidad para la transferencia de datos entre dos direcciones.
>
>Lo que hace que socat sea tan versátil es el hecho de que una dirección puede representar un zócalo de red, cualquier descriptor de archivo, un datagrama de dominio Unix o un zócalo de flujo, TCP y UDP (sobre IPv4 e IPv6), SOCKS 4/4a sobre IPv4/IPv6, SCTP, PTY, zócalos de datagramas y de flujo, tuberías con y sin nombre, zócalos IP sin procesar, OpenSSL, o en Linux incluso cualquier dispositivo de red arbitrario.
>
> Mas sobre socat [Aqui](https://copyconstruct.medium.com/socat-29453e9fc8a6)

En la pagina de [gtfobins](https://gtfobins.github.io/gtfobins/socat/#sudo) se puede ver como aprovechar este binario para otorgarnos una shell como root que se haria con la siguiente secuencia de comandos.


```bash
socat stdin exec:/bin/sh
```
<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/THM/LFI/intrusion/root.png">
</p>
