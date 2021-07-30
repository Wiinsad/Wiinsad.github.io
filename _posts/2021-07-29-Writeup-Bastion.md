---
title: "Bastion - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/Bastion/data/Bastion.png"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Smb
  - Mount
  - Hash NTLM
  -
  - Windows
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/machines/HTB/Bastion/data/BastionHTB.png">
</p>

La máquina **Bastion** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **Easy** una calificación en el rank de 4.6, **11551 USER OWNS** y **10939 SYSTEM OWNS** con la ip **10.10.10.134** y un sistema operativo **Windows**.

## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.134]** utilizando los parámetros:
- **-p-:**    Escaneo a toda la gama de puertos (65536).
- **-n:**     No hacen la resolución del DNS.
- **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
- **--open:** Muestra sólo los puertos con un estatus abierto.
- **-oG:**    Guarda la el output en formato Grepeable.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/scan/scanPort.png">
  </p>


  Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPorts**, vemos que los puertos que destacaron en este caso fueron **22,135,139,445,5985,47001,49664,49665,49666,49667,49668,49669,49670**:

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/scan/Ports.png">
  </p>


  Ya identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - **-sC:** Script de enumeración básica de sondeo en puertos y servicios alojados.
  - **-sV:** Detección de versiones en servicios alojados en puertos.
  - **-p :** Puertos a inspeccionar.
  - **-oN:** Formatos de Nmap en los que se guardará el archivo.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/scan/PortServ.png">
  </p>

  Con este escaneo a versiones y servicios podemos ver que el servicio **smb** esta disponible y usando la herramienta de **smbmap** listo los recursos que tiene disponible el servicio haciendo uso de una **null sesion**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/scan/smbmap.png">
  </p>

  Como muestra la imagen se puede ver que el acceso con una **null session** es posible y así mismo nos lista los recursos que están disponibles, el recurso que mas me llamo la atención fue el recurso **Backups** el cual con la **null session** tengo permisos de **READ, WRITE**.


  Para poder trabajar mas cómodo haré uso de una montura del recurso **Backups** en mi equipo usando la herramienta **mount**:

  ```bash
  mount -t cifs //10.10.10.134 /mnt/smbmount -o username=null
  ```

  - **-t cifs:** tipo de devices en este caso **CIFS** que es un estándar del sistema de archivos de Windows diseñado para compartir archivos a través de Internet.
  - **-o:** Especificamos el usuario con **'username'** en este caso el username es **'null'**.

  <p align="center">
  <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/scan/mount.png">
  </p>

 Ya teniendo la montura en mi equipo y enumerando el recurso compartido de Backups yendo a la ruta ***/mnt/smbmount/WindowsImageBackup/L4mpje-PC/Backup 2019-02-22 124351*** puedo ver que existen dos archivos con extensión **.vhd**

 <p align="center">
 <img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/scan/lis.png">
 </p>


 Las extensión **.vhd** son extensiones las cuales son un formato de archivo que representa una unidad de disco duro virtual.

 Haciendo uso de la herramienta **guestmount** montare este disco virtual en mi equipo yo montare el que pesa mas ya que por lógica cuenta con mas información que puedo tomar de el.

 Ejecutando la siguiente sentencia de comando es como monto el disco virtual en mi equipo.

 ```bash
guestmount --add /mnt/smbmount/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --ro /mnt/vhd/ -m /dev/sda1
 ```

- **--add option:** es para la imagen a la que quieres acceder.
- **--ro:** se establece como de sólo lectura, alternativamente se puede usar --rw para lectura/escritura.
- **/mnt/vhd:** la ruta en la que desea montar la unidad.
- **-m /dev/sda1:** especifique qué partición dentro del archivo .vhd quiere montar.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/mount.png">
</p>

Lo que se alcanza a ver es que la unidad virtual es un sistema de archivos de Windows en el cual tenemos acceso total a toda la información por lo que si me  dirijo a la ruta ```/mnt/vhd/Windows/System32/config``` en esta ruta se encuentra la **SAM** y el **SYSTEM** y con estos dos archivos podemos dumpear los **Hashes NTLM** y con ellos crackearlos para hacer ya sea **pass the hash** o conseguir la contraseña en texto plano de el usuario.

Para esto usare la herramienta de **samdump2** pasando le el **SYSTEM** y la **SAM** para dumpear los hashes.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/hash.png">
</p>

Ya teniendo los hashes los exporto a un archivo y con la herramienta de **john** crackeo los hashes y como se puede ver la herramienta logro romper el hash de **L4mpje** el cual su contraseña es **bureaulampje**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/NT.png">
</p>

Sabiendo esto use la herramienta de **crackmapexec** para comprobar si tenia un **Pwn3d!** lo cual significaría que podría hacer **psexec** y entrar en el equipo pero en este caso no es así pero aun asi la herramienta muestra que las credenciales son validas al mostrar el **[+]** y no un **[-]**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/crack.png">
</p>

Recordando los puertos que están abiertos uno de ellos destaca el cual es el puertos **22** y utilizando las credenciales que acabo de conseguir puedo ver que son validas y de este modo entro al equipo.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/ssh.png">
</p>

Enumerando el sistema veo que la herramienta **mRemoteNG** esta instalada en el sistema, una pista de esto es yendo a la ruta **C:\Program Files (x86)** se puede ver que esta el recurso **mRemoteNG**.


```windows
l4mpje@BASTION C:\>cd "Program Files (x86)"                                                                                     

l4mpje@BASTION C:\Program Files (x86)>dir                                                                                       
 Volume in drive C has no label.                                                                                                
 Volume Serial Number is 0CB3-C487                                                                                              

 Directory of C:\Program Files (x86)                                                                                            

22-02-2019  15:01    <DIR>          .                                                                                           
22-02-2019  15:01    <DIR>          ..                                                                                          
16-07-2016  15:23    <DIR>          Common Files                                                                                
23-02-2019  10:38    <DIR>          Internet Explorer                                                                           
16-07-2016  15:23    <DIR>          Microsoft.NET                                                                               
22-02-2019  15:01    <DIR>          mRemoteNG                                                                                   
23-02-2019  11:22    <DIR>          Windows Defender                                                                            
23-02-2019  10:38    <DIR>          Windows Mail                                                                                
23-02-2019  11:22    <DIR>          Windows Media Player                                                                        
16-07-2016  15:23    <DIR>          Windows Multimedia Platform                                                                 
16-07-2016  15:23    <DIR>          Windows NT                                                                                  
23-02-2019  11:22    <DIR>          Windows Photo Viewer                                                                        
16-07-2016  15:23    <DIR>          Windows Portable Devices                                                                    
16-07-2016  15:23    <DIR>          WindowsPowerShell                                                                           
               0 File(s)              0 bytes                                                                                   
              14 Dir(s)  11.232.141.312 bytes free                                                                              

l4mpje@BASTION C:\Program Files (x86)>
```
Esta herramienta almacena credenciales cifradas en la ruta **C:\Users\*UserName*\AppData\Roaming\mRemoteNG** en el archivo **confCons.xml**.


Buscando en este archivo podemos ver que se encuentra las credenciales del usuario **Administrator** almacenadas en este archivo.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/pad.png">
</p>


Investigando llegue con una *[herramienta en github](https://github.com/haseebT/mRemoteNG-Decrypt)* la cual esta en python3 y sirve para decifrar las passwords cd mRemoteNG y usando la herramienta pude decifrar la password del usuario **Administrator**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/pas.png">
</p>

Ya teniendo la password use la herramienta **crackmapexec** comprobé las credenciales para ver si tenia capacidad de psexec y efectivamente la herramienta me mostró un **Pwn3d!**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/cram.png">
</p>

Usando la herramienta **psexec** entre al equipo con las credenciales del usuario administrador y así pudiendo ir al directorio de **Administrator** y visualizar la flag de **root**.

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/Bastion/intrusion/adm.png">
</p>
