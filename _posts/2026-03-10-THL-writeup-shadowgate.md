---
layout: single
title: Shadowgate - TheHackerLabs
excerpt: "."
date: 2026-03-12
classes: wide
header:
  teaser: /assets/images/THL-writeup-shadowgate/shadowgate.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Linux
  - SSH
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-shadowgate/shadowgate.png)

texto

Herramientas utilizadas:
* *ping*
* *nmap*
* *nc*
* *ffuf*
* *gobuster*
* *BurpSuite*
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
				<li><a href="#Puerto56789">Analizando Servicio Activo en Puerto 56789</a></li>
				<li><a href="#fuzz">Fuzzing</a></li>
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
ping -c 4 192.168.100.220
PING 192.168.100.220 (192.168.100.220) 56(84) bytes of data.
64 bytes from 192.168.100.220: icmp_seq=1 ttl=64 time=1.48 ms
64 bytes from 192.168.100.220: icmp_seq=2 ttl=64 time=0.731 ms
64 bytes from 192.168.100.220: icmp_seq=3 ttl=64 time=0.849 ms
64 bytes from 192.168.100.220: icmp_seq=4 ttl=64 time=0.913 ms

--- 192.168.100.220 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3045ms
rtt min/avg/max/mdev = 0.731/0.992/1.475/0.286 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.220 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-10 11:40 -0600
Initiating ARP Ping Scan at 11:40
Scanning 192.168.100.220 [1 port]
Completed ARP Ping Scan at 11:40, 0.44s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:40
Scanning 192.168.100.220 [65535 ports]
Discovered open port 22/tcp on 192.168.100.220
Discovered open port 8080/tcp on 192.168.100.220
Discovered open port 56789/tcp on 192.168.100.220
Completed SYN Stealth Scan at 11:40, 9.04s elapsed (65535 total ports)
Nmap scan report for 192.168.100.220
Host is up, received arp-response (0.0012s latency).
Scanned at 2026-03-10 11:40:32 CST for 9s
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE    REASON
22/tcp    open  ssh        syn-ack ttl 64
8080/tcp  open  http-proxy syn-ack ttl 64
56789/tcp open  unknown    syn-ack ttl 64
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 9.67 seconds
           Raw packets sent: 65546 (2.884MB) | Rcvd: 65546 (2.622MB
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

Encontramos 3 puertos abiertos, pero me da curiosidad el **puerto 8080** y el **puerto 56789**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,8080,56789 192.168.100.220 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-10 11:41 -0600
Nmap scan report for 192.168.100.220
Host is up (0.00080s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 84:05:fe:ed:47:16:ab:28:70:0f:44:6e:f6:8d:0c:6f (ECDSA)
|_  256 99:a9:88:76:ee:c8:ed:ce:73:57:2a:22:da:9f:7b:7e (ED25519)
8080/tcp  open  http    Werkzeug httpd 3.1.3 (Python 3.12.3)
|_http-server-header: Werkzeug/3.1.3 Python/3.12.3
|_http-title: 403 Forbidden
56789/tcp open  unknown
| fingerprint-strings: 
|   DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, RPCCheck, RTSPRequest: 
|     Shadow Gate v1.0 :: Not all doors are locked. Some wait for n0cturne. Listen closely
|     patterns hide in the noise. Sequence always matters.
|     Access denied. The gate remains closed.
|   NULL: 
|     Shadow Gate v1.0 :: Not all doors are locked. Some wait for n0cturne. Listen closely
|     patterns hide in the noise. Sequence always matters.
|_    Listening... but no response.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port56789-TCP:V=7.98%I=7%D=3/10%Time=69B057B5%P=x86_64-pc-linux-gnu%r(N
SF:ULL,AA,"Shadow\x20Gate\x20v1\.0\x20::\x20Not\x20all\x20doors\x20are\x20
SF:locked\.\x20Some\x20wait\x20for\x20n0cturne\.\x20Listen\x20closely\xe2\
SF:x80\x94patterns\x20hide\x20in\x20the\x20noise\.\x20Sequence\x20always\x
SF:20matters\.\nListening\.\.\.\x20but\x20no\x20response\.\n")%r(GenericLi
SF:nes,B4,"Shadow\x20Gate\x20v1\.0\x20::\x20Not\x20all\x20doors\x20are\x20
SF:locked\.\x20Some\x20wait\x20for\x20n0cturne\.\x20Listen\x20closely\xe2\
SF:x80\x94patterns\x20hide\x20in\x20the\x20noise\.\x20Sequence\x20always\x
SF:20matters\.\nAccess\x20denied\.\x20The\x20gate\x20remains\x20closed\.\n
SF:")%r(GetRequest,B4,"Shadow\x20Gate\x20v1\.0\x20::\x20Not\x20all\x20door
SF:s\x20are\x20locked\.\x20Some\x20wait\x20for\x20n0cturne\.\x20Listen\x20
SF:closely\xe2\x80\x94patterns\x20hide\x20in\x20the\x20noise\.\x20Sequence
SF:\x20always\x20matters\.\nAccess\x20denied\.\x20The\x20gate\x20remains\x
SF:20closed\.\n")%r(HTTPOptions,B4,"Shadow\x20Gate\x20v1\.0\x20::\x20Not\x
SF:20all\x20doors\x20are\x20locked\.\x20Some\x20wait\x20for\x20n0cturne\.\
SF:x20Listen\x20closely\xe2\x80\x94patterns\x20hide\x20in\x20the\x20noise\
SF:.\x20Sequence\x20always\x20matters\.\nAccess\x20denied\.\x20The\x20gate
SF:\x20remains\x20closed\.\n")%r(RTSPRequest,B4,"Shadow\x20Gate\x20v1\.0\x
SF:20::\x20Not\x20all\x20doors\x20are\x20locked\.\x20Some\x20wait\x20for\x
SF:20n0cturne\.\x20Listen\x20closely\xe2\x80\x94patterns\x20hide\x20in\x20
SF:the\x20noise\.\x20Sequence\x20always\x20matters\.\nAccess\x20denied\.\x
SF:20The\x20gate\x20remains\x20closed\.\n")%r(RPCCheck,B4,"Shadow\x20Gate\
SF:x20v1\.0\x20::\x20Not\x20all\x20doors\x20are\x20locked\.\x20Some\x20wai
SF:t\x20for\x20n0cturne\.\x20Listen\x20closely\xe2\x80\x94patterns\x20hide
SF:\x20in\x20the\x20noise\.\x20Sequence\x20always\x20matters\.\nAccess\x20
SF:denied\.\x20The\x20gate\x20remains\x20closed\.\n")%r(DNSVersionBindReqT
SF:CP,B4,"Shadow\x20Gate\x20v1\.0\x20::\x20Not\x20all\x20doors\x20are\x20l
SF:ocked\.\x20Some\x20wait\x20for\x20n0cturne\.\x20Listen\x20closely\xe2\x
SF:80\x94patterns\x20hide\x20in\x20the\x20noise\.\x20Sequence\x20always\x2
SF:0matters\.\nAccess\x20denied\.\x20The\x20gate\x20remains\x20closed\.\n"
SF:);
MAC Address: XX (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.69 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Después de analizar los escaneos, esto es lo más destacado:
* La página web activa en el **puerto 8080** parece que solo muestra el **código de estado 403**, por lo que puede que no tengamos acceso a la página o no estamos entrando a la ruta correcta.
* El servicio del **puerto 56789** muestra algunos mensajes y parece que podemos conectarnos a este.

Antes de ir a la página web, investiguemos un poco más sobre el servicio del **puerto 56789**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Puerto56789">Analizando Servicio Activo en Puerto 56789</h2>

Probemos a conectarnos a ese puerto usando **netcat**:
```bash
nc 192.168.100.220 56789
Shadow Gate v1.0 :: Not all doors are locked. Some wait for n0cturne. Listen closely—patterns hide in the noise. Sequence always matters.

Access denied. The gate remains closed
```
Necesitamos alguna clave o una contraseña para que nos dé acceso a lo que sea que muestre.

Curiosamente, el mensaje tiene la frase **n0cturne**, que parece más una contraseña o clave agregada intencionalmente.

Probémosla:
```bash
nc 192.168.100.220 56789
Shadow Gate v1.0 :: Not all doors are locked. Some wait for n0cturne. Listen closely—patterns hide in the noise. Sequence always matters.
n0cturne
Shadow Gate v1.0 :: Not all doors are locked. Some wait for n0cturne. Listen closely—patterns hide in the noise. Sequence always matters.

YzRmN2QzNQ==
u6wLUZqVjHq9zqkk1AUGKQ==
Bh5flJ0ipCSwSEFszeH3JQ==
i0xh/Dcf2rhD+aN+gTsRtg==
7qxQxcO+lgqcUMwUs2dJdw==
Z0ypLNpNI3eYPsHjf1cwIA==
SFHenVqViVg4OzTZqHEqog==
j8tFlkw7uB6pJZkeWhOKug==
qeSKHCpwYueNcMwnC5izWg==
Connection closing...
```
Obtuvimos respuesta y vemos varios strings codificados en **base64**.

Al tratar de decodificarlas, no podremos leer el contenido, ya que verás bytes crudos que nos dan a entender que los datos están cifrados:
```bash
echo -n "YzRmN2QzNQ==" | base64 -d
c4f7d35
echo -n "u6wLUZqVjHq9zqkk1AUGKQ==" | base64 -d
��
  Q���z�Ω$�)                                                                                                                                                                                              
echo -n "Bh5flJ0ipCSwSEFszeH3JQ==" | base64 -d
_��"�$�HAl���%
...
```

También los podemos decodificar con la herramienta **CyberChef**:
* <a href="https://gchq.github.io/CyberChef/" target="_blank">CyberChef</a>

Pruébalo:

<p align="center">
<img src="/assets/images/THL-writeup-shadowgate/Captura1.png">
</p>

Si volvemos a conectarnos a este servicio y lo ejecutamos, nos darán distintos resultados, a excepción del primero, ya que parece que ese no cambia nunca:
```bash
echo -n "YzRmN2QzNQ==" | base64 -d
c4f7d35
```
Tomemos en cuenta esto para más adelante.

Investiguemos la página web, quizá encontremos una ruta válida si aplicamos **Fuzzing**.

<br>

<h2 id="fuzz">Fuzzing</h2>

Primero probemos con la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt:FUZZ -u http://192.168.100.220:8080/FUZZ -fs 213

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.100.220:8080/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 213
________________________________________________

login                   [Status: 200, Size: 39, Words: 7, Lines: 1, Duration: 49ms]
:: Progress: [128623/128623] :: Job [1/1] :: 295 req/sec :: Duration: [0:06:52] :: Errors: 1 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-fs*	     | Para aplicar un filtro que no muestre respuestas de un tamaño específico. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.100.220:8080 -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -s 200,301 -b "" --ne
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:            http://192.168.100.220:8080
[+] Method:         GET
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt
[+] Status codes:   200,301
[+] User Agent:     gobuster/3.8.2
[+] Timeout:        10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
login                (Status: 200) [Size: 39]
Progress: 128623 / 128623 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-s*	     | Para aplicar un filtro que muestre solo los códigos de estado específicos. |
| *-b*	     | Para que funcione el filtro anterior. |
| *--ne*     | Para que no muestre errores. |

<br>

Muy bien, en ambas parece que encontramos un login.

Pero al entrar nos da este mensaje:

<p align="center">
<img src="/assets/images/THL-writeup-shadowgate/Captura2.png">
</p>

Al mencionar un método incorrecto, supongo que debe ser por método **POST** para poder agregar un usuario y contraseña.

Podemos probarlo con **BurpSuite**, capturando esta petición y mandándola al **Repeater**, pero recuerda cambiar la petición cuando la tenga en el **Repeater** para que se convierta 
en **petición POST**.

Normalmente, estos datos se ponen con los parámetros `username=` y `password=`:

<p align="center">
<img src="/assets/images/THL-writeup-shadowgate/Captura3.png">
</p>

Se acepta la petición, pero el usuario es incorrecto.

Ahora debemos buscar un usuario válido, pero todo parece indicar que quien oculta este dato será el servicio del **puerto 56789**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
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
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id=""></h2>





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
