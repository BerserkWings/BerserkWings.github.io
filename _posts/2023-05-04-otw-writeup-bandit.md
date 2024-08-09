---
layout: single
title: Bandit - Over The Wire
excerpt: "Lo que haremos en esta ocasión, será resolver todos los niveles que se encuentran en la sección **Bandit**, esto como práctica de pentesting en entornos Linux."
date: 2023-05-04
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
				<li><a href="#Sol0">Solución</a></li>
			</ul>
		<li><a href="#Nivel1">Nivel 1 a 2</a></li>
			<ul>
                                <li><a href="#Exp1">Descripción de Comandos</a></li>
                                <li><a href="#Sol1">Solución</a></li>
                        </ul>			
                <li><a href="#Nivel2">Nivel 2 a 3</a></li>
			<ul>
                                <li><a href="#Sol2">Solución</a></li>
                        </ul>
                <li><a href="#Nivel3">Nivel 3 a 4</a></li>
			<ul>
                                <li><a href="#Exp3">Descripción de Comandos</a></li>
                                <li><a href="#Sol3">Solución</a></li>
                        </ul>
                <li><a href="#Nivel4">Nivel 4 a 5</a></li>
			<ul>
                                <li><a href="#Exp4">Descripción de Comandos</a></li>
                                <li><a href="#Sol4">Solución</a></li>
                        </ul>
                <li><a href="#Nivel5">Nivel 5 a 6</a></li>
			<ul>
                                <li><a href="#Sol5">Solución</a></li>
                        </ul>
                <li><a href="#Nivel6">Nivel 6 a 7</a></li>
			<ul>
                                <li><a href="#Sol6">Solución</a></li>
                        </ul>
                <li><a href="#Nivel7">Nivel 7 a 8</a></li>
			<ul> 
                                <li><a href="#Sol7">Solución</a></li>
                        </ul>
                <li><a href="#Nivel8">Nivel 8 a 9</a></li>
			<ul> 
                                <li><a href="#Exp8">Descripción de Comandos</a></li>
                                <li><a href="#Sol8">Solución</a></li>
                        </ul>
                <li><a href="#Nivel9">Nivel 9 a 10</a></li>
			<ul> 
                                <li><a href="#Exp9">Descripción de Comandos</a></li>
                                <li><a href="#Sol9">Solución</a></li>
                        </ul>
                <li><a href="#Nivel10">Nivel 10 a 11</a></li>
			<ul> 
                                <li><a href="#Exp10">Descripción de Comandos</a></li>
                                <li><a href="#Sol10">Solución</a></li>
                        </ul>
                <li><a href="#Nivel11">Nivel 11 a 12</a></li>
			<ul> 
                                <li><a href="#Exp11">Descripción de Comandos</a></li>
                                <li><a href="#Sol11">Solución</a></li>
                        </ul>
		<li><a href="#Nivel12">Nivel 12 a 13</a></li>
		<li><a href="#Nivel13">Nivel 13 a 14</a></li>
			<ul> 
                                <li><a href="#Exp13">Descripción de Comandos</a></li>
                                <li><a href="#Sol13">Solución</a></li>
                        </ul>
		<li><a href="#Nivel14">Nivel 14 a 15</a></li>
		<li><a href="#Nivel15">Nivel 15 a 16</a></li>
		<li><a href="#Nivel16">Nivel 16 a 17</a></li>
		<li><a href="#Nivel17">Nivel 17 a 18</a></li>
		<li><a href="#Nivel18">Nivel 18 a 19</a></li>
		<li><a href="#Nivel19">Nivel 19 a 20</a></li>
		<li><a href="#Links">Links de Investigación</a></li>
	</ul>
</div>


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Nivel0" style="text-align:center;">Nivel 0 a 1</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>

----------------
**INSTRUCCIONES**:

El objetivo de este nivel es que inicies sesión en el juego conectandote al **servicio SSH** que tienen activo. El host al que debes conectarte es **bandit.labs.overthewire.org**, en el **puerto 2220**. El nombre de usuario es **bandit0** y la contraseña es **bandit0** (esto mismo te lo menciona la descripción del nivel). Una vez que hayas iniciado sesión, ve a la página del siguiente nivel (Nivel 0 a Nivel 1) para averiguar cómo entrar al Nivel 1.

La contraseña para el siguiente nivel se almacena en un archivo llamado **readme** ubicado en el directorio de inicio del nivel 0. Usa esta contraseña para iniciar sesión en **bandit1** usando **SSH**. 

Siempre que encuentres una contraseña para un nivel, usa el **servicio SSH** (en el **puerto 2220**) para iniciar sesión en ese nivel y continuar el juego.

---------------

<h2 id="Exp0">Descripción de Comandos Usados</h2>

Estos son los comandos que usamos en este nivel:

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

Continuamos con la solución.

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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en un archivo llamado **-** ubicado en el directorio de inicio.

----------------

<h2 id="Exp1">Descripción de Comandos</h2>

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

| **Sintaxis $()** |
|:-----------:|
| *La sintaxis $() en Bash se utiliza para ejecutar un comando y capturar su salida. El resultado del comando que se ejecuta dentro de $() se reemplaza en línea por la salida de ese comando.* |

Forma de uso:
```bash
Comando $(comando_A_usar)
```

Ejemplos:
* Usando un comando dentro de un string de echo:
```bash
echo "Estoy en: $(pwd)"
Estoy en: /home/berserk/Desktop
```

<br>

Continuamos con la solución.

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
La contraseña es la que encontraste en el nivel 0.

Ya nos indicaron donde se encuentra la contraseña para el siguiente nivel. El problema es qué el nombre del archivo es un simple guion y si intentamos leerlo, nos dará un error:
```bash
bandit1@bandit:~$ whoami
bandit1
bandit1@bandit:~$ ls
-
bandit1@bandit:~$ cat -
^C
```

Ni me saco un error, por eso tuve que cancelar la acción. Para resolver esto, **Over The Wire** nos da una página con una solución:
* https://www.webservertalk.com/dashed-filename

En resumen, para leer el archivo, simplemente usamos el siguiente signo: **<**. 

Intentémoslo:
```bash
bandit1@bandit:~$ cat < -
...
```
¡Muy bien! Ya tenemos la contraseña para el siguiente nivel. Antes de ir a otro nivel, hay otras formas para poder ver esta clase de archivos, te las mostrare a continuación:

* Leer el archivo usando el PATH:
```bash
bandit1@bandit:~$ cat /home/bandit1/-
...
```

* Utilizar otros caracteres para leerlo
```bash
bandit1@bandit:~$ cat ./-
...
```

* Usando el comando pwd:
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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
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
La contraseña es la que encontraste en el nivel 1.

Ya sabemos que hay un archivo llamado **spaces** que tiene la contraseña, vamos a verlo:
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

* Leyendo todos los archivos que empiecen por s:
```bash
bandit2@bandit:~$ cat s*
...
```

* Usando el PATH para leer todos los archivos que empiecen en s:
```bash
bandit2@bandit:~$ cat /home/bandit2/s*
...
```

* Leyendo todos los archivos dentro de un PATH:
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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>

----------------

**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en un archivo oculto en el directorio **inhere**.

----------------

<h2 id="Exp3">Descripción de Comandos</h2>

| **Comando cd** |
|:-----------:|
| *Se utiliza para cambiar el directorio de trabajo actual en la terminal. Es la abreviatura de "change directory".* |

Formas de uso:
```bash
cd nombre_directorio
```

Ejemplos:
* Moviendonos al un directorio /home:
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

| **Comando xargs** |
|:-----------:|
| *Se utiliza para construir y ejecutar comandos a partir de la salida de otros comandos. Toma la salida de un comando (normalmente una lista de elementos) y la usa como argumentos para otro comando.* |

Formas de uso: 
```bash
find archivo.txt | xargs comando
```

Ejemplos:
* Leyendo un archivo que se encontro con el comando find:
```bash
find . -type f | xargs cat
```

* Eliminando un archivo encontrado con el comando find:
```bash
find . --name "*.tmp" | xargs rm
```

<br>

| **Tuberías** |
|:-----------:|
| *Representadas por el símbolo "|", se utilizan para conectar la salida de un comando directamente como entrada para otro comando. Esto permite encadenar varios comandos de manera eficiente y procesar datos en una secuencia.* |

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

Continuamos con la solución.

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
La contraseña es la que encontraste en el nivel 2.

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

* Usando find y xargs para imprimir el contenido de los archivos en el directorio actual:
```bash
find . -type f | xargs cat
```

* Buscando e imprimiendo un archivo que tenga el nombre "hidden":
```bash
find . -type f | grep "hidden" | xargs cat
```

¡Listo! Ya tenemos la contraseña para el siguiente nivel, sal y entra en el siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Nivel4" style="text-align:center;">Nivel 4 a 5</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el único archivo legible por humanos en el directorio **inhere**. Consejo: si su terminal está desordenada, intente con el comando **"reset"**.

----------------

<h2 id="Exp4">Descripción de Comandos</h2>

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

Continuamos con la solución.

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
La contraseña es la que encontraste en el nivel 3.

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

Mmmmm, los permisos indican que no se pueden ni leer con **cat**, a menos que los ejecutemos como programas. Entonces, veamos qué tipo de archivos son con el comando **file**:
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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel, se almacena en un archivo en algún lugar del directorio **inhere** y tiene todas las siguientes propiedades:
* legible por humanos
* 1033 bytes de tamaño
* no ejecutable

----------------

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
La contraseña es la que encontraste en el nivel 4.

Hay varias condiciones que tiene el archivo que contiene la contraseña, para buscarlo, usaremos el comando **find** con varios argumentos. Te comparto unos links con ejemplos de como usar el comando **find** y sobre los permisos de archivos:
* https://itsfoss.com/es/comando-find-linux/#buscar-varios-archivos-con-varias-extensiones-o-condici%C3%B3n
* https://www.hostinger.mx/tutoriales/como-usar-comando-find-locate-en-linux/#Busqueda_por_tipo
* https://gospelidea.com/blog/que-son-los-permisos-chmod

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

Demasiados directorios, prodríamos buscar nuestro archivo, pero eso no es óptimo, así que usemos el comando **find**. Le agregaremos los siguientes parámetros:
* -type f: Para que busque archivos.
* -size 1033c: Para que busque los archivos con 1033 bytes de tamaño. 
* -perm 640: Para que busque por archivos con permisos de no ejecución.
* grep ASCII: Para que con **grep**, busque archivos que sean legibles por humanos.

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
¡Listo! Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Nivel6" style="text-align:center;">Nivel 6 a 7</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en algún lugar del servidor y tiene todas las siguientes propiedades:
* propiedad del usuario bandit7
* propiedad del grupo bandit6
* 33 bytes de tamaño

----------------

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
La contraseña es la que encontraste en el nivel 5.

Hay varias condiciones, igual usaremos el comando **find** y usemos las mismas páginas de referencia que puse en el nivel anterior para guiarnos. Aunque me sirvió más esta página:
* https://www.ionos.mx/digitalguide/servidores/configuracion/comando-linux-find/

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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo **data.txt** junto a la palabra **millionth**.

----------------

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
La contraseña es la que encontraste en el nivel 6.

Como nos dice, hay un archivo llamado **data.txt** y la contraseña se encuentra junto a la palabra **millionth**, para encontrar esa palabra usaremos el comando **grep**, aquí te dejo este link con información muy útil sobre **grep**:
* https://geekland.eu/uso-del-comando-grep-en-linux-y-unix-con-ejemplos/

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
Y ahí está. Ya tenemos la contraseña, sal y entra al siguiente.


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Nivel8" style="text-align:center;">Nivel 8 a 9</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo **data.txt** y es la única línea de texto que aparece una sola vez.

----------------

<h2 id="Sol8">Descripción de Comandos</h2>

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

Continuamos con la solución.

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
La contraseña es la que encontraste en el nivel 7.

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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>

----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo **data.txt**, en una de las pocas cadenas legibles por humanos, precedida por varios caracteres **'='**.

----------------

<h2 id="Sol9">Descripción de Comandos</h2>

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

Continuamos con la solución

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
La contraseña es la que encontraste en el nivel 8.

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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo data.txt, que contiene datos codificados en base64.

----------------

<h2 id="Sol8">Descripción de Comandos</h2>

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

Continuamos con la solución.

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
La contraseña es la que encontraste en el nivel 9.

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
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>

----------------
**INSTRUCCIONES**:

La contraseña para el siguiente nivel se almacena en el archivo data.txt, donde todas las letras minúsculas (a-z) y mayúsculas (A-Z) se han rotado 13 posiciones.

----------------

<h2 id="Sol11">Descripción de Comandos</h2>

| **Comando tr** |
|:-----------:|
| *Se utiliza para traducir, reemplazar o eliminar caracteres en la entrada estándar. Es especialmente útil para transformar texto mediante la sustitución de ciertos caracteres o la eliminación de caracteres no deseados.* |

Formas de uso:
```bash
echo "texto Random" | tr 'caracteres a eliminar' 'caracteres a reemplazar'
```

Ejemplos:
* Reemplazando las mayusculas de un texto por minusculas:
```bash
echo "TEXTO EN MAYÚSCULAS" | tr 'A-Z' 'a-z'
```
* Eliminando espacios de un texto:
```bash
echo "Texto con  muchos espacios  ." | tr -d ' '
```

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
La contraseña es la que encontraste en el nivel 10.

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
* A-Z = N-ZA-M
* a-z = n-za-m

¡Muy bien! Ya tenemos la contraseña, sal y entra al siguiente.

# CONTINUARA...

<br>
<br>
<div style="position: relative;">
 <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
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
