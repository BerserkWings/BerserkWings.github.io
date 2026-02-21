---
layout: single
title: Welcome To The Jungle - TheHackerLabs
excerpt: "."
date: 2026-02-19
classes: wide
header:
  teaser: /assets/images/THL-writeup-WelcomeToTheJungle/wlcometothejungle.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Windows
  - 
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-WelcomeToTheJungle/wlcometothejungle.png)

texto

Herramientas utilizadas:
* *ping*
* *nmap*
* **
* **
* **
* **


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
				<li><a href="#"></a></li>
				<li><a href="#"></a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#"></a></li>
				<li><a href="#"></a></li>
			</ul>
		<li><a href="#Post">Post Explotación</a></li>
			<ul>
				<li><a href="#"></a></li>
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


<h2 id="Hosts">Descubrimiento de Hosts</h2>

Hagamos un descubrimiento de hosts para encontrar a nuestro objetivo.

Lo haremos primero con **nmap**:
```bash
nmap -sn 192.168.100.0/24
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-24 12:46 -0600
...
MAC Address: XX (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.100.200
Host is up (0.0012s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 2.10 seconds
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.100.0/24
Interface: eth0, type: EN10MB, MAC: XX, IPv4: Tu_IP
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.100.200	XX:XX:XX:XX:XX:XX	PCS Systemtechnik GmbH

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.211 seconds (115.78 hosts/sec). 3 responded
```

Encontramos nuestro objetivo y es: `192.168.100.200`.

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.100.200
PING 192.168.100.200 (192.168.100.200) 56(84) bytes of data.
64 bytes from 192.168.100.200: icmp_seq=1 ttl=128 time=2.27 ms
64 bytes from 192.168.100.200: icmp_seq=2 ttl=128 time=0.945 ms
64 bytes from 192.168.100.200: icmp_seq=3 ttl=128 time=1.03 ms
64 bytes from 192.168.100.200: icmp_seq=4 ttl=128 time=0.763 ms

--- 192.168.100.200 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3065ms
rtt min/avg/max/mdev = 0.763/1.251/2.268/0.594 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.200 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-19 17:39 -0600
Initiating ARP Ping Scan at 17:39
Scanning 192.168.100.200 [1 port]
Completed ARP Ping Scan at 17:39, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 17:39
Scanning 192.168.100.200 [65535 ports]
Discovered open port 139/tcp on 192.168.100.200
Discovered open port 135/tcp on 192.168.100.200
Discovered open port 80/tcp on 192.168.100.200
Discovered open port 445/tcp on 192.168.100.200
Discovered open port 49671/tcp on 192.168.100.200
Discovered open port 49668/tcp on 192.168.100.200
Discovered open port 49664/tcp on 192.168.100.200
Discovered open port 49665/tcp on 192.168.100.200
Discovered open port 49666/tcp on 192.168.100.200
Discovered open port 47001/tcp on 192.168.100.200
Discovered open port 49667/tcp on 192.168.100.200
Discovered open port 49670/tcp on 192.168.100.200
Discovered open port 5985/tcp on 192.168.100.200
Completed SYN Stealth Scan at 17:40, 31.83s elapsed (65535 total ports)
Nmap scan report for 192.168.100.200
Host is up, received arp-response (0.0010s latency).
Scanned at 2026-02-19 17:39:38 CST for 32s
Not shown: 57259 closed tcp ports (reset), 8263 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      REASON
80/tcp    open  http         syn-ack ttl 128
135/tcp   open  msrpc        syn-ack ttl 128
139/tcp   open  netbios-ssn  syn-ack ttl 128
445/tcp   open  microsoft-ds syn-ack ttl 128
5985/tcp  open  wsman        syn-ack ttl 128
47001/tcp open  winrm        syn-ack ttl 128
49664/tcp open  unknown      syn-ack ttl 128
49665/tcp open  unknown      syn-ack ttl 128
49666/tcp open  unknown      syn-ack ttl 128
49667/tcp open  unknown      syn-ack ttl 128
49668/tcp open  unknown      syn-ack ttl 128
49670/tcp open  unknown      syn-ack ttl 128
49671/tcp open  unknown      syn-ack ttl 128
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 32.06 seconds
           Raw packets sent: 159759 (7.029MB) | Rcvd: 57324 (2.293MB)
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

Vemos varios puertos abiertos, pero me da curiosidad el **puerto 80** que nos indica que hay una página web activa.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 80,135,139,445,5985,47001,49664,49665,49666,49667,49668,49670,49671 192.168.100.200 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-19 17:40 -0600
Nmap scan report for 192.168.100.200
Host is up (0.0013s latency).

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Welcome to the Jungle - The Hex Guns
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
MAC Address: XX (Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: THEHEXGUNS, NetBIOS user: <unknown>, NetBIOS MAC: XX (Oracle VirtualBox virtual NIC)
|_clock-skew: 11m32s
| smb2-time: 
|   date: 2026-02-19T23:53:00
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.44 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Analizando el escaneo de servicios, vemos un par de cosas interesantes:
* La página web activa en el **puerto 80**, no tiene un dominio asignado.
* Necesitaremos credenciales válidas para ver el **servicio SMB**, aunque por el mensaje de **"Message signing enables but not required"** nos puede permitir hacer **NTLM** o un **SMB Relay Attack**.

Por el momento, enfoquemonos en la página web, así que vamos a analizarla.


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
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura1.png">
</p>

Parecer ser una página dedicada a un grupo musical.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura2.png">
</p>

No hay mucho que destacar, salvo el uso de **PHP**.

Si observas en la parte superior derecha, podemos visitar una subpágina llamada **Albums**:

<p align="center">
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura3.png">
</p>

Pero al revisar el código fuente, podemos ver un mensaje que dejo alguien y mencionan un nombre:

<p align="center">
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura4.png">
</p>

Podríamos aplicarle **Fuerza Bruta** para ver si pertecene a algún servicio de la máquina víctima, pero esto no nos llevara a nada.

Apliquemos **Fuzzing** para saber si hay algún directorio o archivo oculto.

<br>

<h2 id="fuzz">Fuzzing</h2>

Al ver que se utiliza **PHP**, podemos enfocar una busqueda por esa extensión y algunas más.

Primero utilizaremos la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt:FUZZ -u http://192.168.100.200/FUZZ -t 300 -e .php,.txt,.html

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.100.200/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

media                   [Status: 301, Size: 161, Words: 9, Lines: 2, Duration: 240ms]
css                     [Status: 301, Size: 159, Words: 9, Lines: 2, Duration: 258ms]
img                     [Status: 301, Size: 159, Words: 9, Lines: 2, Duration: 246ms]
index.php               [Status: 200, Size: 1209, Words: 231, Lines: 29, Duration: 327ms]
header.php              [Status: 200, Size: 189, Words: 35, Lines: 7, Duration: 129ms]
albums.php              [Status: 200, Size: 1915, Words: 455, Lines: 42, Duration: 143ms]
footer.php              [Status: 200, Size: 81, Words: 13, Lines: 3, Duration: 358ms]
CSS                     [Status: 301, Size: 159, Words: 9, Lines: 2, Duration: 111ms]
Media                   [Status: 301, Size: 161, Words: 9, Lines: 2, Duration: 130ms]
Css                     [Status: 301, Size: 159, Words: 9, Lines: 2, Duration: 128ms]
IMG                     [Status: 301, Size: 159, Words: 9, Lines: 2, Duration: 79ms]
Img                     [Status: 301, Size: 159, Words: 9, Lines: 2, Duration: 82ms]
Index.php               [Status: 200, Size: 1209, Words: 231, Lines: 29, Duration: 176ms]
Footer.php              [Status: 200, Size: 81, Words: 13, Lines: 3, Duration: 103ms]
Header.php              [Status: 200, Size: 189, Words: 35, Lines: 7, Duration: 99ms]
MEDIA                   [Status: 301, Size: 161, Words: 9, Lines: 2, Duration: 111ms]
index.php               [Status: 200, Size: 1209, Words: 231, Lines: 29, Duration: 115ms]
.                       [Status: 200, Size: 1209, Words: 231, Lines: 29, Duration: 113ms]
Albums.php              [Status: 200, Size: 1915, Words: 455, Lines: 42, Duration: 119ms]
INDEX.php               [Status: 200, Size: 1209, Words: 231, Lines: 29, Duration: 105ms]
FOOTER.php              [Status: 200, Size: 81, Words: 13, Lines: 3, Duration: 147ms]
:: Progress: [514492/514492] :: Job [1/1] :: 2762 req/sec :: Duration: [0:03:12] :: Errors: 4 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-e*       | Para indicar una busqueda de archivos con extensiones específicas. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.100.200 -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 300 -x php,txt,html --ne
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.200
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
img                  (Status: 301) [Size: 159] [--> http://192.168.100.200/img/]
media                (Status: 301) [Size: 161] [--> http://192.168.100.200/media/]
index.php            (Status: 200) [Size: 1209]
css                  (Status: 301) [Size: 159] [--> http://192.168.100.200/css/]
header.php           (Status: 200) [Size: 189]
albums.php           (Status: 200) [Size: 1915]
footer.php           (Status: 200) [Size: 81]
CSS                  (Status: 301) [Size: 159] [--> http://192.168.100.200/CSS/]
Media                (Status: 301) [Size: 161] [--> http://192.168.100.200/Media/]
Css                  (Status: 301) [Size: 159] [--> http://192.168.100.200/Css/]
IMG                  (Status: 301) [Size: 159] [--> http://192.168.100.200/IMG/]
Img                  (Status: 301) [Size: 159] [--> http://192.168.100.200/Img/]
Index.php            (Status: 200) [Size: 1209]
MEDIA                (Status: 301) [Size: 161] [--> http://192.168.100.200/MEDIA/]
Footer.php           (Status: 200) [Size: 81]
Header.php           (Status: 200) [Size: 189]
error_log.html      (Status: 400) [Size: 324]
error_log.txt       (Status: 400) [Size: 324]
error_log.php       (Status: 400) [Size: 324]
error_log           (Status: 400) [Size: 324]
index.php            (Status: 200) [Size: 1209]
.                    (Status: 200) [Size: 1209]
Albums.php           (Status: 200) [Size: 1915]
INDEX.php            (Status: 200) [Size: 1209]
FOOTER.php           (Status: 200) [Size: 81]
Progress: 514492 / 514492 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*	     | Para indicar una busqueda de archivos con extensiones específicas. |
| *--ne*     | Para que no muestre errores. |

<br>

No encontramos gran cosa, pero si vemos un directorio que se llama `/media`.

Probemos si hay algún archivo oculto, pero aparte de usar las extensiones que usamos al inicio, agreguemos algunas más:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt -u http://192.168.100.200/media/FUZZ -t 300 -e .php,.txt,.html,.rar,.zip,.db,.bak -mc 200,301,302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.100.200/media/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt
 :: Extensions       : .php .txt .html .rar .zip .db .bak 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200,301,302
________________________________________________

songs.zip               [Status: 200, Size: 3918, Words: 23, Lines: 11, Duration: 136ms]
Songs.zip               [Status: 200, Size: 3918, Words: 23, Lines: 11, Duration: 124ms]
:: Progress: [1026960/1026960] :: Job [1/1] :: 3118 req/sec :: Duration: [0:06:01] :: Errors: 0 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-mc*      | Para aplicar un flitro que solo muestra resultados con un código de estado especifico. |
| *-e*       | Para indicar una busqueda de archivos específicos. |

<br>

Excelente, encontramos un **archivo ZIP** que podemos descargar.

Descargalo y descomprimelo:
```bash
unzip songs.zip
Archive:  songs.zip
  inflating: digital_destruction.txt  
  inflating: neon_rebellion.txt      
  inflating: paradaise_404.txt       
  inflating: solo_final.wav
```
Nos dio 3 archivos de texto y un **archivo WAV**, que es un audio que se puede escuchar.

Analicemos cada archivo.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Steganography">Identificando Esteganografia en Archivos Obtenidos de Archivo ZIP</h2>

Leyendo cada archivo de texto que obtuvimos, vemos que parecen ser letras de canciones:
```bash
cat digital_destruction.txt
Binary burns through the wires,
1s and 0s flying higher...
# nothing special here
                                                                                                                                                                                              
cat neon_rebellion.txt
Rise against the static tide,
firewalls can't stop our ride.
                                                                                                                                                                                              
cat paradaise_404.txt
They tried to hide, but we still found,
The jungle echoes with a sound...

There's always one password we’ve used since the first rehearsal...
```

Trate de escuchar el **archivo WAV** utilizando **Audicity**, pero parece que no contiene ningún audio:

<p align="center">
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura5.png">
</p>

Quizá tenga algún espectograma oculto, pero como no hay audio como tal, no veremos nada:

<p align="center">
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura6.png">
</p>

Viendo la información del archivo, si encontramos la palabra data, puede que contenga un mensaje o archivo oculto:
```bash
file solo_final.wav
solo_final.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 44100 Hz
```
Ahí esta **data**, hay una gran posibilidad de que se haya usado Esteganografía para ocultar algo en este archivo.

Probemos si la herramienta **stegseek** puede encontrar algo:
```bash
stegseek solo_final.wav
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Progress: 99.26% (132.5 MB)           
[!] error: Could not find a valid passphrase
```
Parece que si hay algo, pero necesita una contraseña.

Trate de utilizar **rockyou.txt**, pero no encontro nada, así que es muy probable que utilice alguna palabra que este dentro de la página web.

Este wordlist contiene palabras que pueden ser una contraseña:
```bash
cat contraseñas.txt
slash
Slash
SLASH
digital_destruction
digitaldestruction
DigitalDestruction
DIGITALDESCTRUION
neon_rebellion
neonrebellion
NeonRebellion
NEONREBELLION
paradaise_404
400paradise
400Paradise
400PARADISE
thehexguns
TheHexGuns
THEHEXGUNS
```

Probemos:
```bash
stegseek solo_final.wav contraseñas.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "thehexguns"
[i] Original filename: "password.txt".
[i] Extracting to "solo_final.wav.out".
```
Excelente, obtumos un archivo de texto.

Leamoslo:
```bash
cat solo_final.wav.out
Password:**********
URL:theh3xgun5
```
Tenemos una página web y una contraseña.

Entremos a esa págna:

<p align="center">
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura7.png">
</p>

Es un login y para este momento, ya tenemos un usuario y contraseña que podemos utilizar:

<p align="center">
<img src="/assets/images/THL-writeup-WelcomeToTheJungle/Captura8.png">
</p>

<br>

<h2 id="Binario">Analizando Binario commonlink.exe</h2>


```bash

```

```bash

```

```bash
strings commlink.exe | grep -n "password"
405:axl password: **********
```

```bash

```

```bash

```

```bash

```

```bash

```

```bash

```



<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id=""></h2>

```bash

```

```bash

```

```bash

```

```bash

```




<br>
<br>
<div style="position: relative;">
  <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


links


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
