---
layout: single
title: NodeCeption - TheHackerLabs
excerpt: "."
date: 2025-09-19
classes: wide
header:
  teaser: /assets/images/THL-writeup-nodeCeption/NodeCeption.jpg
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Easy Machine
tags:
  - Linux
  - 
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-nodeCeption/NodeCeption.jpg)

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


<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.100.120
PING 192.168.100.120 (192.168.100.120) 56(84) bytes of data.
64 bytes from 192.168.100.120: icmp_seq=1 ttl=64 time=1.54 ms
64 bytes from 192.168.100.120: icmp_seq=2 ttl=64 time=0.860 ms
64 bytes from 192.168.100.120: icmp_seq=3 ttl=64 time=0.882 ms
64 bytes from 192.168.100.120: icmp_seq=4 ttl=64 time=0.888 ms

--- 192.168.100.120 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3090ms
rtt min/avg/max/mdev = 0.860/1.043/1.543/0.288 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.120 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 11:20 CST
Initiating ARP Ping Scan at 11:20
Scanning 192.168.100.120 [1 port]
Completed ARP Ping Scan at 11:20, 0.08s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:20
Scanning 192.168.100.120 [65535 ports]
Discovered open port 22/tcp on 192.168.100.120
Discovered open port 5678/tcp on 192.168.100.120
Discovered open port 8765/tcp on 192.168.100.120
Completed SYN Stealth Scan at 11:20, 7.62s elapsed (65535 total ports)
Nmap scan report for 192.168.100.120
Host is up, received arp-response (0.0078s latency).
Scanned at 2025-09-19 11:20:33 CST for 8s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE        REASON
22/tcp   open  ssh            syn-ack ttl 64
5678/tcp open  rrac           syn-ack ttl 64
8765/tcp open  ultraseek-http syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 7.90 seconds
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

Hay 3 puertos abiertos, pero 2 de ellos no los he visto antes.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,5678,8765 192.168.100.120 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-19 11:21 CST
Nmap scan report for 192.168.100.120
Host is up (0.00086s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 67:df:2b:e6:48:16:ec:91:b1:a6:67:25:37:05:fc:0f (ECDSA)
|_  256 dc:ab:74:b7:be:b5:49:6a:c8:7b:db:6b:7c:91:73:69 (ED25519)
5678/tcp open  rrac?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Accept-Ranges: bytes
|     Cache-Control: public, max-age=86400
|     Last-Modified: Fri, 19 Sep 2025 17:14:41 GMT
|     ETag: W/"7b7-19962f890cd"
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1975
|     Vary: Accept-Encoding
|     Date: Fri, 19 Sep 2025 17:21:19 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <script type="module" crossorigin src="/assets/polyfills-B8p9DdqU.js"></script>
|     <meta charset="utf-8" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge" />
|     <meta name="viewport" content="width=device-width,initial-scale=1.0" />
|     <link rel="icon" href="/favicon.ico" />
|     <style>@media (prefers-color-scheme: dark) { body { background-color: rgb(45, 46, 46) } }</style>
|     <script type="text/javascript">
|     window.BASE_PATH = '/';
|     window.REST_ENDPOINT = 'rest';
|     </script>
|     <script src="/rest/sentry.js"></script>
|     <script>!function(t,e){var o,n,
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Vary: Accept-Encoding
|     Date: Fri, 19 Sep 2025 17:21:19 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
8765/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5678-TCP:V=7.95%I=7%D=9/19%Time=68CD910C%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,8DC,"HTTP/1\.1\x20200\x20OK\r\nAccept-Ranges:\x20bytes\r\nCach
SF:e-Control:\x20public,\x20max-age=86400\r\nLast-Modified:\x20Fri,\x2019\
SF:x20Sep\x202025\x2017:14:41\x20GMT\r\nETag:\x20W/\"7b7-19962f890cd\"\r\n
SF:Content-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x201975
SF:\r\nVary:\x20Accept-Encoding\r\nDate:\x20Fri,\x2019\x20Sep\x202025\x201
SF:7:21:19\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html
SF:\x20lang=\"en\">\n\t<head>\n\t\t<script\x20type=\"module\"\x20crossorig
SF:in\x20src=\"/assets/polyfills-B8p9DdqU\.js\"></script>\n\n\t\t<meta\x20
SF:charset=\"utf-8\"\x20/>\n\t\t<meta\x20http-equiv=\"X-UA-Compatible\"\x2
SF:0content=\"IE=edge\"\x20/>\n\t\t<meta\x20name=\"viewport\"\x20content=\
SF:"width=device-width,initial-scale=1\.0\"\x20/>\n\t\t<link\x20rel=\"icon
SF:\"\x20href=\"/favicon\.ico\"\x20/>\n\t\t<style>@media\x20\(prefers-colo
SF:r-scheme:\x20dark\)\x20{\x20body\x20{\x20background-color:\x20rgb\(45,\
SF:x2046,\x2046\)\x20}\x20}</style>\n\t\t<script\x20type=\"text/javascript
SF:\">\n\t\t\twindow\.BASE_PATH\x20=\x20'/';\n\t\t\twindow\.REST_ENDPOINT\
SF:x20=\x20'rest';\n\t\t</script>\n\t\t<script\x20src=\"/rest/sentry\.js\"
SF:></script>\n\t\t<script>!function\(t,e\){var\x20o,n,")%r(HTTPOptions,18
SF:3,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Policy:\x20defa
SF:ult-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nContent-Type
SF::\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r\nVary:\x20
SF:Accept-Encoding\r\nDate:\x20Fri,\x2019\x20Sep\x202025\x2017:21:19\x20GM
SF:T\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en
SF:\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</title>\n</head>
SF:\n<body>\n<pre>Cannot\x20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RTS
SF:PRequest,183,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Poli
SF:cy:\x20default-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nC
SF:ontent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r
SF:\nVary:\x20Accept-Encoding\r\nDate:\x20Fri,\x2019\x20Sep\x202025\x2017:
SF:21:19\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x
SF:20lang=\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</titl
SF:e>\n</head>\n<body>\n<pre>Cannot\x20OPTIONS\x20/</pre>\n</body>\n</html
SF:>\n");
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.32 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

En los puertos desconocidos, tenemos 2 páginas web activas.

Viendo el escaneo de la página web del **puerto 5678**, podemos identificar el endpoint `/rest` que supongo tiene que ver con la tecnología **API REST**.

Del **puerto 8765**, solamente nos muestra la página por defecto de **Apache2**.

Vamos a analizar primero la página web del **puerto 5678** y luego, la página web del **puerto 8765**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Puerto5678">Analizando Página Web Activa en el Puerto 5678</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura1.png">
</p>

Entramos en un login y vemos que es la plataforma **n8n**.

| **Plataforma n8n** |
|:------------------:|
| *n8n es una plataforma de automatización visual de código abierto para crear flujos de trabajo que conectan aplicaciones y servicios, automatizando así tareas repetitivas y moviendo datos entre ellos, todo ello sin necesidad de codificación y con la opción de autoalojamiento. Es una alternativa a herramientas como Zapier y Make que permite integrar más de 400 aplicaciones y servicios mediante nodos y visualmente, para escalar operaciones empresariales* |

<br>

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura2.png">
</p>

Son bastantes las tecnologías que se están usando, pero no veo algo que nos ayude de momento.

Si revisamos el código fuente, veremos el endpoint `/rest` que menciona el escaneo de servicios:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura3.png">
</p>

Al capturar con **BurpSuite** la petición de inicio de sesión del login, veremos otro endpoint y que la data es enviada en formato **JSON**:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura4.png">
</p>

Esto refuerza más el hecho de que nos enfrentamos ante una **API**.

Por último, vemos el mensaje de error ante un inicio de sesión fallido:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura5.png">
</p>

No encontraremos algo más, así que vayamos a analizar la otra página web.

<br>

<h2 id="Puerto8765">Analizando Página Web Activa en el Puerto 8765</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura6.png">
</p>

Solamente es la página por defecto de **Apache2**.

Revisando el código fuente, nos encontraremos un mensaje oculto de algún desarrollador:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura7.png">
</p>

Tenemos un posible usuario y una pista sobre la contraseña que debemos usar.

No encontraremos algo más aquí, entonces, aplicaremos **Fuzzing** a ambas páginas para ver que cosillas encontramos.

<br>

<h2 id="fuzzPuerto5678">Fuzzing a Página Web del Puerto 5678</h2>

Usando **ffuf** para descubrir directorios ocultos, solamente encontraremos estos directorios:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://192.168.100.120:5678/FUZZ -t 300

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.100.120:5678/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

static                  [Status: 301, Size: 156, Words: 6, Lines: 11, Duration: 1433ms]
assets                  [Status: 301, Size: 156, Words: 6, Lines: 11, Duration: 2024ms]
types                   [Status: 301, Size: 155, Words: 6, Lines: 11, Duration: 133ms]
                        [Status: 200, Size: 1975, Words: 85, Lines: 32, Duration: 243ms]
:: Progress: [220545/220545] :: Job [1/1] :: 1011 req/sec :: Duration: [0:03:01] :: Errors: 0 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |

<br>

Al intentar entrar a cada uno, nos dirá que no puede por **peticiones GET**:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura8.png">
</p>

Ahora, si enfocamos el **Fuzzing** en el endpoint `/rest`, veremos más directorios que antes, pero no tendremos acceso a estos:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://192.168.100.120:5678/rest/FUZZ/ -t 300

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.100.120:5678/rest/FUZZ/
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

login                   [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 49ms]
projects                [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 117ms]
tags                    [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 32ms]
license                 [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 32ms]
Login                   [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 59ms]
users                   [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 775ms]
Projects                [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 137ms]
settings                [Status: 200, Size: 3448, Words: 1, Lines: 1, Duration: 159ms]
LICENSE                 [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 116ms]
Users                   [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 40ms]
License                 [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 160ms]
push                    [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 159ms]
PROJECTS                [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 148ms]
Push                    [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 198ms]
roles                   [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 153ms]
SETTINGS                [Status: 200, Size: 3448, Words: 1, Lines: 1, Duration: 94ms]
variables               [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 88ms]
PUSH                    [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 213ms]
LogIn                   [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 116ms]
Tags                    [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 159ms]
LOGIN                   [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 48ms]
credentials             [Status: 401, Size: 43, Words: 1, Lines: 1, Duration: 135ms]
:: Progress: [220545/220545] :: Job [1/1] :: 913 req/sec :: Duration: [0:03:21] :: Errors: 0 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.100.120:5678/rest -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.120:5678/rest
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/users                (Status: 401) [Size: 43]
/login                (Status: 401) [Size: 43]
/projects             (Status: 401) [Size: 43]
/license              (Status: 401) [Size: 43]
/tags                 (Status: 401) [Size: 43]
/Login                (Status: 401) [Size: 43]
/Projects             (Status: 401) [Size: 43]
/settings             (Status: 200) [Size: 3448]
/LICENSE              (Status: 401) [Size: 43]
/Users                (Status: 401) [Size: 43]
/License              (Status: 401) [Size: 43]
/push                 (Status: 401) [Size: 43]
/PROJECTS             (Status: 401) [Size: 43]
/Push                 (Status: 401) [Size: 43]
/roles                (Status: 401) [Size: 43]
/SETTINGS             (Status: 200) [Size: 3448]
/variables            (Status: 401) [Size: 43]
/PUSH                 (Status: 401) [Size: 43]
/LogIn                (Status: 401) [Size: 43]
/Tags                 (Status: 401) [Size: 43]
/LOGIN                (Status: 401) [Size: 43]
/credentials          (Status: 401) [Size: 43]
Progress: 220544 / 220544 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |

<br>

En ambos casos se encontraron los mismos endpoints y solamente tendremos acceso al `/settings`:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura9.png">
</p>

Gracias a este endpoint, podemos obtener algunas cosillas como:
* La versión de la **plataforma n8n** que es **1.102.4**.
* La existencia del **API público**, es decir, endpoint `/api` activo.
* La API Key pública.

Y otras cosillas más, pero para poder ver otros endpoints, necesitamos estar logueados en la **plataforma n8n**, lo cual aun no es posible.

<br>

<h2 id="fuzzPuerto8765">Fuzzing a Página Web del Puerto 8765</h2>

Al no encontrar algún directorio oculto, enfocamos la busqueda en archivos ocultos:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://192.168.100.120:8765/FUZZ -t 300 -e .txt,.html,.php

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.100.120:8765/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Extensions       : .txt .html .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

index.html              [Status: 200, Size: 10833, Words: 3517, Lines: 369, Duration: 57ms]
login.php               [Status: 200, Size: 1723, Words: 637, Lines: 75, Duration: 9131ms]
.html                   [Status: 403, Size: 282, Words: 20, Lines: 10, Duration: 20ms]
                        [Status: 200, Size: 10833, Words: 3517, Lines: 369, Duration: 20ms]
.php                    [Status: 403, Size: 282, Words: 20, Lines: 10, Duration: 20ms]
server-status           [Status: 403, Size: 282, Words: 20, Lines: 10, Duration: 53ms]
:: Progress: [882180/882180] :: Job [1/1] :: 265 req/sec :: Duration: [0:07:52] :: Errors: 2 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-e*       | Para indicar una busqueda de archivos específicos. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.100.120:8765 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300 -x txt,html,php
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.120:8765
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10833]
/login.php            (Status: 200) [Size: 1723]
/server-status        (Status: 403) [Size: 282]
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

En ambos casos encontramos un login al que podemos entrar:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura10.png">
</p>

En su código fuente no encontraremos algo relevante, pero si capturamos una petición de inicio de sesión veremos que hay una relación con el login de la **plataforma n8n**:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura11.png">
</p>

Y por supuesto, tenemos el mensaje de error:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura12.png">
</p>

Por último, este login no parece ser vulnerable a **Inyecciones SQL**, por lo que si o si debemos darle un usuario y contraseña válidos.

Nos queda aplicar fuerza bruta en ambos logins.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="FuerzaBruta">Aplicando Fuerza Bruta a Cada Login</h2>

Antes de aplicar la fuerza bruta, debemos crear nuestro wordlist que debe tener las siguientes caracterizticas:
* Debe tener minimo 8 caracteres.
* Debe tener al menos 1 letra mayúscula.
* Debe tener al menos 1 número.

Podemos usar expresiones regulares contra el wordlist rockyou.txt, para obtener un nuevo wordlist que cumpla con estas caracterizticas.

Lo haremos con el siguiente comando:
```bash
grep -P '^(?=.{8,}$)(?=.*[A-Z])(?=.*\d).*$' /usr/share/wordlists/rockyou.txt > rockyouMatch.txt
```
* 
* 
* 

Entonces, para realizar el ataque a cada login, debemos de darle lo siguiente:
* A que lugar será dirigido el ataque, un servicio, un login web, etc.
* Los parámetros que se usan para enviar el usuario y contraseña.
* Y un filtro que indique un resultado correcto, ya sea que evite resultados con cierto texto, que solo acepte resultados que nos redirijan (obteniendo el código de estado 302), etc. Podemo usar un filtro que evite respuestas que den mensajes de error.

Pero aquí nos enfrentamos a un problema, pues al aplicar la fuerza bruta al login de la página web del **puerto 5678**, nos dará falsos positivos:
```bash
hydra -l 'usuario@maildelctf.com' -P ./rockyouMatch.txt 192.168.100.120 -s 5678 http-form-post "/rest/login:emailOrLdapLoginId=^USER^&password=^PASS^:F=Wrong username or password. Do you have caps lock on?"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-19 15:15:22
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 611309 login tries (l:1/p:611309), ~38207 tries per task
[DATA] attacking http-post-form://192.168.100.120:5678/rest/login:emailOrLdapLoginId=^USER^&password=^PASS^:F=Wrong username or password. Do you have caps lock on?
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: BABYGIRL1
[ERROR] the target is using HTTP auth, not a web form, received HTTP error code 401. Use module "http-get" instead.
[ERROR] the target is using HTTP auth, not a web form, received HTTP error code 401. Use module "http-get" instead.
[ERROR] the target is using HTTP auth, not a web form, received HTTP error code 401. Use module "http-get" instead.
[ERROR] the target is using HTTP auth, not a web form, received HTTP error code 401. Use module "http-get" instead.
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: Blink182
[ERROR] the target is using HTTP auth, not a web form, received HTTP error code 401. Use module "http-get" instead.
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: P@ssw0rd
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: Password1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: Michael1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: !QAZ2wsx
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: PRINCESS1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: Princess1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: ANTHONY1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: ILOVEYOU1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: ILOVEYOU2
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: Charlie1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: MYSPACE1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: PASSWORD1
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: Passw0rd
[5678][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: JORDAN23
1 of 1 target successfully completed, 16 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-19 15:15:33
```
Además, el aplicar fuerza bruta puede desactivar el login, pues detectara muchas peticiones realizadas, lo que me da a entender que hay un limite de intentos:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura13.png">
</p>

En cambio, la fuerza bruta se podra aplicar en el login de la página web del **puerto 8765**, sin ningún problema:
```bash
hydra -l 'usuario@maildelctf.com' -P ./rockyouMatch.txt 192.168.100.120 -s 8765 http-post-form "/login.php:email=^USER^&password=^PASS^:F=Credenciales incorrectas."
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-19 15:09:20
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 611309 login tries (l:1/p:611309), ~38207 tries per task
[DATA] attacking http-post-form://192.168.100.120:8765/login.php:email=^USER^&password=^PASS^:F=Credenciales incorrectas.
[8765][http-post-form] host: 192.168.100.120   login: usuario@maildelctf.com   password: ********
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-19 15:09:31
```

Y si probamos esa contraseña, nos darán un mensaje interesante:

<p align="center">
<img src="/assets/images/THL-writeup-nodeCeption/Captura14.png">
</p>

```bash

```

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
