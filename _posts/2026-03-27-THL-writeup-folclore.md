---
layout: single
title: Folclore - TheHackerLabs
excerpt: "."
date: 2026-03-30
classes: wide
header:
  teaser: /assets/images/THL-writeup-folclore/_logo.png
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
![](/assets/images/THL-writeup-folclore/_logo.png)

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
				<li><a href="#Hosts">Descubrimiento de Hosts</a></li>
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
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 09:53 -0600
...
Nmap scan report for 192.168.100.230
Host is up (0.0016s latency).
MAC Address: XX:XX:XX:XX:XX:XX (Oracle VirtualBox virtual NIC)
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.100.0/24
Interface: eth0, type: EN10MB, MAC: XX:XX:XX:XX:XX:XX, IPv4: Tu_IP
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
...
192.168.100.230	XX:XX:XX:XX:XX:XX	PCS Systemtechnik GmbH
...
```

Encontramos nuestro objetivo y es: `192.168.100.230`.

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.100.230
PING 192.168.100.230 (192.168.100.230) 56(84) bytes of data.
64 bytes from 192.168.100.230: icmp_seq=1 ttl=128 time=2.86 ms
64 bytes from 192.168.100.230: icmp_seq=2 ttl=128 time=0.978 ms
64 bytes from 192.168.100.230: icmp_seq=3 ttl=128 time=2.00 ms
64 bytes from 192.168.100.230: icmp_seq=4 ttl=128 time=1.16 ms

--- 192.168.100.230 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3045ms
rtt min/avg/max/mdev = 0.978/1.748/2.856/0.747 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.230 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 09:55 -0600
Initiating ARP Ping Scan at 09:55
Scanning 192.168.100.230 [1 port]
Completed ARP Ping Scan at 09:55, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 09:55
Scanning 192.168.100.230 [65535 ports]
Discovered open port 445/tcp on 192.168.100.230
Discovered open port 139/tcp on 192.168.100.230
Discovered open port 7680/tcp on 192.168.100.230
Completed SYN Stealth Scan at 09:56, 42.29s elapsed (65535 total ports)
Nmap scan report for 192.168.100.230
Host is up, received arp-response (0.0015s latency).
Scanned at 2026-03-27 09:55:25 CST for 43s
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE      REASON
139/tcp  open  netbios-ssn  syn-ack ttl 128
445/tcp  open  microsoft-ds syn-ack ttl 128
7680/tcp open  pando-pub    syn-ack ttl 128
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 42.48 seconds
           Raw packets sent: 131070 (5.767MB) | Rcvd: 6 (248B)
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

Solamente hay 3 puertos abiertos y me da curiosidad el **puerto 7680**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 139,445,7680 192.168.100.230 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 09:58 -0600
Nmap scan report for 192.168.100.230
Host is up (0.0024s latency).

PORT     STATE SERVICE       VERSION
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
7680/tcp open  pando-pub?
MAC Address: XX (Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
|_nbstat: NetBIOS name: FOLCLORE, NetBIOS user: <unknown>, NetBIOS MAC: XX (Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2026-03-27T15:59:06
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 102.81 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Si bien no vemos información sobre el **puerto 7680**, podemos ver algunos puntos sobre el **servicio SMB**:
* Parece que esta usando SMB 3.1.1, que es una versión moderna.
* El mensaje **"Message signing enabled but not required"**, quiere decir que requiere firmado en cada paquete enviado a **SMB**, sin embargo, menciona que esto es opcional, lo que permitiría ataques como **Fuerza Bruta**, **Ataque SMB Relay**, etc.

A su vez, parece que la explotación estara enfocada en **SMB**, así que comencemos con la enumeración.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="SMB">Enumeración de Servicio SMB</h2>

Primero veamos más información sobre este servicio con **netexec**:
```bash
nxc smb 192.168.100.230
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
```


Veamos si podemos enumerar los archivos compartidos con el usuario invitado:
```bash
smbmap -H 192.168.100.230 -u 'guest' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 0 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.100.230:445	Name: 192.168.100.230       	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dia de Muertos                                    	NO ACCESS	Al final, todos somos polvo y hueso.
	IPC$                                              	READ ONLY	IPC remota
	Libertad                                          	NO ACCESS	El secreto está en quien se atreve a buscarlo; sigue tu camino sin miedo.
	Lluvia                                            	NO ACCESS	Presta atención, pues en esta imagen el camino se revela sólo a quien sabe escuchar el trueno
	Oro                                               	NO ACCESS	¿Buscas llegar a Quetzalcóatl? Aquí inicia tu camino.
	Santuario                                         	NO ACCESS	El Refugio de Ixchel
	Viento                                            	NO ACCESS	El viento sagrado ejecutará la tarea a su debido tiempo
[*] Closed 1 connections
```
Excelente, vemos varios directorios, pero no tenemos acceso para ver alguno.

Podríamos tratar de aplicar fuerza bruta, pero desconocemos cuales son los usuarios existentes.

Es por esto que la mejor opción, es aplicar un **RID Brute Forcing**:

EXPLICACION

Al aplicarlo, podremos ver los usuarios existentes en la máquina y lo haremos con **netexec**:
```bash
nxc smb 192.168.100.230 -u 'guest' -p '' --rid-brute
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
SMB         192.168.100.230   445    FOLCLORE         [+] Folclore\guest: (Guest)
SMB         192.168.100.230   445    FOLCLORE         500: FOLCLORE\Administrador (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         501: FOLCLORE\Invitado (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         503: FOLCLORE\DefaultAccount (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         504: FOLCLORE\WDAGUtilityAccount (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         513: FOLCLORE\Ninguno (SidTypeGroup)
SMB         192.168.100.230   445    FOLCLORE         1001: FOLCLORE\Quetzalcoatl (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         1003: FOLCLORE\El_charro_negro (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         1004: FOLCLORE\Ix_Chel (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         1005: FOLCLORE\Tlaloc (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         1006: FOLCLORE\La_mulata_de_Cordoba (SidTypeUser)
SMB         192.168.100.230   445    FOLCLORE         1007: FOLCLORE\La_Catrina (SidTypeUser)
```
Excelente, tenemos a varios usuarios.

Intentemos aplicar Fuerza Bruta a algunos de estos.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="FuerzaBruta">Aplicando Fuerza Bruta al Servicio SMB</h2>


Probando entre varios usuarios, con un ataque de minimo 10 minutos, encontramos la contraseña del siguiente usuario:
```bash
nxc smb 192.168.100.230 -u 'El_charro_negro' -p /usr/share/wordlists/rockyou.txt --ignore-pw-decoding | grep -E '\[\+\]|STATUS_PASSWORD_EXPIRED'
SMB                      192.168.100.230   445    FOLCLORE         [-] Folclore\El_charro_negro:abc123 STATUS_PASSWORD_EXPIRED
```

```bash
nxc smb 192.168.100.230 -u 'El_charro_negro' -p 'abcd123'
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
SMB         192.168.100.230   445    FOLCLORE         [+] Folclore\El_charro_negro:abcd12
```


```bash
smbmap -H 192.168.100.230 -u 'El_charro_negro' -p 'abcd123' -r Oro --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.100.230:445	Name: lavashop.thl        	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dia de Muertos                                    	NO ACCESS	Al final, todos somos polvo y hueso.
	IPC$                                              	READ ONLY	IPC remota
	Libertad                                          	NO ACCESS	El secreto está en quien se atreve a buscarlo; sigue tu camino sin miedo.
	Lluvia                                            	NO ACCESS	Presta atención, pues en esta imagen el camino se revela sólo a quien sabe escuchar el trueno
	Oro                                               	READ, WRITE	¿Buscas llegar a Quetzalcóatl? Aquí inicia tu camino.
	./Oro
	dr--r--r--                0 Sun Mar 29 23:35:19 2026	.
	dr--r--r--                0 Sun Mar 29 23:35:19 2026	..
	fr--r--r--            50061 Mon Jul 28 18:40:26 2025	El_Charro_Negro.jpeg
	Santuario                                         	NO ACCESS	El Refugio de Ixchel
	Viento                                            	NO ACCESS	El viento sagrado ejecutará la tarea a su debido tiempo
```

```bash
smbmap -H 192.168.100.230 -u 'El_charro_negro' -p 'abcd123' --download Oro/El_charro_Negro.jpeg --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
[+] Starting download: Oro\El_charro_Negro.jpeg (50061 bytes)                                                            
[+] File output to: /../TheHackersLabs/Folclore/content/192.168.100.230-Oro_El_charro_Negro.jpeg
[*] Closed 1 connections

mv 192.168.100.230-Oro_El_charro_Negro.jpeg El_charro_Negro.jpeg
```

<h2 id=""></h2>


MOSTRAR IMAGEN

```bash
file El_charro_Negro.jpeg
El_charro_Negro.jpeg: JPEG image data, Exif standard: [TIFF image data, big-endian, direntries=2, copyright=El camino es largo pero no complicado. Busca a Ix Chel; ella tiene la siguiente pista aqui tienes la clave: 4+9Ii1wK], progressive, precision 8, 736x981, components 3
```

```bash
exiftool El_charro_Negro.jpeg
ExifTool Version Number         : 13.44
File Name                       : El_charro_Negro.jpeg
Directory                       : .
File Size                       : 50 kB
File Modification Date/Time     : 2026:03:29 23:35:40-06:00
File Access Date/Time           : 2026:03:29 23:35:59-06:00
File Inode Change Date/Time     : 2026:03:29 23:35:54-06:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Big-endian (Motorola, MM)
Y Cb Cr Positioning             : Centered
Copyright                       : El camino es largo pero no complicado. Busca a Ix Chel; ella tiene la siguiente pista aqui tienes la clave: 4+9Ii1wK
Image Width                     : 736
Image Height                    : 981
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 736x981
Megapixels                      : 0.722
```

```bash
nxc smb 192.168.100.230 -u 'Ix_Chel' -p 'ixchel123'
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
SMB         192.168.100.230   445    FOLCLORE         [+] Folclore\Ix_Chel:ixchel123
```


```bash
smbmap -H 192.168.100.230 -u 'Ix_Chel' -p 'ixchel123' -r Santuario --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.100.230:445	Name: lavashop.thl        	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dia de Muertos                                    	NO ACCESS	Al final, todos somos polvo y hueso.
	IPC$                                              	READ ONLY	IPC remota
	Libertad                                          	NO ACCESS	El secreto está en quien se atreve a buscarlo; sigue tu camino sin miedo.
	Lluvia                                            	NO ACCESS	Presta atención, pues en esta imagen el camino se revela sólo a quien sabe escuchar el trueno
	Oro                                               	NO ACCESS	¿Buscas llegar a Quetzalcóatl? Aquí inicia tu camino.
	Santuario                                         	READ, WRITE	El Refugio de Ixchel
	./Santuario
	dr--r--r--                0 Sun Mar 29 23:46:27 2026	.
	dr--r--r--                0 Sun Mar 29 23:46:27 2026	..
	fr--r--r--              573 Mon Jul 28 18:59:58 2025	La clave de Kukulcán.txt
	Viento                                            	NO ACCESS	El viento sagrado ejecutará la tarea a su debido tiempo
[*] Closed 1 connections
```

```bash
smbmap -H 192.168.100.230 -u 'Ix_Chel' -p 'ixchel123' --download 'Santuario/La clave de kukulcán.txt' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
[+] Starting download: Santuario\La clave de kukulcán.txt (573 bytes)
[+] File output to: /../TheHackersLabs/Folclore/content/192.168.100.230-Santuario_La clave de kukulcán.txt
[*] Closed 1 connections

mv 192.168.100.230-Santuario_La\ clave\ de\ kukulcán.txt La_clave_de_kukulcán.txt
```

```bash
cat La_clave_de_kukulcán.txt
Recuerdo cuando conocí a Kukulcán, la serpiente emplumada que otros llaman Quetzalcóatl; su mirada guardaba el misterio del viento y la sabiduría, y con una sonrisa me confió que su clave más preciada tenía solo ocho caracteres, sencilla pero poderosa, como él mismo. Nuestros caminos se entrelazan: yo, guardiana de la luna y la vida; él, portador del cielo y el conocimiento, unidos bajo el manto eterno de las estrellas. Ahora, sigue adelante, porque Quetzalcóatl te espera. Continua buscando a Tláloc,te despejare el camino entregándote su clave: 677Kn$q."
```

```bash
nxc smb 192.168.100.230 -u 'Tlaloc' -p 'tlaloc123'
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
SMB         192.168.100.230   445    FOLCLORE         [+] Folclore\Tlaloc:tlaloc123
```


```bash
smbmap -H 192.168.100.230 -u 'Tlaloc' -p 'tlaloc123' -r Lluvia --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 192.168.100.230:445	Name: lavashop.thl        	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dia de Muertos                                    	NO ACCESS	Al final, todos somos polvo y hueso.
	IPC$                                              	READ ONLY	IPC remota
	Libertad                                          	NO ACCESS	El secreto está en quien se atreve a buscarlo; sigue tu camino sin miedo.
	Lluvia                                            	READ, WRITE	Presta atención, pues en esta imagen el camino se revela sólo a quien sabe escuchar el trueno
	./Lluvia
	dr--r--r--                0 Mon Mar 30 00:33:39 2026	.
	dr--r--r--                0 Mon Mar 30 00:33:39 2026	..
	fr--r--r--           111216 Mon Jul 28 19:35:18 2025	Tlaloc.jpg
	Oro                                               	NO ACCESS	¿Buscas llegar a Quetzalcóatl? Aquí inicia tu camino.
	Santuario                                         	NO ACCESS	El Refugio de Ixchel
	Viento                                            	NO ACCESS	El viento sagrado ejecutará la tarea a su debido tiempo
[*] Closed 1 connections
```

```bash
smbmap -H 192.168.100.230 -u 'Tlaloc' -p 'tlaloc123' --download Lluvia/Tlaloc.jpg --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
[+] Starting download: Lluvia\Tlaloc.jpg (111216 bytes)                                                                  
[+] File output to: /../TheHackersLabs/Folclore/content/192.168.100.230-Lluvia_Tlaloc.jpg        
[*] Closed 1 connections

mv 192.168.100.230-Lluvia_Tlaloc.jpg Lluvia_Tlaloc.jpg
```

```bash
file Lluvia_Tlaloc.jpg
Lluvia_Tlaloc.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 736x731, components 3
```

```bash
stegseek Lluvia_Tlaloc.jpg /usr/share/wordlists/rockyou.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "knight"
[i] Original filename: "Las primeras señales.txt".
[i] Extracting to "Lluvia_Tlaloc.jpg.out".
```

```bash
cat Lluvia_Tlaloc.jpg.out
He conseguido estos últimos dos caracteres para la clave de Kukulcán, la serpiente emplumada que llaman Quetzalcóatl: %/. Te los entrego para que completes tu búsqueda. Además, aquí tienes otra clave, aunque debo confesar que no recuerdo a quién pertenece exactamente: 45yD#k7. Confía en tu instinto para usarla sabiamente.
```

```bash
nxc smb 192.168.100.230 -u usuarios.txt -p '45yD#k7' --no-bruteforce --continue-on-success
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
SMB         192.168.100.230   445    FOLCLORE         [-] Folclore\Administrador:45yD#k7 STATUS_LOGON_FAILURE 
SMB         192.168.100.230   445    FOLCLORE         [+] Folclore\Ninguno:45yD#k7 (Guest)
SMB         192.168.100.230   445    FOLCLORE         [-] Folclore\La_mulata_de_Cordoba:45yD#k7 STATUS_PASSWORD_EXPIRED 
SMB         192.168.100.230   445    FOLCLORE         [-] Folclore\La_Catrina:45yD#k7 STATUS_LOGON_FAILURE
```

```bash
nxc smb 192.168.100.230 -u 'La_mulata_de_Cordoba' -p 'cordoba123'
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
SMB         192.168.100.230   445    FOLCLORE         [+] Folclore\La_mulata_de_Cordoba:cordoba123
```

```bash
smbmap -H 192.168.100.230 -u 'La_mulata_de_Cordoba' -p 'cordoba123' -r Libertad --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 192.168.100.230:445	Name: lavashop.thl        	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dia de Muertos                                    	NO ACCESS	Al final, todos somos polvo y hueso.
	IPC$                                              	READ ONLY	IPC remota
	Libertad                                          	READ, WRITE	El secreto está en quien se atreve a buscarlo; sigue tu camino sin miedo.
	./Libertad
	dr--r--r--                0 Mon Mar 30 01:06:55 2026	.
	dr--r--r--                0 Mon Mar 30 01:06:55 2026	..
	fr--r--r--              169 Mon Jul 28 19:54:06 2025	Mi_amiga.txt
	fr--r--r--              432 Mon Jul 28 19:49:18 2025	Pacto.zip
	Lluvia                                            	NO ACCESS	Presta atención, pues en esta imagen el camino se revela sólo a quien sabe escuchar el trueno
	Oro                                               	NO ACCESS	¿Buscas llegar a Quetzalcóatl? Aquí inicia tu camino.
	Santuario                                         	NO ACCESS	El Refugio de Ixchel
	Viento                                            	NO ACCESS	El viento sagrado ejecutará la tarea a su debido tiempo
[*] Closed 1 connections
```

```bash
smbmap -H 192.168.100.230 -u 'La_mulata_de_Cordoba' -p 'cordoba123' --download Libertad/Mi_amiga.txt --no-banner
smbmap -H 192.168.100.23 -u 'La_mulata_de_Cordoba' -p 'cordoba123' --download Libertad/Pacto.zip --no-banner
```

```bash
cat Mi_amiga.txt
Mi amiga ‘La Catrina’ me ha dejado su cuenta para que le ayude con algunos de sus trabajitos de vez en cuando. Escribiré la clave aquí para no olvidarla: 3nM93{S#
```

```bash
7z x Pacto.zip

7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03
 64-bit locale=es_MX.UTF-8 Threads:6 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 432 bytes (1 KiB)

Extracting archive: Pacto.zip
--
Path = Pacto.zip
Type = zip
Physical Size = 432

    
Enter password (will not be echoed):
```

		      ^
APLICAR CRACKEO A ZIP |


```bash
nxc smb 192.168.100.230 -u 'La_Catrina' -p 'catrina123'
SMB         192.168.100.230   445    FOLCLORE         [*] Windows 10 / Server 2019 Build 19041 (name:FOLCLORE) (domain:Folclore) (signing:False) (SMBv1:None)
SMB         192.168.100.230   445    FOLCLORE         [+] Folclore\La_Catrina:catrina123
```

```bash
smbmap -H 192.168.100.230 -u 'La_Catrina' -p 'catrina123' -r 'Dia de Muertos' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.100.230:445	Name: lavashop.thl        	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dia de Muertos                                    	READ, WRITE	Al final, todos somos polvo y hueso.
	./Dia de Muertos
	dr--r--r--                0 Mon Mar 30 01:16:04 2026	.
	dr--r--r--                0 Mon Mar 30 01:16:04 2026	..
	fr--r--r--              278 Mon Jul 28 20:22:59 2025	Ultimo_mensaje.txt
	IPC$                                              	READ ONLY	IPC remota
	Libertad                                          	NO ACCESS	El secreto está en quien se atreve a buscarlo; sigue tu camino sin miedo.
	Lluvia                                            	NO ACCESS	Presta atención, pues en esta imagen el camino se revela sólo a quien sabe escuchar el trueno
	Oro                                               	NO ACCESS	¿Buscas llegar a Quetzalcóatl? Aquí inicia tu camino.
	Santuario                                         	NO ACCESS	El Refugio de Ixchel
	Viento                                            	NO ACCESS	El viento sagrado ejecutará la tarea a su debido tiempo
```

```bash
smbmap -H 192.168.100.230 -u 'La_Catrina' -p 'catrina123' --download 'Dia de Muertos/Ultimo_mensaje.txt' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
[+] Starting download: Dia de Muertos\Ultimo_mensaje.txt (278 bytes)                                                     
[+] File output to: /../TheHackersLabs/Folclore/content/192.168.100.230-Dia de Muertos_Ultimo_mensaje.txt
[*] Closed 1 connections

mv 192.168.100.230-Dia\ de\ Muertos_Ultimo_mensaje.txt Ultimo_mensaje.txt
```

```bash
cat Ultimo_mensaje.txt
Solo puedo darte el quinto carácter de su clave: ‘U’. Los demás caracteres que faltan deberás encontrarlos por tu cuenta, aunque no debe ser tan difícil; solo son números o letras. Confía en tu ingenio y no olvides que el misterio siempre se revela a quien persevera.
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
