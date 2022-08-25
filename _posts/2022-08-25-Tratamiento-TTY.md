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
  - Tty
  - Bash
  - Linux
---

<p align="center">
<img src="https://raw.githubusercontent.com/Wiinsad/winsad/master/assets/images/teasers/Phrase2.png">
</p>


Cuando estamos haciendo una prueba de penetracion en un aplicativo el cual tenga como base un sistema operativo **Linux** y logremos obtener una **ReversShell** mediante cualquier tecnica, es importante que por temas practicos y de comodidad hagamos un trataamiento de nuestra shell para asi tener una shell full interactive.

Uno de los problemas que nos puede causar el no hacer un tratamiento de la terminal son los siguientes:
- Una estabilidad nula.
- Al hacer la convinacion de CTRL + C la conexion morira.
- Al hacer la conbinacion de CTRL + Z se aplicara en nuestro equipo y no en el victima.
- No poder entrar a modos interactivos de aplicacions (PHP,MYSQL,SUDO,SSH,ETC.).
- No poder usar autocompletados o tabulaciones.
- No poder hacer uso de las flechas para deplazarnos entre comandos y editarlos con mas comodidad.

Como se puede ver el no hacer un tratamiento de la TTY puede generar muchos problemas de comodidad y eficioencia a la hora de estar trabajando en un sistema linux.

La siguiente tabla contiene multiples comandos los cuales nos ayudaran a spawnear una **bash** o una **sh**.

| Command | Description |
|---------| ----------- |
| script /dev/null -c bash | El comando script se utiliza para tipificar o grabar todos los procesos de la terminal. Este comando viene de base en linux asi que la mayoria de las veces es el que mejor funciona |
| python -c ‘import pty; pty.spawn(“/bin/bash”)’ | El comando con python es eficiente siempre y cuando el equipo tenga python instalado |
| python3 -c ‘import pty; pty.spawn(“/bin/bash”)’ | El comando con python3 es eficiente siempre y cuando el equipo tenga python3 instalado |
|
