---
layout: single
title: Bandit - Over The Wire
excerpt: "Lo que haremos en esta ocasión, será resolver todos los niveles que se encuentran en la sección **Bandit**, esto como práctica de pentesting en entornos Linux."
date: 2024-07-10
classes: wide
header:
  teaser: /assets/images/otw-writeups/overthewire-logo.jpg
  teaser_home_page: true
  icon: /assets/images/otw-writeups/hacker.png
categories:
  - OverTheWire
  - Bandit
tags:
  - Linux
  - Comandos Linux
---
<p align="center">
<img src="/assets/images/otw-writeups/overthewire-logo2.jpg">
</p>

Lo que haremos en esta ocasión, será resolver todos los niveles que se encuentran en la sección **Bandit**, esto como práctica de pentesting en entornos Linux.

--------------
**RECOMENDACIÓN**:

Guarda las contraseñas que vayas encontrando, por si vas haciendo notas o por si continuaras haciendo niveles en otro dia o simplemente por si las dudas.


<br>
<hr>
<div id="Indice">
	<h1>Índice</h1>
	<ul>
		<li><a href="#Nivel0">Nivel 0 a 1</a></li>
			<ul>
				<li><a href="#Exp0">Descripción de Comandos</a></li>
				<ul>
					<li><a href="#ssh">Comando ssh</a></li>
					<li><a href="#whoami">Comando whoami</a></li>
					<li><a href="#ls">Comando ls</a></li>
					<li><a href="#cat">Comando cat</a></li>
				</ul>
				<li><a href="#Sol0">Solución</a></li>
			</ul>
		<li><a href="#Nivel1">Nivel 1 a 2</a></li>
			<ul>
                                <li><a href="#Exp1">Descripción de Comandos</a></li>
				<ul>
                                        <li><a href="#pwd">Comando pwd</a></li>
					<li><a href="#sustitucion">Sustitución de Comandos</a></li>
				</ul>
                                <li><a href="#Sol1">Solución</a></li>
                        </ul>			
                <li><a href="#Nivel2">Nivel 2 a 3</a></li>
			<ul>
                                <li><a href="#Sol2">Solución</a></li>
                        </ul>
                <li><a href="#Nivel3">Nivel 3 a 4</a></li>
			<ul>
                                <li><a href="#Exp3">Descripción de Comandos</a></li>
				<ul>
                                        <li><a href="#cd">Comando cd</a></li>
					<li><a href="#find">Comando find</a></li>
					<li><a href="#grep">Comando grep</a></li>
					<li><a href="#xargs">Comando xargs</a></li>
					<li><a href="#tuberias">Tuberias - Pipes</a></li>
				</ul>
                                <li><a href="#Sol3">Solución</a></li>
                        </ul>
                <li><a href="#Nivel4">Nivel 4 a 5</a></li>
			<ul>
                                <li><a href="#Exp4">Descripción de Comandos</a></li>
				<ul>
                                        <li><a href="#file">Comando file</a></li>
                                </ul>
                                <li><a href="#Sol4">Solución</a></li>
                        </ul>
                <li><a href="#Nivel5">Nivel 5 a 6</a></li>
			<ul>
				<li><a href="#Exp5">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#head">Comando head</a></li>
                                </ul>
                                <li><a href="#Sol5">Solución</a></li>
                        </ul>
                <li><a href="#Nivel6">Nivel 6 a 7</a></li>
			<ul>
                                <li><a href="#Sol6">Solución</a></li>
                        </ul>
                <li><a href="#Nivel7">Nivel 7 a 8</a></li>
			<ul>
				<li><a href="#Exp7">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#awk">Comando awk</a></li>
					<li><a href="#cut">Comando cut</a></li>
					<li><a href="#rev">Comando rev</a></li>
					<li><a href="#tr">Comando tr</a></li>
					<li><a href="#tail">Comando tail</a></li>
                                </ul>
                                <li><a href="#Sol7">Solución</a></li>
                        </ul>
                <li><a href="#Nivel8">Nivel 8 a 9</a></li>
			<ul> 
                                <li><a href="#Exp8">Descripción de Comandos</a></li>
				<ul>
                                        <li><a href="#sort">Comando sort</a></li>
					<li><a href="#uniq">Comando uniq</a></li>
                                </ul>
                                <li><a href="#Sol8">Solución</a></li>
                        </ul>
                <li><a href="#Nivel9">Nivel 9 a 10</a></li>
			<ul> 
                                <li><a href="#Exp9">Descripción de Comandos</a></li>
				<ul>
                                        <li><a href="#strings">Comando strings</a></li>
                                </ul>
                                <li><a href="#Sol9">Solución</a></li>
                        </ul>
                <li><a href="#Nivel10">Nivel 10 a 11</a></li>
			<ul> 
                                <li><a href="#Exp10">Descripción de Comandos</a></li>
				<ul>
                                        <li><a href="#base64">Comando base64</a></li>
                                </ul>
                                <li><a href="#Sol10">Solución</a></li>
                        </ul>
                <li><a href="#Nivel11">Nivel 11 a 12</a></li>
			<ul>
                                <li><a href="#Sol11">Solución</a></li>
                        </ul>
		<li><a href="#Nivel12">Nivel 12 a 13</a></li>
                        <ul> 
                                <li><a href="#Exp12">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#xxd">Comando xxd</a></li>
					<li><a href="#7z">Comando 7z</a></li>
					<li><a href="#gzip">Comando gzip</a></li>
					<li><a href="#bzip2">Comando bzip2</a></li>
					<li><a href="#tar">Comando tar</a></li>
					<li><a href="#sponge">Comando sponge</a></li>
                                </ul>
                                <li><a href="#Sol12">Solución</a></li>
				<ul>
					<li><a href="#descompresion1">Primera Forma de Descompresión con 7z</a></li>
					<li><a href="#descompresion2">Segunda Forma de Descompresión con gzip, bzip2 y tar</a></li>
				</ul>
                        </ul>
		<li><a href="#Nivel13">Nivel 13 a 14</a></li>
                        <ul> 
                                <li><a href="#Exp13">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#llaves">Llave Privada y Pública de Servicio SSH</a></li>
                                </ul>
                                <li><a href="#Sol13">Solución</a></li>
                        </ul>
		<li><a href="#Nivel14">Nivel 14 a 15</a></li>
                        <ul> 
                                <li><a href="#Exp14">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#nc">Comando nc</a></li>
                                </ul>
                                <li><a href="#Sol14">Solución</a></li>
                        </ul>
		<li><a href="#Nivel15">Nivel 15 a 16</a></li>
                        <ul> 
                                <li><a href="#Exp15">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#ncat">Comando ncat</a></li>
                                </ul>
                                <li><a href="#Sol15">Solución</a></li>
                        </ul>
		<li><a href="#Nivel16">Nivel 16 a 17</a></li>
                        <ul> 
                                <li><a href="#Exp16">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#chmod">Comando chmod</a></li>
					<li><a href="#mktemp">Comando mktemp</a></li>
					<li><a href="#nmap">Comando nmap</a></li>
                                </ul>
                                <li><a href="#Sol16">Solución</a></li>
                        </ul>
		<li><a href="#Nivel17">Nivel 17 a 18</a></li>
                        <ul> 
                                <li><a href="#Exp17">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#diff">Comando diff</a></li>
                                </ul>
                                <li><a href="#Sol17">Solución</a></li>
                        </ul>

		<li><a href="#Nivel18">Nivel 18 a 19</a></li>
                        <ul> 
                                <li><a href="#Exp18">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#sshpass">Comando sshpass</a></li>
                                </ul>
                                <li><a href="#Sol18">Solución</a></li>
                        </ul>
		<li><a href="#Nivel19">Nivel 19 a 20</a></li>
                        <ul> 
                                <li><a href="#Exp19">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#suid">Permiso SUID</a></li>
                                </ul>
                                <li><a href="#Sol19">Solución</a></li>
                        </ul>
		<li><a href="#Nivel?">Nivel 20 a 21</a></li>
                        <ul> 
                                <li><a href="#Exp?">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#"></a></li>
                                </ul>
                                <li><a href="#Sol?">Solución</a></li>
                        </ul>
		<li><a href="#Nivel?">Nivel ? a ?</a></li>
                        <ul> 
                                <li><a href="#Exp?">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#"></a></li>
                                </ul>
                                <li><a href="#Sol?">Solución</a></li>
                        </ul>
		<li><a href="#Nivel?">Nivel ? a ?</a></li>
                        <ul> 
                                <li><a href="#Exp?">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#"></a></li>
                                </ul>
                                <li><a href="#Sol?">Solución</a></li>
                        </ul>
		<li><a href="#Nivel?">Nivel ? a ?</a></li>
                        <ul> 
                                <li><a href="#Exp?">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#"></a></li>
                                </ul>
                                <li><a href="#Sol?">Solución</a></li>
                        </ul>
		<li><a href="#Nivel?">Nivel ? a ?</a></li>
                        <ul> 
                                <li><a href="#Exp?">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#"></a></li>
                                </ul>
                                <li><a href="#Sol?">Solución</a></li>
                        </ul>
		<li><a href="#Nivel?">Nivel ? a ?</a></li>
                        <ul> 
                                <li><a href="#Exp?">Descripción de Comandos</a></li>
                                <ul>
                                        <li><a href="#"></a></li>
                                </ul>
                                <li><a href="#Sol?">Solución</a></li>
                        </ul>
		<li><a href="#Links">Links de Investigación</a></li>
	</ul>
</div>


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel0" style="text-align:center;">Nivel 0 a 1</h1>
</div>
<br>


**INSTRUCCIONES**:

El objetivo de este nivel es que inicies sesión en el juego conectandote al **servicio SSH** que tienen activo. El host al que debes conectarte es **bandit.labs.overthewire.org**, en el **puerto 2220**. El nombre de usuario es **bandit0** y la contraseña es **bandit0** (esto mismo te lo menciona la descripción del nivel). Una vez que hayas iniciado sesión, ve a la página del siguiente nivel (Nivel 0 a Nivel 1) para averiguar cómo entrar al Nivel 1.

La contraseña para el siguiente nivel se almacena en un archivo llamado **readme** ubicado en el directorio de inicio del nivel 0. Usa esta contraseña para iniciar sesión en **bandit1** usando **SSH**. 

Siempre que encuentres una contraseña para un nivel, usa el **servicio SSH** (en el **puerto 2220**) para iniciar sesión en ese nivel y continuar el juego.

---------

<br>

<h2 id="Exp0">Descripción de Comandos Usados</h2>

Estos son los comandos que usamos en este nivel:

<br>

<h3 id="ssh">Comando ssh</h3>

| **Comando ssh** |
|:-----------:|
| *Se utiliza para establecer conexiones remotas seguras entre dos computadoras a través de una red. Permite a los usuarios acceder y administrar remotamente otro sistema como si estuvieran trabajando localmente en él.* |

Forma de uso:
```bash
ssh usuario@dirección_ip_o_hostname
```

Ejemplos:
* Conectandose al **servicio SSH**:
```bash
ssh berserk@10.10.10.10
```

* Especificando un puerto:
```bash
ssh berserk@10.10.10.10 -p 22
```

<br>

<h3 id="whoami">Comando whoami</h3>

| **Comando whoami** |
|:-----------:|
| *Se utiliza para mostrar el nombre del usuario actual con el que se ha iniciado sesión en el sistema. Es una combinación de las palabras "who am I" (¿quién soy?), y es útil para confirmar la identidad del usuario en la sesión actual, especialmente cuando se manejan múltiples cuentas o se cambia de usuario con su o sudo.* |

Forma de uso:
```bash
whoami
usuario_actual
```

Ejemplos:
* Obteniendo el usuario actual:
```bash
whoami
root
```

<br>

<h3 id="ls">Comando ls</h3>

| **Comando ls** |
|:-----------:|
| *Se utiliza para listar el contenido de un directorio. Es uno de los comandos más básicos y comunes, y muestra los archivos y subdirectorios dentro del directorio en el que te encuentras o en el que especifiques.* |

Forma de uso:
```bash
ls
```

Ejemplos:
* Listando archivos y subdirectorios del directorio actual
```bash
ls
```

* Listando archivos y subdirectorios de un directorio anterior:
```bash
ls ../
```

* Listando archivos y subdirectorios de un directorio especifico:
```bash
ls /home
```

* Listando archivos y subdirectorios del directorio actual, incluyendo los ocultos
```bash
ls -a
```

* Listando archivos y subdirectorios del directorio, incluyendo los permisos, fecha y hora de creación para cada uno de estos:
```bash
ls -l
```

<br>

<h3 id="cat">Comando cat</h3>

| **Comando cat** |
|:-----------:|
| *Se utiliza para concatenar y mostrar el contenido de archivos en la terminal. Es muy versátil y comúnmente se emplea para leer rápidamente el contenido de archivos de texto.* |

Formas de uso:
```bash
cat archivo_a_leer
```

Ejemplos:
* Leyendo un archivo:
```bash
cat archivo.txt
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol0">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit0@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit0@bandit.labs.overthewire.org's password:
```
Recuerda que la contraseña es **bandit0**

Es muy simple, la flag la encuentras solamente usando el comando **ls**:
```bash
bandit0@bandit:~$ whoami
bandit0
bandit0@bandit:~$ ls
readme
```

Y ya solo debes leer ese archivo:
```bash
bandit0@bandit:~$ cat readme
...
```
¡Listo! Ya tenemos la contraseña para el nivel 1. Para salir, solo usa el comando **exit**. Vayamos al siguiente nivel.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel1" style="text-align:center;">Nivel 1 a 2</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en un archivo llamado **-** ubicado en el directorio de inicio.

----------------

<br>

<h2 id="Exp1">Descripción de Comandos</h2>

<br>

<h3 id="pwd">Comando pwd</h3>

| **Comando pwd** |
|:-----------:|
| *Muestra la ruta completa del directorio de trabajo actual, es decir, el directorio en el que te encuentras en ese momento en la terminal. El comando pwd es útil para verificar tu ubicación en el sistema de archivos, especialmente cuando navegas entre múltiples directorios.* |

Forma de uso:
```bash
pwd
```

Ejemplos:
* Mostrando directorio actual:
```bash
pwd
/home/berserk/Desktop
```

<br>

<h3 id="sustitucion">Sustitución de Comandos</h3>

| **Sustitución de Comandos** |
|:-----------:|
| *La sintaxis $() en Bash se utiliza para ejecutar un comando y capturar su salida. El resultado del comando que se ejecuta dentro de $() se reemplaza en línea por la salida de ese comando.* |

Forma de uso:
```bash
Comando $(comando_A_usar)
```

Ejemplos:
* Usando un comando dentro de un **string** de **echo**:
```bash
echo "Estoy en: $(pwd)"
Estoy en: /home/berserk/Desktop
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol1">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit1@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit1@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 0**.

Ya nos indicaron donde se encuentra la contraseña para el siguiente nivel.

El problema es qué el nombre del archivo es un simple guion y si intentamos leerlo, nos dará un error:
```bash
bandit1@bandit:~$ whoami
bandit1
bandit1@bandit:~$ ls
-
bandit1@bandit:~$ cat -
^C
```

Ni me saco un error, por eso tuve que cancelar la acción. Para resolver esto, **Over The Wire** nos da una página con una solución:
* <a href="https://www.webservertalk.com/dashed-filename" target="_blank">Dashed Filename – Learn How to Create, Remove, List, Read & Copy!</a>

En resumen, para leer el archivo, simplemente usamos el siguiente signo **<**. 

Intentémoslo:
```bash
bandit1@bandit:~$ cat < -
...
```
¡Muy bien! Ya tenemos la contraseña para el siguiente nivel. 

Antes de ir a otro nivel, hay otras formas para poder ver esta clase de archivos, te las mostrare a continuación:

* Leer el archivo usando el **PATH**:
```bash
bandit1@bandit:~$ cat /home/bandit1/-
...
```

* Utilizar otros caracteres para leerlo
```bash
bandit1@bandit:~$ cat ./-
...
```

* Usando el comando **pwd**:
```bash
bandit1@bandit:~$ cat $(pwd)/-
...
```

Ahora si ya terminamos, sal y entra al siguiente nivel.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel2" style="text-align:center;">Nivel 2 a 3</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en un archivo llamado **spaces**, ubicado en el directorio de inicio.

----------------

<h2 id="Sol2">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit2@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit2@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 1**.

Ya sabemos que hay un archivo llamado **spaces** que tiene la contraseña. 

Vamos a verlo:
```bash
bandit2@bandit:~$ whoami
bandit2
bandit2@bandit:~$ ls
spaces in this filename
```

Para leerlo, simplemente escribe las primeras letras del nombre del archivo y oprime la tecla **TAB** para que automáticamente te rellene el nombre del archivo:
```bash
bandit2@bandit:~$ cat spaces

bandit2@bandit:~$ cat spaces\ in\ this\ filename
...
```

Hay otras formas de poder leerlo, veamos cuales son:

* Leyendo todos los archivos que empiecen por **s**:
```bash
bandit2@bandit:~$ cat s*
...
```

* Usando el **PATH** para leer todos los archivos que empiecen en **s**:
```bash
bandit2@bandit:~$ cat /home/bandit2/s*
...
```

* Leyendo todos los archivos dentro de un **PATH**:
```bash
bandit2@bandit:~$ cat /home/bandit2/*
...
```

* Usando el comando **pwd**:
```bash
bandit2@bandit:~$ cat $(pwd)/*
...
```

¡Excelente! Ya tenemos la contraseña del siguiente nivel, sal y entra en el que sigue.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel3" style="text-align:center;">Nivel 3 a 4</h1>
</div>
<br>



**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en un archivo oculto en el directorio **inhere**.

----------------

<br>

<h2 id="Exp3">Descripción de Comandos</h2>

<br>

<h3 id="cd">Comando cd</h3>

| **Comando cd** |
|:-----------:|
| *Se utiliza para cambiar el directorio de trabajo actual en la terminal. Es la abreviatura de "change directory".* |

Formas de uso:
```bash
cd nombre_directorio
```

Ejemplos:
* Moviendonos al un directorio **/home**:
```bash
cd
```

* Moviendonos a un directorio anterior:
```bash
cd ../
```

* Moviendonos a el directorio raiz:
```bash
cd /
```

* Moviendonos a un directorio especifico:
```bash
cd /home/Usuario/Desktop
```

<br>

<h3 id="find">Comando find</h3>

| **Comando find** |
|:-----------:|
| *Se utiliza para buscar archivos y directorios dentro de un sistema de archivos, según criterios específicos como nombre, tipo, fecha de modificación, tamaño, entre otros.* |

Formas de uso: 
```bash
find nombre_archivo_a_buscar
```

Ejemplos:
* Buscando un archivo en el directorio actual:
```bash
find archivo.txt
```

* Buscar un archivo entre todos los archivos del directorio actual:
```bash
find .
```

* Buscar un archivo en todo el sistema:
```bash
find /
```

* Buscando un archivo dentro de un directorio especifico y especificando su nombre:
```bash
find /home/usuario/Desktop --name "archivo.txt"
```

* Buscando un archivo de acuerdo a su tipo:
```bash
find . -type f -> archivo normal
find . -type d -> directorio
```

* Buscando un archivo por sus permisos:
```bash
find . -perm 666 
```

* Buscando un archivo por su tamaño:
```bash
find . -size 1500c -> bytes
find . -size 1500k -> kilobytes
find . -size 1500M -> megabytes
find . -size 1500G -> gigabytes
```

<br>

<h3 id="grep">Comando grep</h3>

| **Comando grep** |
|:-----------:|
| *Se utiliza para buscar patrones específicos de texto dentro de archivos o en la salida de otros comandos. Es ideal para encontrar líneas que contienen ciertas palabras o expresiones regulares.* |

Formas de uso: 
```bash
grep parametros
```

Ejemplos:
* Buscando una palabra dentro de un archivo:
```bash
grep "Hola" archivo.txt
```

* Buscando una cadena de texto dentro de un archivo:
```bash
grep -F "Cadena de texto a buscar" archivo.txt
```

* Mostrando el número de linea/s en donde se encuentra una cadena de texto o una palabra:
```bash
grep -n "Cadena de texto a buscar" archivo.txt
```

* Buscando una palabra sin importar que sea con mayusculas o minusculas:
```bash
grep -i "hola" archivo.txt
```

<br>

<h3 id="xargs">Comando xargs</h3>

| **Comando xargs** |
|:-----------:|
| *Se utiliza para construir y ejecutar comandos a partir de la salida de otros comandos. Toma la salida de un comando (normalmente una lista de elementos) y la usa como argumentos para otro comando.* |

Formas de uso: 
```bash
find archivo.txt | xargs comando
```

Ejemplos:
* Leyendo un archivo que se encontro con el comando **find**:
```bash
find . -type f | xargs cat
```

* Eliminando un archivo encontrado con el comando **find**:
```bash
find . --name "*.tmp" | xargs rm
```

<br>

<h3 id="tuberias">Tuberias - Pipes</h3>

| **Tuberías - Pipes** |
|:-----------:|
| *Representadas por el símbolo "\|", se utilizan para conectar la salida de un comando directamente como entrada para otro comando. Esto permite encadenar varios comandos de manera eficiente y procesar datos en una secuencia.* |

Formas de uso: 
```bash
comando | comando
```

Ejemplos:
* Imprimiendo contenido de un archivo y buscando una palabra:
```bash
cat archivo.txt | grep "contraseña"
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol3">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit3@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit3@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 2**.

Es muy sencillo, primero veamos qué hay:
```bash
bandit3@bandit:~$ whoami
bandit3
bandit3@bandit:~$ ls
inhere
bandit3@bandit:~$ cd inhere/
```

Entonces, el archivo que tiene la contraseña está almacenado en un archivo oculto, para verlo, agregamos los parámetros **-la** al comando **ls**:
```bash
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Apr 23 18:04 .
drwxr-xr-x 3 root    root    4096 Apr 23 18:04 ..
-rw-r----- 1 bandit4 bandit3   33 Apr 23 18:04 .hidden
```

Y ya solo vemos el archivo con **cat**:
```bash
bandit3@bandit:~/inhere$ cat .hidden
...
```

En este punto, podemos jugar un poco con los comandos que nos muestra el nivel:

* Usando **find y xargs** para imprimir el contenido de los archivos en el directorio actual:
```bash
find . -type f | xargs cat
```

* Buscando e imprimiendo un archivo que tenga el nombre **"hidden"**:
```bash
find . -type f | grep "hidden" | xargs cat
```

¡Listo! Ya tenemos la contraseña para el siguiente nivel, sal y entra en el siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel4" style="text-align:center;">Nivel 4 a 5</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el único archivo legible por humanos en el directorio **inhere**. Consejo: si su terminal está desordenada, intente con el comando **"reset"**.

----------------

<br>

<h2 id="Exp4">Descripción de Comandos</h2>

<br>

<h3 id="file">Comando file</h3>

| **Comando file** |
|:-----------:|
| *Se utiliza para determinar el tipo de archivo de uno o más archivos especificados. En lugar de basarse únicamente en la extensión del archivo, file analiza el contenido del archivo para identificar su tipo.* |

Formas de uso:
```bash
fiel archivo
```

Ejemplos:
* Mostrando que tipo de archivo es un archivo elegido:
```bash
file archivo.txt
archivo.txt: ASCII text
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol4">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit4@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit4@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 3**.

Nos dice que la contraseña, está en una archivo que solo lo pueden leer los humanos...bueno, veamos qué hay dentro:
```bash
bandit4@bandit:~$ ls
inhere
bandit4@bandit:~$ cd inhere/
bandit4@bandit:~/inhere$ ls -la
total 48
drwxr-xr-x 2 root    root    4096 Apr 23 18:04 .
drwxr-xr-x 3 root    root    4096 Apr 23 18:04 ..
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file00
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file01
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file02
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file03
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file04
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file05
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file06
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file07
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file08
-rw-r----- 1 bandit5 bandit4   33 Apr 23 18:04 -file09
```

Mmmmm, los permisos indican que no se pueden ni leer con **cat**, a menos que los ejecutemos como programas. 

Entonces, veamos qué tipo de archivos son con el comando **file**:
```bash
bandit4@bandit:~/inhere$ file ./-file*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: Non-ISO extended-ASCII text, with no line terminators
```

También pudimos hacerlo combinando varios comandos como el comando **find, grep y xargs**:
```bash
bandit4@bandit:~/inhere$ find . -type f | grep "\-file" | xargs file
./-file08: data
./-file02: data
./-file09: data
./-file01: data
./-file00: data
./-file05: data
./-file07: ASCII text
./-file03: data
./-file06: data
./-file04: data
```

Solamente hay un archivo que es de texto, vamos a verlo:
```bash
bandit4@bandit:~/inhere$ cat ./-file07
...
```
¡Excelente! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel5" style="text-align:center;">Nivel 5 a 6</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel, se almacena en un archivo en algún lugar del directorio **inhere** y tiene todas las siguientes propiedades:
* legible por humanos
* 1033 bytes de tamaño
* no ejecutable

----------------

<br>

<h2 id="Exp5">Descripción de Comandos</h2>

<br>

<h3 id="head">Comando head</h3>

| **Comando head** |
|:-----------:|
| *El comando head en Linux se utiliza para mostrar las primeras líneas de un archivo o la salida de otro comando. Por defecto, head muestra las primeras 10 líneas de un archivo de texto, pero puedes modificar esta cantidad utilizando opciones adicionales.* |

Formas de uso:
```bash
head [opciones] [archivo]
```

Ejemplos:
* Mostrando las primeras 10 lineas de un archivo:
```bash
head archivo.txt
```

* Mostrando las primeras 5 lineas de un archivo:
```bash
cat archivo.txt | head -n 5
```

* Mostrando la primera linea de un archivo:
```bash
grep "archivo1.txt" | xargs cat | head -n 1
```

* Mostrando los primeros 100 bytes de un archivo:
```bash
head -c 100 archivo.txt
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol5">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit5@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit5@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 4**.

Hay varias condiciones que tiene el archivo que contiene la contraseña, para buscarlo, usaremos el comando **find** con varios argumentos. 

Te comparto unos links con ejemplos de como usar el comando **find** y sobre los permisos de archivos:
* <a href="https://itsfoss.com/es/comando-find-linux/#buscar-varios-archivos-con-varias-extensiones-o-condici%C3%B3n" target="_blank">15 ejemplos súper útiles del comando Find en Linux</a>
* <a href="https://www.hostinger.mx/tutoriales/como-usar-comando-find-locate-en-linux/#Busqueda_por_tipo" target="_blank">Cómo usar los comandos find y locate en Linux</a>
* <a href="https://gospelidea.com/blog/que-son-los-permisos-chmod" target="_blank">¿Qué son los permisos CHMOD en Linux?</a>

Bien, ahora entremos:
```bash
bandit5@bandit:~$ ls
inhere
bandit5@bandit:~$ cd inhere/
bandit5@bandit:~/inhere$ ls -la
total 88
drwxr-x--- 22 root bandit5 4096 Apr 23 18:04 .
drwxr-xr-x  3 root root    4096 Apr 23 18:04 ..
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere00
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere01
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere02
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere03
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere04
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere05
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere06
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere07
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere08
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere09
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere10
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere11
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere12
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere13
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere14
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere15
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere16
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere17
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere18
drwxr-x---  2 root bandit5 4096 Apr 23 18:04 maybehere19
```

Demasiados directorios, prodríamos buscar nuestro archivo, pero eso no es óptimo, así que usemos el comando **find**. 

Le agregaremos los siguientes parámetros:
* *-type f*: Para que busque archivos.
* *-size 1033c*: Para que busque los archivos con **1033 bytes** de tamaño. 
* *-perm 640*: Para que busque por archivos con permisos de no ejecución.
* *grep ASCII*: Para que con **grep**, busque archivos que sean legibles por humanos.

Quedaría así:
```bash
bandit5@bandit:~/inhere$ find . -type f -size 1033c -perm 640 | grep ASCII
./maybehere07/.file2
```

Ahora, veamos el contenido de ese archivo:
```bash
cat maybehere07/.file2
...
```

También podiamos obtener el resultado de manera directa:
```bash
bandit5@bandit:~/inhere$ find . -type f ! -executable -size 1033c | xargs cat | head -n 1
```

¡Listo! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel6" style="text-align:center;">Nivel 6 a 7</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en algún lugar del servidor y tiene todas las siguientes propiedades:
* propiedad del usuario bandit7
* propiedad del grupo bandit6
* 33 bytes de tamaño

----------------

<br>

<h2 id="Sol6">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit6@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit6@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 5**.

Hay varias condiciones, igual usaremos el comando **find** y usemos las mismas páginas de referencia que puse en el nivel anterior para guiarnos. 

Aunque me sirvió más esta página:
* <a href="https://www.ionos.mx/digitalguide/servidores/configuracion/comando-linux-find/" target="_blank">Linux find comando</a>

Veamos que hay dentro:
```bash
bandit6@bandit:~$ ls
bandit6@bandit:~$ ls -la
total 20
drwxr-xr-x  2 root root 4096 Apr 23 18:04 .
drwxr-xr-x 70 root root 4096 Apr 23 18:05 ..
-rw-r--r--  1 root root  220 Jan  6  2022 .bash_logout
-rw-r--r--  1 root root 3771 Jan  6  2022 .bashrc
-rw-r--r--  1 root root  807 Jan  6  2022 .profile
```

No hay nada, entonces vamos a buscar en todo el servidor como nos mencionan, usemos el comando **find**. Para buscar en todo el servidor, usamos un **/** en vez de un **.** :
```bash
bandit6@bandit:~$ find / -type f -user bandit7 -group bandit6 -size 33c
find: ‘/var/log’: Permission denied
find: ‘/var/crash’: Permission denied
find: ‘/var/spool/rsyslog’: Permission denied
find: ‘/var/spool/bandit24’: Permission denied
find: ‘/var/spool/cron/crontabs’: Permission denied
...
```

Salieron muchos, entonces vamos a redirigir los errores al **/dev/null** para que solo nos muestre los resultados correctos:
```bash
bandit6@bandit:~$ find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
```

Ahora veamos el contenido de ese archivo:
```bash
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
...
```
¡Listo! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel7" style="text-align:center;">Nivel 7 a 8</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo **data.txt** junto a la palabra **millionth**.

----------------

<br>

<h2 id="Exp7">Descripción de Comandos</h2>

<br>

<h3 id="awk">Comando awk</h3>

| **Comando awk** |
|:-----------:|
| *awk es un potente lenguaje de procesamiento de texto y un comando en Linux que se utiliza para manipular y extraer datos de archivos o entradas de texto. Se especializa en procesar texto estructurado en columnas o campos separados por delimitadores (como espacios o comas).* |

Formas de uso:
```bash
awk 'patron {accion_a_realizar} archivo'
```

Ejemplos:
* Mostrando la segunda columna **($2)** de un archivo:
```bash
awk '{print $2}' archivo.txt
```

* Utilizando un delimitador distinto (una coma):
```bash
awk -F',' '{print $1, $3}' archivo.csv
```

<br>

<h3 id="cut">Comando cut</h3>

| **Comando cut** |
|:-----------:|
| *cut es una herramienta más simple que se utiliza para extraer partes de líneas de texto en función de posiciones de caracteres, bytes o delimitadores. Es útil cuando necesitas extraer columnas específicas o secciones de texto de un archivo.* |

Formas de uso:
```bash
cut [opciones] archivo
```

Ejemplos:
* Extrayendo la segunda columna de un archivo delimitado por tabulaciones:
```bash
cut -f 2 archivo.txt
```

* Extrayendo la primera y tercera columna de un archivo delimitado por comas:
```bash
cut -d ',' -f 1,3 archivo.csv
```

<br>

<h3 id="rev">Comando rev</h3>

| **Comando rev** |
|:-----------:|
| *El comando rev en Linux se utiliza para invertir el orden de los caracteres en cada línea de un archivo o entrada de texto. Es una herramienta muy sencilla y directa.* |

Formas de uso:
```bash
rev [archivo]
```

Ejemplos:
* Invirtiendo el contenido de un **string**:
```bash
echo "Hola mundo" | rev
```

* Invirtiendo la lineas de un archivo
```bash
rev archivo.txt
```

<br>

<h3 id="tr">Comando tr</h3>

| **Comando tr** |
|:-----------:|
| *El comando tr (translate) se utiliza para traducir, reemplazar o eliminar caracteres de un flujo de texto. Es muy útil para operaciones simples de sustitución de caracteres o eliminación de espacios y saltos de línea.* |

Formas de uso:
```bash
tr [opciones] set1 [set2]
```

Ejemplos:
* Convirtiendo minúsculas a mayúsculas:
```bash
echo "hola mundo" | tr 'a-z' 'A-Z'
```

* Eliminando espacios en blanco:
```bash
echo "Hola   Mun do" | tr -d ' '
```

* Reemplazando espacios por guiones:
```bash
echo "Hola Mundo" | tr ' ' '-'
```

* Comprimiendo multiples espacios en uno solo:
```bash
echo "Hola      Mundo" | tr -s ' '
```

<br>

<h3 id="tail">Comando tail</h3>

| **Comando tail** |
|:-----------:|
| *El comando tail en Linux se utiliza para mostrar las últimas líneas de un archivo o la salida de otro comando. Por defecto, muestra las últimas 10 líneas, pero puedes modificar este comportamiento con diferentes opciones.* |

Formas de uso:
```bash
tail [opciones[ [archivo]
```

Ejemplos:
* Mostrando las 10 últimas lineas de un archivo:
```bash
tail archivo.txt
```

* Mostrando las 20 últimas lineas de un archivo:
```bash
tail -n 20 archivo.txt
```

* Mostrando los últimos 100 bytes de una archivo:
```bash
tail -c 100 archivo.txt
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol7">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit7@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit7@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 6**.

Como nos dice, hay un archivo llamado **data.txt** y la contraseña se encuentra junto a la palabra **millionth**, para encontrar esa palabra usaremos el comando **grep**, aquí te dejo este link con información muy útil sobre **grep**:
* <a href="https://geekland.eu/uso-del-comando-grep-en-linux-y-unix-con-ejemplos/" target="_blank">Uso del comando grep en Linux y UNIX con ejemplos</a>

Veamos que contiene ese archivo:
```bash
bandit7@bandit:~$ ls
data.txt
bandit7@bandit:~$ cat data.txt 
Worcester's     fyKdWWh7VVgusiIKPygHJe6TlkDHhLHl
arousal r8mfBurE2OvHu8NFQc7mJ2x14iNjwkin
counterespionage's      4jmzYqFkqwciprPrJleFCI9tyjbXBtdt
Willard's       ctbhPNPRDGAll4Whhsrz3Mwv6qJHM8Et
midwife Kk9VZkoTUNUfmIa031vovUN2UKksZ56S
...
```

El archivo tiene mucho texto, ahora usemos **grep** de tal forma que busque la palabra **"millionth"** de manera exacta (usaremos el parámetro **-w** para eso y complementamos con **-E**):
```bash
bandit7@bandit:~$ grep -E -w 'millionth' data.txt
millionth		...
```
Y ahí está. 

Vamos a complicarnos un poquito para utilizar otros comandos que nos pueden servir más adelante.

Utilicemos el **comando awk** para encontrar y obtener directamente la contraseña.

Te dejo estos links de apoyo:
* <a href="https://www.shortcutfoo.com/app/dojos/awk/cheatsheet" target="_blank">AWK Cheatsheet</a>
* <a href="https://bl831.als.lbl.gov/~gmeigs/scripting_help/awk_cheat_sheet.pdf" target="_blank">AWK cheat sheets</a>

Probemos:
* Extrayendo el último campo de cada linea del archivo con **awk**:
```bash
bandit7@bandit:~$ cat data.txt | grep "millionth" | awk 'NF{print $NF}'
...
```

* Obteniendo la segunda columna con **awk**
```bash
bandit7@bandit:~$ cat data.txt | grep "millionth" | awk '{print $2}'
...
```

Ahora probemos con el **comando cut**:

* Extrayendo la segunda columna y eliminando espacios con **cut**:
```bash
cat data.txt | grep "millionth" | cut -d ' ' -f 2
...
```

Combinación con el **comando rev y awk**:
```bash
bandit7@bandit:~$ cat data.txt | grep "millionth" | rev | awk '{print $1}' | rev
...
```

Obteniendo contraseña en combinación con el **comando tr y tail**:
```bash
bandit7@bandit:~$ cat data.txt | grep "millionth" | xargs | tr ' ' '\n' | tail -n 1
...
```

Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel8" style="text-align:center;">Nivel 8 a 9</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo **data.txt** y es la única línea de texto que aparece una sola vez.

----------------

<br>

<h2 id="Exp8">Descripción de Comandos</h2>

<br>

<h3 id="sort">Comando sort</h3>

| **Comando sort** |
|:-----------:|
| *Se utiliza para ordenar las líneas de texto de un archivo o la entrada estándar. Puede ordenar alfabéticamente, numéricamente, y en orden ascendente o descendente, entre otras opciones.* |

Formas de uso:
```bash
sort archivo.txt
```

Ejemplos:
* Ordenando un archivo:
```bash
sort archivo.txt
```

* Ordenando el contenido de un archivo de manera inversa:
```bash
sort -r archivo.txt
```

* Ordenando y eliminando palabras duplicados:
```bash
sort -u archivo.txt > archivo_sin_duplicados.txt
```

<br>

<h3 id="uniq">Comando uniq</h3>

| **Comando uniq** |
|:-----------:|
| *Se utiliza para filtrar líneas repetidas consecutivas en un archivo o entrada estándar. Generalmente se combina con sort para eliminar duplicados no consecutivos.* |

Formas de uso:
```bash
sort archivo.txt | unique
```

Ejemplos:
* Mostrando lineas unicas dentro de un archivo:
```bash
sort archivo.txt | unique
```

* Mostrando lineas que aparecen solamente una vez dentro de un archivo:
```bash
sort archivo.txt | unique -u
```

* Mostrando lineas que aparecen al menos 2 veces dentro de un archivo:
```bash
sort archivo.txt | unique -d
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol8">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit8@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit8@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 7**.

Hay que buscar una línea de texto que sea única de todas las que hay en el archivo **data.txt**, para esto, usaremos los comandos **sort** y **uniq**.

Primero, veamos el contenido:
```bash
bandit8@bandit:~$ ls
data.txt
bandit8@bandit:~$ cat data.txt 
QWiiBJhqUoMj0lCD9XNrkTM1M94eIPMV
UkKkkIJoUVJG6Zd1TDfEkBdPJptq2Sn7
ITQY9WLlsn3q168qH29wYMLQjgPH9lNP
JddNHIO2SAqKPHrrCcL7yTzArusoNwrt
0dEKX1sDwYtc4vyjrKpGu30ecWBsDDa9
...
```

El contenido del archivo es demasiado, ahora usemos los comandos:
```bash
bandit8@bandit:~$ sort data.txt | uniq -u
...
```

¡Exacto! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel9" style="text-align:center;">Nivel 9 a 10</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo **data.txt**, en una de las pocas cadenas legibles por humanos, precedida por varios caracteres **'='**.

----------------

<br>

<h2 id="Exp9">Descripción de Comandos</h2>

<br>

<h3 id="strings">Comando strings</h3>

| **Comando strings** |
|:-----------:|
| *Se utiliza para extraer y mostrar las cadenas de texto legible (ASCII o Unicode) que se encuentran dentro de archivos binarios o cualquier archivo no textual. Es útil para inspeccionar archivos binarios, como ejecutables, en busca de contenido de texto que pueda ser relevante.* |

Formas de uso:
```bash
strings archivo.txt
```

Ejemplos:
* Mostrando las cadenas legibles de un archivo:
```bash
strings archivo.bin
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol9">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit8@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit8@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 8**.

Tendremos que ver los permisos del archivo y que contenga más de un signo **=**. El problema es que el archivo está compuesto de solo data:
```bash
bandit9@bandit:~$ file data.txt 
data.txt: data
```
Lo que tendremos que hacer, será convertir la data a strings, para que sea un texto legible y luego buscar la línea de texto que tenga más de 2 signos **"=="**.

Hagámoslo:
```bash
strings data.txt | grep "==="
4========== the#
========== password
========== is
========== *****
```
¡Exacto! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel10" style="text-align:center;">Nivel 10 a 11</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo data.txt, que contiene datos codificados en base64.

----------------

<br>

<h2 id="Exp10">Descripción de Comandos</h2>

<br>

<h3 id="base64">Comando base64</h3>

| **Comando base64** |
|:-----------:|
| *Se utiliza para codificar y decodificar datos en el esquema de codificación Base64, que convierte datos binarios en texto ASCII. Es comúnmente usado para transmitir datos binarios en formatos que solo admiten texto, como en correos electrónicos o URLs.* |

Formas de uso:
```bash
base64 archivo.txt > archivo_en_base64.txt
```

Ejemplos:
* Convirtiendo el contenido de un archivo a **base64**:
```bash
base64 archivo.txt > archivo_en_base64.txt
```

* Convirtiendo un texto en **base64**:
```bash
echo "Hola como estas?" | base64
```

* Decodificando un archivo en **base64** y un texto en **base64**:
```bash
base64 -d archivo_en_base64.txt > archivo.txt
echo "SG9sYSBjb21vIGVzdGFzPwo=" | base64 -d
```

<br>

**Continuamos con la solución.**

<br>

<h2 id="Sol10">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit10@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit10@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 9**.

Si bien el archivo está en **base64**, lo único que debemos hacer, será decodificar esa **base64**. Lo haremos con el comando **base64** y el parámetro **-d** que sirve para decodificar esa base:
```bash
bandit10@bandit:~$ ls
data.txt
bandit10@bandit:~$ file data.txt 
data.txt: ASCII text
bandit10@bandit:~$ cat data.txt 
VGhlIHBhc3N3b3JkIGlzIDZ6UGV6aUxkUjJSS05kTllGTmI2blZDS3pwaGxYSEJNCg==
```

Como puedes observar, el hash pareciera que es la contraseña, pero no es así.

Ahora, apliquemos el comando:
```bash
bandit10@bandit:~$ base64 -d data.txt 
The password is *****
```

¡Muy bien! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Nivel11" style="text-align:center;">Nivel 11 a 12</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo data.txt, donde todas las letras minúsculas (a-z) y mayúsculas (A-Z) se han rotado 13 posiciones.

----------------

<br>

<h2 id="Sol11">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit11@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       
                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit11@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 10**.

Para resolver esto, usaremos el comando **tr**.

Entonces, con **tr** vamos a mover todas las letras a donde estaban originalmente. Primero, veamos como quedaron las letras:
* Cada letra se movió 13 posiciones, es decir, la letra **a**, ya no es la **a** sino otra letra.
* Si miras el alfabeto, cuenta de la letra a 13 posiciones, debería quedar en la letra **m**. Debes contar desde A a Z, no te saltes la A.
* Esto es muy similar al cifrado de cesar, en el que se rotaban las letras, de acuerdo a una clave para que el mensaje quede encriptado.
* También se le conoce como ROT13.

Ahora sí, hagámoslo:
```bash
bandit11@bandit:~$ ls
data.txt
bandit11@bandit:~$ cat data.txt 
Gur cnffjbeq vf WIAOOSFzMjXXBC0KoSKBbJ8puQm5lIEi
bandit11@bandit:~$ file data.txt 
data.txt: ASCII text
bandit11@bandit:~$ tr 'A-Za-z' 'N-ZA-Mn-za-m' < data.txt 
The password is
```

Si te fijas, lo que hicimos fue sustituir **'A-Za-z'** por **'N-ZA-Mn-za-m'**, en donde:
* *A-Z = N-ZA-M*
* *a-z = n-za-m*

¡Muy bien! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel12" style="text-align:center;">Nivel 12 a 13</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo data.txt, que es un **hexdump** de un archivo que ha sido comprimido repetidamente. Para este nivel puede ser útil crear un directorio bajo **/tmp** en el que puedas trabajar. Utilice **mkdir** con un nombre de directorio difícil de adivinar. O mejor, utilice el comando **"mktemp -d"**. Luego copia el archivo de datos usando **cp**, y renómbralo usando mv (¡lee las páginas de manual!)

----------------
<br>

<h2 id="Exp12">Descripción de Comandos</h2>

<br>

<h3 id="xxd">Comando xxd</h3>

| **Comando xxd** |
|:-----------:|
| *El comando xxd en Linux se utiliza para crear un volcado hexadecimal de un archivo o de la entrada estándar. También se puede usar para convertir un volcado hexadecimal de vuelta a su formato binario original.* |

Formas de uso:
```bash
xxd [opciones] [archivo]
```

Ejemplo:
* Mostrando contenido de un archivo en formato hexadecimal:
```bash
xxd archivo.txt
```

* Creando un volcado hexadecimal de un archivo en formato simple (solo bytes):
```bash
xxd -p archivo.txt
```

* Convertir un volcado hexadecimal de vuelta a binario:
```bash
xxd -r archivo.txt > archivoBin
```

<br>

<h3 id="7z">Comando 7z</h3>

| **Comando 7z** |
|:-----------:| 
| *El comando 7z es parte de la utilidad 7-Zip, que se utiliza para comprimir y descomprimir archivos con múltiples formatos. Es muy eficiente para comprimir grandes archivos y puede manejar varios formatos, como 7z, zip, tar, gzip, entre otros.* |

Formas de uso: 
```bash
7z [opciones] archivo
```

Ejemplo:
* Comprimiendo un archivo o directorio en formato **7z**:
```bash
7z a archivo.7z archivo.txt
```

* Listando el contenido de un archivo comprimido:
```bash
7z l archivo.7z
```

* Descomprimiendo un archivo **7z**:
```bash
7z x archivo.7z
```

<br>

<h3 id="gzip">Comando gzip</h3>

| **Comando gzip** |
|:-----------:| 
| *El comando gzip se utiliza para comprimir archivos en Linux usando el algoritmo de compresión DEFLATE. El archivo original se reemplaza por uno con la extensión gz.* |

Formas de uso: 
```bash
gzip [opciones] [archivo]
```

Ejemplo:
* Comprimiendo un archivo:
```bash
gzip archivo.txt
```

* Descomprimiendo un archivo:
```bash
gzip -d archivo.gz
```

* Comprimiendo todos los archivos en un directorio de forma recursiva:
```bash
gzip -r /ruta/al/directorio
```

<br>

<h3 id="bzip2">Comando bzip2</h3>

| **Comando bzip2** |
|:-----------:| 
| *El comando bzip2 se utiliza para comprimir archivos utilizando el algoritmo Burrows-Wheeler y la codificación Huffman. Los archivos comprimidos con bzip2 tienen la extensión bz2. Este comando suele generar archivos más pequeños que gzip, pero puede ser un poco más lento.* |

Formas de uso: 
```bash
bzip2 [opciones] [archivo]
```

Ejemplo:
* Comprimiendo un archivo:
```bash
bzip2 archivo.txt
```

* Descomprimiendo un archivo:
```bash
bzip2 -d archivo.bz2
```

* Comprimiendo un archivo sin eliminar el original:
```bash
bzip2 -k archivo.txt
```

<br>

<h3 id="tar">Comando tar</h3>

| **Comando tar** |
|:-----------:| 
| *El comando tar se utiliza para agrupar varios archivos y carpetas en un solo archivo, generalmente con la extensión tar. Aunque no comprime archivos por sí solo, es común utilizarlo junto con herramientas de compresión como gzip o bzip2 para crear archivos comprimidos .tar.gz o .tar.bz2.* | 

Formas de uso: 
```bash
tar [opciones] [archivo_tar] [archivos_o_directorios]
```

Ejemplo:
* Creando un archivo **tar** (sin comprimir):
```bash
tar -cvf archivo.tar archivo1.txt archivo2.txt
```

* Extrayendo un archivo **tar**:
```bash
tar -xvf archivo.tar
```

<br>

<h3 id="sponge">Comando sponge</h3>

| **Comando sponge** |
|:-----------:|
| *El comando sponge es parte de la colección moreutils y se utiliza para leer toda la entrada estándar y luego escribirla en un archivo, permitiendo que se sobrescriba el archivo de entrada. Lo que lo hace especial es que espera a leer completamente la entrada antes de escribir, lo cual evita problemas que ocurren cuando intentas sobrescribir un archivo mientras lo estás leyendo.* |

Formas de uso:
```bash
comando | sponge [archivo]
```

Ejemplo:
* Sobreescribiendo un archivo con contenido modificado:
```bash
sort archivo.txt | sponge archivo.txt
```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol12">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit12@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit12@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 11**.

Esta parte es un poco tediosa, como dicen las instrucciones, tenememos un archivo que ha sido comprimido varias veces, por lo que debemos descomprimirlo varias veces hasta encontrar el archivo que contiene la contraseña.

Vamos a hacerlo de 2 maneras:
* La primera será utilizando los comandos **xxd, sponge y 7z**.
* La segunda será utilizando los comandos **xxd, gzip, b2zip, y tar**.

<br>

<h3 id="descompresion1">Primera Forma de Descompresión con 7z</h3>

Vamos a copiar el archivo en nuestra máquina, tan solo usa **cat** para ver el contenido del archivo y copialo en tu máquina.

Una vez copiado, vamos a convertir el archivo hexadecimal en un archivo binario y usamos **sponge** para evitar errores:
```bash
cat data.txt | xxd -r | sponge data
ls
data.txt data
```

Puedes analizar el archivo con el comando **file**, cambia su nombre para ponerle la **extensión gz**:
```bash
file data
data: gzip compressed data, was "data2.bin"...
mv data data.gz
```

También podemos listar el contenido de este archivo con **7z**:
```bash
7z l data.gz
Scanning the drive for archives:
1 file, 607 bytes (1 KiB)
*
Listing archive: data.gz
--
Path = data.gz
Type = gzip
Headers Size = 20
*
   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2023-05-25 01:08:15 .....          574          607  data2.bin
------------------- ----- ------------ ------------  ------------------------
2023-05-25 01:08:15                574          607  1 files
```

Por último, solamente debemos descomprimir todos los archivos resultantes hasta obtener un archivo de texto, que lo identificaras porque es un archivo tipo **ASCII**, a continuación solamente pondre los archivos obtenidos y su descompresión:

* Primer descompresión:
```bash
7z x data.gz
Extracting archive: data.gz
--
Path = data.gz
Type = gzip
Headers Size = 20
Everything is Ok
Size:       574
Compressed: 607
*
file data2.bin
data2.bin: bzip2 compressed data, block size = 900k
```

* Segunda descompresión:
```bash
7z x data2.bin
Extracting archive: data2.bin
--
Path = data2.bin
Type = bzip2
Everything is Ok
Size:       432
Compressed: 574
*
file data2
data2: gzip compressed data, was "data4.bin"..
```

* Tercera descompresión:
```bash
7z x data2
Extracting archive: data2
--
Path = data2
Type = gzip
Headers Size = 20
Everything is Ok
Size:       20480
Compressed: 432
*
file data4.bin
data4.bin: POSIX tar archive (GNU)
```

* Cuarta descompresión:
```bash
7z x data4.bin
Extracting archive: data4.bin
--
Path = data4.bin
Type = tar
Physical Size = 20480
Headers Size = 10240
Code Page = UTF-8
Everything is Ok
Size:       10240
Compressed: 20480
*
file data5.bin
data5.bin: POSIX tar archive (GNU)
```

* Quinta descompresión:
```bash
7z x data5.bin
Extracting archive: data5.bin
--
Path = data5.bin
Type = tar
Physical Size = 10240
Headers Size = 9728
Code Page = UTF-8
Everything is Ok
Size:       221
Compressed: 10240
*
file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
```

* Sexta descompresión:
```bash
7z x data6.bin
Extracting archive: data6.bin
--
Path = data6.bin
Type = bzip2
Everything is Ok
Size:       10240
Compressed: 221
*
file data6
data6: POSIX tar archive (GNU)
```

* Septima descompresión:
```bash
7z x data6
Extracting archive: data6
--
Path = data6
Type = tar
Physical Size = 10240
Headers Size = 9728
Code Page = UTF-8
Everything is Ok
Size:       79
Compressed: 10240
*
file data8.bin
data8.bin: gzip compressed data, was "data9.bin"...
```

* Octava descompresión:
```bash
7z x data8.bin
Extracting archive: data8.bin
--
Path = data8.bin
Type = gzip
Headers Size = 20
Everything is Ok
Size:       49
Compressed: 79
*
file data9.bin
data9.bin: ASCII text
```

Y listo, ya tenemos nuestro archivo, tan solo hay leerlo y tendremos la contraseña para el siguiente nivel:
```bash
cat data9.bin
...
```

<br>

<h3 id="descompresion2">Segunda Forma de Descompresión con gzip, bzip2 y tar</h3>

Ahora para la segunda forma es similar el proceso, pero vamos a usar los **comandos gzip, bzip2 y tar**.

Como dice las instrucciones, vamos a movernos al directorio **/tmp** y vamos a crear un directorio temporal con el **comando mktemp**:
```bash
bandit12@bandit:~$ cd /tmp
bandit12@bandit:/tmp$ mktemp -d
/tmp/tmp.nq7qMbq79D
bandit12@bandit:/tmp$ cd /tmp/tmp.nq7qMbq79D$
bandit12@bandit:/tmp/tmp.nq7qMbq79D$
```

Copiamos el archivo en nuestro directorio actual:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ cp ~/data.txt .
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt
```

Y convertimos el archivo que es hexadecimal a binario con el **comando xxd**:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ cat data.txt | xxd -r > hexdata
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt  hexdata
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file hexdata 
hexdata: gzip compressed data, was "data2.bin"...
```

Con esto listo, vamos a empezar a descomprimir todos los archivos que vayan resultando, la diferencia a como lo hicimos antes es que debemos ir agregando las extensiones a cada archivo resultante y cada archivo se debe descomprimir con su debida herramienta.

Vamos a por ello:

* Primer descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv hexdata hexdata.gz
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ gzip -d hexdata.gz
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt  hexdata
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file hexdata 
hexdata: bzip2 compressed data, block size = 900k
```

* Segunda descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv hexdata hexdata.bz2
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt  hexdata.bz2
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ bzip2 -d hexdata.bz2 
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt  hexdata
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file hexdata 
hexdata: gzip compressed data, was "data4.bin"
```

* Tercera descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv hexdata hexdata.gz
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt  hexdata.gz
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ gzip -d hexdata.gz 
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt  hexdata
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file hexdata 
hexdata: POSIX tar archive (GNU)
```

* Cuarta descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv hexdata hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data.txt  hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ tar xf hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data5.bin  data.txt  hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file data5.bin
data5.bin: POSIX tar archive (GNU)
```

* Quinta descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv data5.bin data.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ tar xf data.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data6.bin  data.tar  data.txt  hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
```

* Sexta descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv data6.bin data6.bz2
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data6.bz2  data.tar  data.txt  hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ bzip2 -d data6.bz2
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data6  data.tar  data.txt  hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file data6
data6: POSIX tar archive (GNU)
```

* Septima descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv data6 data6.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ tar xf data6.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data6.tar  data8.bin  data.tar  data.txt  hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file data8.bin 
data8.bin: gzip compressed data, was "data9.bin"...
```

* Octavo descompresión:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ mv data8.bin data8.gz
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ gzip -d data8.gz 
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ ls
data8  data6.tar  data.tar  data.txt  hexdata.tar
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ file data8
data8: ASCII text
```

Y listo, tenemos otra vez la contraseña:
```bash
bandit12@bandit:/tmp/tmp.nq7qMbq79D$ cat data8
The password is ...
```


<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel13" style="text-align:center;">Nivel 13 a 14</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en **/etc/bandit_pass/bandit14** y sólo puede ser leída por el usuario **bandit14**. Para este nivel, no obtienes la siguiente contraseña, pero obtienes una clave SSH privada que puede ser usada para iniciar sesión en el siguiente nivel. Nota: localhost es un nombre de host que se refiere a la máquina en la que estás trabajando.

----------------
<br>

<h2 id="Exp13">Descripción de Comandos</h2>

<br>

<h3 id="llaves">Llave Privada y Pública de Servicio SSH</h3>

| **Llaves del Servicio SSH** |
|:-----------:|
| *Las llaves públicas y privadas en SSH son un mecanismo utilizado para la autenticación segura entre un cliente y un servidor. En lugar de usar contraseñas, se emplea criptografía asimétrica con un par de llaves: una llave privada (que se mantiene secreta) y una llave pública (que se comparte).* | 

<br>

| **Llaves Privada** |
|:-----------:|
| *Es un archivo que solo el usuario debe tener. Se guarda en el cliente y nunca se comparte. Es usada para desencriptar los mensajes cifrados con la llave pública y para firmar los mensajes enviados.* |

<br>

| **Llaves Pública** |
|:-----------:|
| *Es un archivo que se puede compartir con cualquier persona o servidor. Se coloca en el servidor al que deseas conectarte. El servidor usa esta llave para cifrar los datos y verificar la autenticidad de quien se conecta.* |

<br>

Pasos para la creación, distribución y autenticación de llaves:
* **Generación de llaves**: El usuario genera un par de llaves (pública y privada).
* **Distribución de la llave pública**: El usuario coloca la llave pública en el servidor al que quiere conectarse. La llave pública se agrega al archivo **~/.ssh/authorized_keys** en el servidor.
* **Autenticación:** 
	* Cuando el cliente intenta conectarse al servidor, el servidor envía un desafío cifrado con la llave pública del usuario.
	* El cliente usa su llave privada para desencriptar el desafío.
	* Si el cliente puede desencriptar correctamente, la autenticación se completa y se establece la conexión.

Formas de uso:

* Generando llaves:
```bash
ssh-keygen -t rsa -b 4096 -C "tu_email@example.com"
```
Esto va a generar la llave privada (**~/.ssh/id_rsa**) y la llave pública (**~/.ssh/id_rsa.pub**).

* Copiando la llave pública al servidor, esto para permitir la autenticación sin contraseña:
```bash
ssh-copy-id usuario@servidor
```

* Conectandose al servidor usando la llave privada:
```bash
ssh -i id_rsa usuario@servidor
```

Acá dejo más información sobre **llaves SSH**:
* <a href="http://codigoelectronica.com/blog/ssh-key-guia-completa-de-las-llaves-publicas-y-privadas-de-ssh" target="_blank">SSH key: guía completa de las llaves públicas y privadas de SSH</a>

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol13">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit13@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit13@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 12**.

Veamos la llave privada que menciona las instrucciones:
```bash
bandit13@bandit:~$ ls -la
total 24
drwxr-xr-x  2 root     root     4096 Sep 19 07:08 .
drwxr-xr-x 70 root     root     4096 Sep 19 07:09 ..
-rw-r--r--  1 root     root      220 Mar 31 08:41 .bash_logout
-rw-r--r--  1 root     root     3771 Mar 31 08:41 .bashrc
-rw-r--r--  1 root     root      807 Mar 31 08:41 .profile
-rw-r-----  1 bandit14 bandit13 1679 Sep 19 07:08 sshkey.private
```

Analicemos un poco la llave privada:
```bash
bandit13@bandit:~$ file sshkey.private 
sshkey.private: PEM RSA private key
bandit13@bandit:~$ cat sshkey.private 
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAxkkOE83W2cOT7IWhFc9aPaaQmQDdgzuXCv+ppZHa++buSkN+
gg0tcr7Fw8NLGa5+Uzec2rEg0WmeevB13AIoYp0MZyETq46t+jk9puNwZwIt9XgB
...
...
...
```

La idea es que nosotros nos conectemos al siguiente nivel usando la llave privada, no es nada más que eso. Debemos especificar el usuario **bandit14** y usamos el servidor local.

Ahora conectemonos al siguiente nivel, usando la llave privada:
```bash
bandit13@bandit:~$ ssh -i sshkey.private bandit14@localhost -p 2220
The authenticity of host '[localhost]:2220 ([127.0.0.1]:2220)' can't be established.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Could not create directory '/home/bandit13/.ssh' (Permission denied).
Failed to add the host to the list of known hosts (/home/bandit13/.ssh/known_hosts).
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

!!! You are trying to log into this SSH server with a password on port 2220 from localhost.
!!! Connecting from localhost is blocked to conserve resources.
!!! Please log out and log in again.
...
...
...
bandit14@bandit:~$ whoami
bandit14
```

Excelente, ahora simplemente veamos la contraseña en la ruta que menciona las instrucciones:
```bash
bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
...
```
Hemos terminado este nivel.


<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel14" style="text-align:center;">Nivel 14 a 15</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se puede recuperar enviando la contraseña del nivel actual al **puerto 30000 en localhost**.

----------------
<br>

<h2 id="Exp14">Descripción de Comandos</h2>

<br>

<h3 id="nc">Comando nc</h3>

| **Comando nc** |
|:-----------:|
| *El comando nc, también conocido como Netcat, es una herramienta de red versátil en Linux que permite crear conexiones de red de manera sencilla, tanto TCP como UDP. Se utiliza para enviar y recibir datos a través de redes, y es útil para diagnósticos de red, transferencias de archivos, escaneo de puertos, y creación de servidores simples.* |

Formas de uso:
```bash
nc [opciones] [host] [puerto]
```

Ejemplo:
* Conectandonos a un servidor en un puerto específico:
```bash
nc ejemplo.com 80
```

* Creando servidor para escuchar en un puerto específico:
```bash
nc -l 1234
```

* Creando servidor para escuchar de manera persistente en un puerto específico, sin resolver nombres de host y mostrando detalles verbosos sobre cualquier conexión entrante:
```bash
nc -lnvp 443
```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol14">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit14@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit14@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 13**.

Esto es bastante simple, una vez que entremos podemos directamente obtener la contraseña para el siguiente nivel con el **comando nc**.

Vamos a probarlo, recuerda que debes escribir la contraseña del nivel actual para que te de la del siguiente nivel:
```bash
bandit14@bandit:~$ nc localhost 30000
contraseña_nivel_actual
Correct!
...
```

Observa lo que pasa si te equivocas:
```bash
bandit14@bandit:~$ nc localhost 30000
ls
Wrong! Please enter the correct current password.
```

Listo, ya terminamos con este nivel.


<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel?" style="text-align:center;">Nivel 15 a 16</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se puede recuperar enviando la contraseña del nivel actual al **puerto 30001 en localhost** utilizando **encriptación SSL/TLS**.

**Nota útil**: ¿Recibes **"DONE"**, **"RENEGOTIATING"** o **"KEYUPDATE"**? Lea la sección **"CONNECTED COMMANDS"** en la página de manual.

----------------
<br>

<h2 id="Exp15">Descripción de Comandos</h2>

<br>

<h3 id="ncat">Comando ncat</h3>

| **Comando ncat** |
|:-----------:|
| *ncat es una herramienta de red mejorada basada en Netcat que forma parte del proyecto Nmap. Ncat ofrece más características avanzadas para realizar conexiones de red, incluyendo soporte para SSL/TLS, redireccionamiento de puertos, autenticación y múltiples clientes. Es útil para transferencias de archivos, pruebas de conectividad, diagnóstico de redes, y más.* |

Formas de uso:
```bash
ncat [opciones] [host] [puerto]
```

Ejemplo:
* Conectandonos a un servidor en un puerto específico:
```bash
ncat ejemplo.com 80
```

* Realizando una conexión segura con **SSL**:
```bash
ncat --ssl ejemplo.com 443
```

* Aplicando redirección de puertos:
```bash
ncat -l 8080 --sh-exec "ncat ejemplo.com 80"
```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol15">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit15@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit15@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 14**.

Es muy simple lo que tenemos que hacer, es lo mismo que hicimos en el nivel anterior, pero usando el **comando ncat** y usando el **parámetro --ssl** para crear una conexión segura, nos pedira la contraseña de la sesión actual y cuando se la demos, nos dara la contraseña del siguiente nivel.

Vamos a probar:

* Vamos a usar el comando ncat para conectarnos al localhost por el puerto 30001 y creando una conexión segura:
```bash
bandit15@bandit:~$ ncat --ssl 127.0.0.1 30001
contraseña_nivel_actual
Correct!
...
```

Y listo, no hay más que hacer. Vayamos al siguiente nivel.


<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel16" style="text-align:center;">Nivel 16 a 17</h1>
</div>
<br>


**INSTRUCCIONES**:

Las credenciales para el siguiente nivel se pueden recuperar enviando la contraseña del nivel actual a un puerto en localhost en el rango 31000 a 32000. Primero averigua cuáles de estos puertos tienen un servidor escuchando en ellos. Luego averigua cuáles de ellos hablan SSL/TLS y cuáles no. Sólo hay 1 servidor que te dará las siguientes credenciales, los otros simplemente te devolverán lo que les envíes.

Nota útil: ¿Recibes "DONE", "RENEGOTIATING" o "KEYUPDATE"? Lee la sección "COMANDOS CONECTADOS" en la página de manual.

----------------
<br>

<h2 id="Exp16">Descripción de Comandos</h2>

<br>

<h3 id="chmod">Comando chmod</h3>

| **Comando chmod** |
|:-----------:|
| *El comando chmod se utiliza para cambiar los permisos de acceso a archivos y directorios en sistemas Unix y Linux. Los permisos controlan quién puede leer, escribir o ejecutar un archivo.* |

Formas de uso:
```bash
chmod [opciones] mode archivo
```

Permisos en formato numérico:
* 4 = Leer (r)
* 2 = Escribir (w)
* 1 = Ejecutar (x)

Los permisos se suman para cada conjunto de usuarios:

* El primer número es para el dueño.
* El segundo número es para el grupo.
* El tercer número es para otros.

Ejemplo:
* Dando al propietario todos los permisos (rwx), al grupo permisos de lectura y ejecución (rx), y a otros solo de lectura (r) de un archivo:
```bash
chmod 755 archivo.txt
```

* Añadiendo permisos de escritura para el propietario:
```bash
chmod u+w archivo.txt
```

* Cambiando permisos de manera recursiva en un directorio:
```bash
chmod -R 644 directorio/
```

<br>

<h3 id="mktemp">Comando mktemp</h3>

| **Comando mktemp** |
|:-----------:|
| *El comando mktemp se utiliza para crear archivos o directorios temporales de manera segura. Genera nombres únicos para evitar colisiones, lo cual es útil en scripts donde se requieren archivos temporales.* |

Formas de uso: 
```bash
mktemp [opciones] [template]
```

**template** es opcional y debe contener una serie de "X" que serán reemplazadas por un identificador único.

Opciones comunes:
* **-d**: Crea un directorio temporal en lugar de un archivo.
* **-q**: Silencia mensajes de error.

Ejemplo:
* Creando un archivo temporal y mostrar su nombre:
```bash
mktemp
```

* Creando un directorio temporal:
```bash
mktemp -d
```

<br>

<h3 id="nmap">Comando nmap</h3>

| **Comando nmap** |
|:-----------:|
| *Nmap es una herramienta de código abierto usada para el escaneo y descubrimiento de redes. Se utiliza principalmente para realizar auditorías de seguridad y pruebas de penetración, pero también sirve para administrar redes y obtener información sobre hosts y servicios disponibles.* |

Formas de uso: 
```bash
nmap [opciones] [objetivo]
```

Ejemplo:
* Escaneo básico de un host:
```bash
nmap 192.168.1.1
```

* Escaneo de todos los puertos:
```bash
nmap -p- 192.168.1.1
```

* Escaneo de puertos y detección de versiones:
```bash
nmap -sV -p 80,443 192.168.1.1
```

RECOMENDABLE: echale un vistaso y estudia bien el uso de nmap, porque es una de las mejores herramientas y necesarias que todo pentester debe conocer.

* <a href="https://nmap.org/" target="_blank">Herramienta NMAP</a>

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol16">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit16@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit16@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 15**.

El proceso es similar al anterior nivel, pero primero debemos identificar que puertos estan actives entre el **puerto 31000 y 32000**, entre estos debe haber un puerto que utilice **SSL/TLS** y ahí aplicaremos el mismo proceso que el nivel anterior.

Entonces, vamos a ocupar la herramienta **nmap** para identificar los puertos abiertos y cual utiliza **SSL/TLS**.

Empecemos creando un directorio temporal y aplicando un **comando de nmap** que nos de los puertos abiertos:
```bash
bandit16@bandit:~$ cd /tmp
bandit16@bandit:/tmp$ mktemp -d
/tmp/tmp.jh1Azu9X8B
bandit16@bandit:/tmp$ cd /tmp/tmp.jh1Azu9X8B
bandit16@bandit:/tmp/tmp.jh1Azu9X8B$
bandit16@bandit:/tmp/tmp.jh1Azu9X8B$ nmap --open -p 31000-32000 -T5 -vvv -n -Pn 127.0.0.1
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Initiating Connect Scan at 01:17
Scanning 127.0.0.1 [1001 ports]
Discovered open port 31790/tcp on 127.0.0.1
Discovered open port 31046/tcp on 127.0.0.1
Discovered open port 31518/tcp on 127.0.0.1
Discovered open port 31960/tcp on 127.0.0.1
Discovered open port 31691/tcp on 127.0.0.1
Completed Connect Scan at 01:17, 0.04s elapsed (1001 total ports)
Nmap scan report for 127.0.0.1
Host is up, received user-set (0.00016s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT      STATE SERVICE REASON
31046/tcp open  unknown syn-ack
31518/tcp open  unknown syn-ack
31691/tcp open  unknown syn-ack
31790/tcp open  unknown syn-ack
31960/tcp open  unknown syn-ack
```

Ya identificamos los puertos, ahora veamos cual de ellos usa **SSL/TLS**, para esto utilizaremos el **script ssl-enum-ciphers de nmap**:
```bash
bandit16@bandit:/tmp/tmp.jh1Azu9X8B$ nmap --script ssl-enum-ciphers -p 31046,31518,31691,31790,31960 127.0.0.1
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00026s latency).

PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
| ssl-enum-ciphers: 
|   TLSv1.2: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
...
...
31691/tcp open  unknown
31790/tcp open  unknown
| ssl-enum-ciphers: 
|   TLSv1.2: 
|     ciphers: 
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
...
...
```

Hay dos puertos que lo utilizan, probemos a conectarnos a ambos y pasemos la contraseña del nivel actual para ver que nos devuelve:
```bash
bandit16@bandit:/tmp/tmp.jh1Azu9X8B$ ncat --ssl 127.0.0.1 31518
contraseña_nivel_actual
^C
bandit16@bandit:/tmp/tmp.jh1Azu9X8B$ ncat --ssl 127.0.0.1 31790
contraseña_nivel_actual
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
...
...
```

Resulto ser el **puerto 31790**, si le damos la contraseña nos devuelve una llave privada, como ya hemos hecho antes vamos a copiar la llave, le daremos los permisos necesarios y la usaremos para conectarnos al siguiente nivel.

Vamos por pasos:
* Copiemos la llave en un archivo llamado **id_rsa**:
```bash
nano id_rsa
```

* Asignemosle los permisos correctos para que funcione la llave:
```bash
chmod 600 id_rsa
```

* Y probemos a conectarnos al siguiente nivel:
```bash
ssh -i id_rsa bandit17@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames
...
...
...
Enjoy your stay!
*
bandit17@bandit:~$ ls -la
total 36
drwxr-xr-x  3 root     root     4096 Sep 19 07:08 .
drwxr-xr-x 70 root     root     4096 Sep 19 07:09 ..
-rw-r-----  1 bandit17 bandit17   33 Sep 19 07:08 .bandit16.password
-rw-r--r--  1 root     root      220 Mar 31 08:41 .bash_logout
-rw-r--r--  1 root     root     3771 Mar 31 08:41 .bashrc
-rw-r-----  1 bandit18 bandit17 3300 Sep 19 07:08 passwords.new
-rw-r-----  1 bandit18 bandit17 3300 Sep 19 07:08 passwords.old
-rw-r--r--  1 root     root      807 Mar 31 08:41 .profile
drwxr-xr-x  2 root     root     4096 Sep 19 07:08 .ssh
```

Si lo deseas, la contraseña de este nivel se encuentra en: 
```bash
bandit17@bandit:~$ ls -l /etc/bandit_pass/bandit17
-r-------- 1 bandit17 bandit17 33 Sep 19 07:07 /etc/bandit_pass/bandit17
bandit17@bandit:~$ cat /etc/bandit_pass/bandit17
...
```
Continuemos al siguiente nivel.


<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel17" style="text-align:center;">Nivel 17 a 18</h1>
</div>
<br>


**INSTRUCCIONES**:

Hay 2 archivos en el homedirectory: **passwords.old y passwords.new**. La contraseña para el siguiente nivel está en **passwords.new** y es la única línea que ha cambiado entre **passwords.old y passwords.new**.

**NOTA**: si has resuelto este nivel y ves "Byebye!" cuando intentas entrar en **bandit18**, esto está relacionado con el siguiente nivel, **bandit19**.

----------------
<br>

<h2 id="Exp17">Descripción de Comandos</h2>

<br>

<h3 id="diff">Comando diff</h3>

| **Comando diff** |
|:-----------:|
| *El comando diff se utiliza para comparar el contenido de dos archivos o directorios en sistemas Unix y Linux. Muestra las diferencias línea por línea, ayudando a identificar cambios o modificaciones entre los archivos. diff toma dos archivos como entrada y compara su contenido. Si encuentra diferencias, muestra las líneas que difieren entre los archivos.* |

Formas de uso:
```bash
diff [opciones] archivo1 archivo2
```

Ejemplo:
* Comparando dos archivos simples:
```bash
diff archivo1.txt archivo2.txt
```

* Usando el formato unificado para una mejor legibilidad:
```bash
diff -u archivo1.txt archivo2.txt
```

* Comparando dos directorios:
```bash
diff -r directorio1 directorio2
```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol17">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit17@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames
*
bandit17@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 16**.

Es muy simple la solución, unicamente usaremos el **comando diff** para encontrar la diferencia entre ambos archivos, dicha diferencia será la contraseña para el siguiente nivel.

Vamos a probarlo:
```bash
bandit17@bandit:~$ ls -la
total 36
drwxr-xr-x  3 root     root     4096 Sep 19 07:08 .
drwxr-xr-x 70 root     root     4096 Sep 19 07:09 ..
-rw-r-----  1 bandit17 bandit17   33 Sep 19 07:08 .bandit16.password
-rw-r--r--  1 root     root      220 Mar 31 08:41 .bash_logout
-rw-r--r--  1 root     root     3771 Mar 31 08:41 .bashrc
-rw-r-----  1 bandit18 bandit17 3300 Sep 19 07:08 passwords.new
-rw-r-----  1 bandit18 bandit17 3300 Sep 19 07:08 passwords.old
-rw-r--r--  1 root     root      807 Mar 31 08:41 .profile
drwxr-xr-x  2 root     root     4096 Sep 19 07:08 .ssh
bandit17@bandit:~$ diff passwords.old passwords.new
42c42
< linea_diferencia
---
> contraseña_bandit18
```
Y listo, ya tenemos la contraseña, pero recuerda las instrucciones pues si la probamos en el siguiente nivel, automaticamente nos sacara.


<br>
<br>
<hr> 
<div style="position: relative;">
        <h1 id="Nivel18" style="text-align:center;">Nivel 18 a 19</h1>
</div>
<br>


**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en un archivo **readme en el homedirectory**. Desafortunadamente, alguien ha modificado .bashrc para cerrar la sesión cuando te conectas con **SSH**.

----------------

<br>

<h2 id="Exp18">Descripción de Comandos</h2>

<br>

<h3 id="sshpass">Comando sshpass</h3>

| **Comando sshpass** |
|:-----------:|
| *sshpass es una utilidad que permite proporcionar una contraseña de manera no interactiva para comandos que utilizan SSH (como ssh, scp, o rsync). En lugar de introducir la contraseña manualmente cuando te conectas a un servidor remoto, sshpass la suministra automáticamente. Es útil en scripts de automatización donde no es posible o conveniente usar autenticación con claves SSH.* |

Formas de uso:
```bash
sshpass -p 'contraseña' comando_ssh
```

Ejemplo:
* Conectandonos a un **servidor SSH** con una contraseña:
```bash
sshpass -p 'mi_contraseña' ssh usuario@servidor.com
```

* Copiando archivos con **comando scp** usando una contraseña:
```bash
sshpass -p 'mi_contraseña' scp archivo.txt usuario@servidor.com:/ruta/destino/
```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol18">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit18@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames
*
bandit18@bandit.labs.overthewire.org's password:
...
...
...
Welcome to OverTheWire!
Byebye !
Connection to bandit.labs.overthewire.org closed.
```
La contraseña es la que encontraste en el **nivel 17**.

Como bien mencionan las instrucciones, una vez que intentamos entrar nos saca. Para este caso, vamos a utilizar el **comando sshpass** para poder leer la contraseña.

Probemos:

* Intentemos ejecutar el **comando whoami** con el **comando sshpass**:
```bash
sshpass -p 'contraseña_nivel_actual' ssh bandit18@bandit.labs.overthewire.org -p2220 whoami
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames
*
bandit18
```

* Ahora veamos si es verdad que se encuentra un archivo llamado **readme** en el **directorio /home**
```bash
sshpass -p 'contraseña_nivel_actual' ssh bandit18@bandit.labs.overthewire.org -p2220 ls
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames
*
readme
```

* Ahí esta, tan solo hay que leerlo:
```bash
sshpass -p 'contraseña_nivel_actual' ssh bandit18@bandit.labs.overthewire.org -p2220 cat readme
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames
*
contraseña_siguiente_nivel
```

* También podemos obtener una **sesión interactiva de Bash**:
```bash
sshpass -p 'x2gLTTjFwMOhQ8oWNbMN362QKxfRqGlO' ssh bandit18@bandit.labs.overthewire.org -p2220 bash
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames
*
whoami
bandit18
ls
readme
cat readme
contraseña_siguiente_nivel
^C#
```

Y listo, ya podemos ir al siguiente nivel.


<br>
<br>
<hr> 
<div style="position: relative;">
        <h1 id="Nivel19" style="text-align:center;">Nivel 19 a 20</h1>
</div>
<br>


**INSTRUCCIONES**:

Para acceder al siguiente nivel, debe utilizar el **binario setuid** en el **directorio home**. Ejecútalo sin argumentos para saber cómo usarlo. La contraseña para este nivel se puede encontrar en el lugar habitual **(/etc/bandit_pass)**, después de haber utilizado el **binario setuid**.

----------------
<br>

<h2 id="Exp19">Descripción de Comandos</h2>

<br>

<h3 id="suid">Permiso SUID</h3>

| **Permiso SUID** |
|:-----------:|
| *El bit SUID (Set User ID) es un permiso especial en sistemas Unix y Linux que, cuando se aplica a un archivo ejecutable, permite que el archivo se ejecute con los privilegios del propietario del archivo, en lugar de los privilegios del usuario que lo ejecuta. Esto es especialmente útil para programas que necesitan privilegios elevados temporalmente. Por ejemplo, el comando passwd utiliza SUID para permitir que los usuarios cambien sus contraseñas, ya que modifica archivos del sistema que solo el superusuario puede cambiar, como /etc/shadow.* |

Formas de uso:
```bash
./script_con_permisos_SUID
```

Ejemplo:
* Ejecutando script con **permisos SUID**:
```bash
./bandit20-do id
```

* Hacer un archivo ejecutable con **SUID**:
```bash
chmod u+s mi_script
```

* Verificar si un archivo tiene **SUID activado**:
```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 150K Aug  5 14:20 /usr/bin/passwd
```
El **permiso rws** en lugar de **rwx** indica que **el bit SUID está activado**.

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol19">Solución</h2>

Primero entremos al **servicio SSH**:
```bash
ssh bandit19@bandit.labs.overthewire.org -p2220
                         _                     _ _ _   
                        | |__   __ _ _ __   __| (_) |_ 
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_ 
                        |_.__/ \__,_|_| |_|\__,_|_|\__|
                                                       

                      This is an OverTheWire game server. 
            More information on http://www.overthewire.org/wargames

bandit19@bandit.labs.overthewire.org's password:
```
La contraseña es la que encontraste en el **nivel 18**.

De acuerdo a las instrucciones, lo unico que debemos hacer es usar el **script bandit20-do** para ejecutar comandos como **bandit20**, vamos a probar si es cierto.

Probemos:
* Ejecutemos el script para ver como se usa:
```bash
bandit19@bandit:~$ ls -la
total 36
drwxr-xr-x  2 root     root      4096 Sep 19 07:08 .
drwxr-xr-x 70 root     root      4096 Sep 19 07:09 ..
-rwsr-x---  1 bandit20 bandit19 14880 Sep 19 07:08 bandit20-do
-rw-r--r--  1 root     root       220 Mar 31 08:41 .bash_logout
-rw-r--r--  1 root     root      3771 Mar 31 08:41 .bashrc
-rw-r--r--  1 root     root       807 Mar 31 08:41 .profile
bandit19@bandit:~$ ./bandit20-do 
Run a command as another user.
  Example: ./bandit20-do id
```

* Ejecutemos el mismo comando de ejemplo que nos pone el script:
```bash
bandit19@bandit:~$ ./bandit20-do id
uid=11019(bandit19) gid=11019(bandit19) euid=11020(bandit20) groups=11019(bandit19)
```

* Veamos con el **comando whoami**:
```bash
bandit19@bandit:~$ ./bandit20-do whoami
bandit20
```

* Excelente, vamos a obtener la contraseña en la ruta que nos indicaron las instrucciones:
```bash
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
...
```

* Por último, podemos obtener una **sesión interactiva de bash**:
```bash
bandit19@bandit:~$ ./bandit20-do bash -p
bash-5.2$ whoami
bandit20
bash-5.2$ ls -la
total 36
drwxr-xr-x  2 root     root      4096 Sep 19 07:08 .
drwxr-xr-x 70 root     root      4096 Sep 19 07:09 ..
-rwsr-x---  1 bandit20 bandit19 14880 Sep 19 07:08 bandit20-do
-rw-r--r--  1 root     root       220 Mar 31 08:41 .bash_logout
-rw-r--r--  1 root     root      3771 Mar 31 08:41 .bashrc
-rw-r--r--  1 root     root       807 Mar 31 08:41 .profile
bash-5.2$ cat /etc/bandit_pass/bandit20
contraseñ_siguiente_nivel
bash-5.2$ exit
exit
bandit19@bandit:~$ exit
logout
Connection to bandit.labs.overthewire.org closed.
```
Continuemos al siguiente nivel.


<br>
<br>
<hr> 
<div style="position: relative;">
        <h1 id="Nivel20" style="text-align:center;">Nivel 20 a 21</h1>
</div>
<br>


**INSTRUCCIONES**:

Hay un binario setuid en el homedirectory que hace lo siguiente: establece una conexión con localhost en el puerto que especifiques como argumento en la línea de comandos. Luego lee una línea de texto de la conexión y la compara con la contraseña del nivel anterior (bandit20). Si la contraseña es correcta, transmitirá la contraseña para el siguiente nivel (bandit21).

NOTA: Prueba a conectarte a tu propio demonio de red para ver si funciona como crees

----------------
<br>

<h2 id="Exp?">Descripción de Comandos</h2>

<br>

<h3 id="?">Comando ?</h3>

| **Comando ** |
|:-----------:|
| ** |

Formas de uso:
```bash

```

Ejemplo:
* 
```bash

```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol20">Solución</h2>

Primero entremos al **servicio SSH**:
```bash

```
La contraseña es la que encontraste en el **nivel 19**.



<br>
<br>
<hr> 
<div style="position: relative;">
        <h1 id="Nivel?" style="text-align:center;">Nivel ? a ?</h1>
</div>
<br>


**INSTRUCCIONES**:

texto 
----------------
<br>

<h2 id="Exp?">Descripción de Comandos</h2>

<br>

<h3 id="?">Comando ?</h3>

| **Comando ** |
|:-----------:|
| ** |

Formas de uso:
```bash

```

Ejemplo:
* 
```bash

```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol?">Solución</h2>

Primero entremos al **servicio SSH**:
```bash

```
La contraseña es la que encontraste en el **nivel ?**.



<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel?" style="text-align:center;">Nivel ? a ?</h1>
</div>
<br>


**INSTRUCCIONES**:

texto
----------------
<br>

<h2 id="Exp?">Descripción de Comandos</h2>

<br>

<h3 id="?">Comando ?</h3>

| **Comando ** |
|:-----------:|
| ** |

Formas de uso:
```bash

```

Ejemplo:
* 
```bash

```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol?">Solución</h2>

Primero entremos al **servicio SSH**:
```bash

```
La contraseña es la que encontraste en el **nivel ?**.




<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel?" style="text-align:center;">Nivel ? a ?</h1>
</div>
<br>


**INSTRUCCIONES**:

texto
----------------
<br>

<h2 id="Exp?">Descripción de Comandos</h2>

<br>

<h3 id="?">Comando ?</h3>

| **Comando ** |
|:-----------:|
| ** |

Formas de uso:
```bash

```

Ejemplo:
* 
```bash

```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol?">Solución</h2>

Primero entremos al **servicio SSH**:
```bash

```
La contraseña es la que encontraste en el **nivel ?**.





<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel?" style="text-align:center;">Nivel ? a ?</h1>
</div>
<br>


**INSTRUCCIONES**:

texto
----------------
<br>

<h2 id="Exp?">Descripción de Comandos</h2>

<br>

<h3 id="?">Comando ?</h3>

| **Comando ** |
|:-----------:|
| ** |

Formas de uso:
```bash

```

Ejemplo:
* 
```bash

```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol?">Solución</h2>

Primero entremos al **servicio SSH**:
```bash

```
La contraseña es la que encontraste en el **nivel ?**.


<br>
<br>
<hr>
<div style="position: relative;">
        <h1 id="Nivel?" style="text-align:center;">Nivel ? a ?</h1>
</div>
<br>


**INSTRUCCIONES**:

texto
----------------
<br>

<h2 id="Exp?">Descripción de Comandos</h2>

<br>

<h3 id="?">Comando ?</h3>

| **Comando ** |
|:-----------:|
| ** |

Formas de uso:
```bash

```

Ejemplo:
* 
```bash

```

<br>

**Continuemos con la solución.**

<br>

<h2 id="Sol?">Solución</h2>

Primero entremos al **servicio SSH**:
```bash

```
La contraseña es la que encontraste en el **nivel ?**.




<br>
<br>
<div style="position: relative;">
	<h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


* https://overthewire.org/wargames/bandit/
* https://www.webservertalk.com/dashed-filename
* https://itsfoss.com/es/comando-find-linux/#buscar-varios-archivos-con-varias-extensiones-o-condici%C3%B3n
* https://www.hostinger.mx/tutoriales/como-usar-comando-find-locate-en-linux/#Busqueda_por_tipo
* https://gospelidea.com/blog/que-son-los-permisos-chmod
* https://www.ionos.mx/digitalguide/servidores/configuracion/comando-linux-find/
* https://geekland.eu/uso-del-comando-grep-en-linux-y-unix-con-ejemplos/
* https://geekland.eu/uso-del-comando-grep-en-linux-y-unix-con-ejemplos/

<br>

# FIN

<footer id="myFooter">
    <!-- Footer para eliminar el botón -->
</footer>

<style>
        #backToIndex {
                display: none;
                position: fixed;
                left: 87%;
                top: 90%;
                z-index: 2000;
                background-color: #81fbf9;
                border-radius: 10px;
                border: none;
                padding: 4px 6px;
                cursor: pointer;
        }
</style>

<a id="backToIndex" href="#Indice">
        <img src="/assets/images/arrow-up.png" style="width: 45px; height: 45px;">
</a>

<script>
    window.onscroll = function() { showButton() };

    function showButton() {
        const scrollPosition = document.documentElement.scrollTop || document.body.scrollTop;
        const indicePosition = document.getElementById("Indice").offsetTop;
        const footerPosition = document.getElementById("myFooter").offsetTop;
        const windowHeight = window.innerHeight;

        const button = document.getElementById("backToIndex");

        // Mostrar el botón si el usuario ha bajado al índice
        if (scrollPosition >= indicePosition && (scrollPosition + windowHeight) < footerPosition) {
            button.style.display = "block";
            button.style.position = "fixed";
            button.style.top = "90%";
        } else {
            button.style.display = "none";
        }
    }
</script>
