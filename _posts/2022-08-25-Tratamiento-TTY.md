---
title: "Tratamiento de TTY"
layout: single
excerpt:
show_date: true
classes: wide
header:
  teaser: "https://github.com/Wiinsad/winsad/blob/master/assets/images/teasers/tty.png?raw=true"
  teaser_home_page: true
  icon: "https://raw.githubusercontent.com/Wiinsad/Wiinsad.github.io/master/assets/images/icons/cmd.jpg"
categories:
  - Informative
tags:
  - tty
  - Bash
  - Linux
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/teasers/TTY.jpg">
</p>


Cuando estamos haciendo una prueba de penetración en un aplicativo el cual tenga como base un sistema operativo **Linux** y logremos obtener una **ReversShell** mediante cualquier técnica, es importante que por temas prácticos y de comodidad hagamos un tratamiento de nuestra shell para así tener una shell full interactive.

Uno de los problemas que nos puede causar el no hacer un tratamiento de la terminal son los siguientes:

- Una estabilidad nula.
- Al hacer la combinación de CTRL + C la conexión morirá.
- Al hacer la combinación de CTRL + Z se aplicará en nuestro equipo y no en el víctima.
- No poder entrar a modos interactivos de aplicaciones (PHP,MYSQL,SUDO,SSH,ETC.).
- No poder usar autocompletados o tabulaciones.
- No poder hacer uso de las flechas para desplazarnos entre comandos y editarlos con más comodidad.

Como se puede ver el no hacer un tratamiento de la TTY puede generar muchos problemas de comodidad y eficiencia a la hora de estar trabajando en un sistema linux.

La siguiente tabla contiene múltiples comandos los cuales nos ayudaran a spawnear una **bash** o una **sh**.

| Command | Description |
|---------| ----------- |
| script /dev/null -c bash | El comando script se utiliza para tipificar o grabar todos los procesos de la terminal. Este comando viene de base en linux asi que la mayoria de las veces es el que mejor funciona |
| python -c ‘import pty; pty.spawn(“/bin/bash”)’ | El comando con python es eficiente siempre y cuando el equipo tenga python instalado |
| python3 -c ‘import pty; pty.spawn(“/bin/bash”)’ | El comando con python3 es eficiente siempre y cuando el equipo tenga python3 instalado |
| echo os.system(‘/bin/bash | Echo bash TTY shell |
| /bin/bash -i | BASH TTY shell |
| perl -e ‘exec “/bin/bash”;’ | Invocacion de una bash con el lenguaje de programacion perl |
| ruby -e ‘exec “/bin/bash”‘ | Invocacion de una bash con el lenguaje de programacion ruby |
| lua -e ‘os.execute(‘/bin/bash’)’ | Invocacion de una bash con el lenguaje de programacion lua |
| exec “/bin/bash” | Invocacion de una bash con el comando propio de UNIX exec |

Ya que pudimos spawnearnos nuestra terminal con **Bash** o **sh** el siguiente paso será hacer la combinación de teclas de:

```bash
      CTRL + Z
winsad@localhost$ ^Z
zsh: suspended  nc -nlvp 443
```

Al hacer esta combinación pondremos en segundo plano la terminal de la maquina victima en este caso, el siguiente paso es resetear la configuración de la shell que dejamos en segundo plano.

Para esto tenemos que escribir los siguientes comandos en nuestra terminal:

```bash
 ~$ stty raw -echo; fg
```

Al escribir esto estamos configurando los ajustes de la terminal y retornando a la sesión que pusimos anteriormente en espera, ahora debemos escribir el comando de **reset**, este comando dependiendo la terminal puede que no se vea a la hora de escribir o puede que sí, depende de la maquina víctima, cuando nos pregunte por el tipo de terminal se escribirá **xterm**.

```bash
[1]  + continued  nc -nlvp 443
                              reset
reset: unknown terminal type unknown
Terminal type? xterm
```

Ya que estamos dentro puede que solo requiera de hacer un enter para que salga la nueva terminal o puede que la terminal no la de inmediatamente, ahora solo queda setear los valores de las variables de entrono **TERM** y **SHELL**, esto se hace de la siguiente manera:

```bash
winsad@localhost$ export TERM=xterm
winsad@localhost$ export SHELL=bash
```

Y para que las proporciones de la terminal que acabamos de configurar sean las adecuadas a las filas y columnas de nuestro equipo debemos irnos a una terminal nueva y escribir el comando de **stty size**:

```bash
~$ ssty size
43 166
```
Ya que cuento con los valores que en mi caso fueron 43 filas y 166 columnas, solo debemos ponerlos en la terminal que estamos configurando:

```bash
winsad@localhost$ stty rows 43 columns 166
```

Y listo tenemos una **full tty interactiva** ya podemos trabajar cómodamente y sin los problemas anteriormente mencionados.
