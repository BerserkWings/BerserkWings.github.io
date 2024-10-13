---
layout: single
title: Manager - Hack The Box
excerpt: "."
date: 2024-10-12
classes: wide
header:
  teaser: /assets/images/htb-writeup-manager/Manager.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Medium Machine
tags:
  - Windows
  - Active Directory
  - 
  - 
---
![](/assets/images/htb-writeup-manager/Manager.png)

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
ping -c 4 10.10.11.236
PING 10.10.11.236 (10.10.11.236) 56(84) bytes of data.
64 bytes from 10.10.11.236: icmp_seq=1 ttl=127 time=67.8 ms
64 bytes from 10.10.11.236: icmp_seq=2 ttl=127 time=71.0 ms
64 bytes from 10.10.11.236: icmp_seq=3 ttl=127 time=68.2 ms
64 bytes from 10.10.11.236: icmp_seq=4 ttl=127 time=67.9 ms

--- 10.10.11.236 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 67.809/68.738/71.023/1.326 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.236 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-12 21:55 CST
Initiating SYN Stealth Scan at 21:55
Scanning 10.10.11.236 [65535 ports]
Discovered open port 53/tcp on 10.10.11.236
Discovered open port 135/tcp on 10.10.11.236
Discovered open port 80/tcp on 10.10.11.236
Discovered open port 139/tcp on 10.10.11.236
Discovered open port 445/tcp on 10.10.11.236
Discovered open port 49689/tcp on 10.10.11.236
Discovered open port 62362/tcp on 10.10.11.236
Discovered open port 49792/tcp on 10.10.11.236
Discovered open port 88/tcp on 10.10.11.236
Discovered open port 49721/tcp on 10.10.11.236
Discovered open port 49693/tcp on 10.10.11.236
Discovered open port 1433/tcp on 10.10.11.236
Discovered open port 49690/tcp on 10.10.11.236
SYN Stealth Scan Timing: About 45.79% done; ETC: 21:56 (0:00:37 remaining)
Discovered open port 636/tcp on 10.10.11.236
Increasing send delay for 10.10.11.236 from 0 to 5 due to 11 out of 35 dropped probes since last increase.
Discovered open port 3269/tcp on 10.10.11.236
Discovered open port 49667/tcp on 10.10.11.236
Discovered open port 3268/tcp on 10.10.11.236
Discovered open port 9389/tcp on 10.10.11.236
Discovered open port 5985/tcp on 10.10.11.236
Discovered open port 593/tcp on 10.10.11.236
Discovered open port 464/tcp on 10.10.11.236
Increasing send delay for 10.10.11.236 from 5 to 10 due to 11 out of 28 dropped probes since last increase.
Completed SYN Stealth Scan at 21:56, 66.88s elapsed (65535 total ports)
Nmap scan report for 10.10.11.236
Host is up, received user-set (0.27s latency).
Scanned at 2024-10-12 21:55:12 CST for 66s
Not shown: 65514 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
80/tcp    open  http             syn-ack ttl 127
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
1433/tcp  open  ms-sql-s         syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5985/tcp  open  wsman            syn-ack ttl 127
9389/tcp  open  adws             syn-ack ttl 127
49667/tcp open  unknown          syn-ack ttl 127
49689/tcp open  unknown          syn-ack ttl 127
49690/tcp open  unknown          syn-ack ttl 127
49693/tcp open  unknown          syn-ack ttl 127
49721/tcp open  unknown          syn-ack ttl 127
49792/tcp open  unknown          syn-ack ttl 127
62362/tcp open  unknown          syn-ack ttl 127

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 67.00 seconds
           Raw packets sent: 327642 (14.416MB) | Rcvd: 84 (3.680KB)
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

Demasiados puertos abiertos, se pueden ver algunos que son conocidos por pertenecer a un **Directorio Activo (AD)**.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 53,80,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389,49667,49689,49690,49693,49721,49792,49833 10.10.11.236 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-12 21:56 CST
Nmap scan report for manager.htb (10.10.11.236)
Host is up (0.070s latency).

PORT      STATE    SERVICE       VERSION
53/tcp    open     domain        Simple DNS Plus
80/tcp    open     http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Manager
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2024-10-13 10:57:05Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
|_ssl-date: 2024-10-13T10:58:35+00:00; +7h00m02s from scanner time.
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-10-13T10:58:36+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
1433/tcp  open     ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.11.236:1433: 
|     Target_Name: MANAGER
|     NetBIOS_Domain_Name: MANAGER
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: manager.htb
|     DNS_Computer_Name: dc01.manager.htb
|     DNS_Tree_Name: manager.htb
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2024-10-13T07:09:51
|_Not valid after:  2054-10-13T07:09:51
|_ssl-date: 2024-10-13T10:58:35+00:00; +7h00m02s from scanner time.
| ms-sql-info: 
|   10.10.11.236:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2024-10-13T10:58:35+00:00; +7h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
3269/tcp  open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: manager.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc01.manager.htb
| Not valid before: 2024-08-30T17:08:51
|_Not valid after:  2122-07-27T10:31:04
|_ssl-date: 2024-10-13T10:58:36+00:00; +7h00m01s from scanner time.
5985/tcp  open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open     mc-nmf        .NET Message Framing
49667/tcp open     msrpc         Microsoft Windows RPC
49689/tcp open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open     msrpc         Microsoft Windows RPC
49693/tcp open     msrpc         Microsoft Windows RPC
49721/tcp open     msrpc         Microsoft Windows RPC
49792/tcp open     msrpc         Microsoft Windows RPC
49833/tcp filtered unknown
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-10-13T10:57:58
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h00m01s, deviation: 0s, median: 7h00m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.85 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Es claro que nos vamos a enfrentar a un **AD**. Tenemos muchas formas por donde podemos empezar a enumerar la página.

El escaneo nos mostros el dominio del **AD** y el nombre del servidor. Registralos en tu **/etc/hosts**:
```bash
nano /etc/hosts
10.10.11.236 manager.htb dc01.manager.htb
```

Vamos a empezar por el **puerto 80**.


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
<img src="/assets/images/htb-writeup-manager/Captura1.png">
</p>

Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/htb-writeup-manager/Captura2.png">
</p>

No veo algo que nos sea útil.

Quizá **whatweb** muestre algo más:
```bash
whatweb http://10.10.11.236
http://10.10.11.236 [200 OK] Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Microsoft-IIS/10.0], IP[10.10.11.236], JQuery[3.4.1], Microsoft-IIS[10.0], Script[text/javascript], Title[Manager], X-UA-Compatible[IE=edge]
```
Tampoco veo algo útil.

Veamos si encontramos algo que nos ayude a la intrusión en la página web.

Parece que encontramos un posible usuario:

<p align="center">
<img src="/assets/images/htb-writeup-manager/Captura3.png">
</p>

Es posible que sea un usuario del AD, pero no estoy muy seguro.

Siguiendo investigando, encontramos una forma de contactar a la empresa:

<p align="center">
<img src="/assets/images/htb-writeup-manager/Captura4.png">
</p>

Esto es todo lo que hay que ver.

Probemos a aplicar **Fuzzing** para ver si hay algo más por ahí.

<h2 id="fuzz">Fuzzing</h2>

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.11.236/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.236/FUZZ
Total requests: 220545

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                         
=====================================================================

000000002:   301        1 L      10 W       150 Ch      "images"                                                                                                                                        
000000189:   301        1 L      10 W       150 Ch      "Images"                                                                                                                                        
000000939:   301        1 L      10 W       146 Ch      "js"                                                                                                                                            
000003659:   301        1 L      10 W       150 Ch      "IMAGES"                                                                                                                                        
000009123:   301        1 L      10 W       146 Ch      "JS"                                                                                                                                            
000008461:   301        1 L      10 W       147 Ch      "CSS"                                                                                                                                           
000045226:   200        506 L    1356 W     18203 Ch    "http://10.10.11.236/"                                                                                                                          
000000536:   301        1 L      10 W       147 Ch      "css"                                                                                                                                           

Total time: 0
Processed Requests: 220545
Filtered Requests: 220537
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
gobuster dir -u http://10.10.11.236/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.236/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 150] [--> http://10.10.11.236/images/]
/Images               (Status: 301) [Size: 150] [--> http://10.10.11.236/Images/]
/css                  (Status: 301) [Size: 147] [--> http://10.10.11.236/css/]
/js                   (Status: 301) [Size: 146] [--> http://10.10.11.236/js/]
/IMAGES               (Status: 301) [Size: 150] [--> http://10.10.11.236/IMAGES/]
/CSS                  (Status: 301) [Size: 147] [--> http://10.10.11.236/CSS/]
/JS                   (Status: 301) [Size: 146] [--> http://10.10.11.236/JS/]
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

No encontramos nada.

Pasemos a enumerar otros servicios.

<h2 id="DNS">Enumeración de DNS</h2>

Veamos que información podemos obtener del **DNS** con la herramienta **dig**:
```bash
dig @10.10.11.236 manager.htb

; <<>> DiG 9.20.2-1-Debian <<>> @10.10.11.236 manager.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60228
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;manager.htb.                   IN      A

;; ANSWER SECTION:
manager.htb.            600     IN      A       10.10.11.236

;; Query time: 72 msec
;; SERVER: 10.10.11.236#53(10.10.11.236) (UDP)
;; WHEN: Sat Oct 12 23:27:51 CST 2024
;; MSG SIZE  rcvd: 56
```
Confirmamos el dominio del **AD**.

Probemos si existe algún correo registrado:
```bash
dig @10.10.11.236 manager.htb mx

; <<>> DiG 9.20.2-1-Debian <<>> @10.10.11.236 manager.htb mx
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5920
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;manager.htb.                   IN      MX

;; AUTHORITY SECTION:
manager.htb.            3600    IN      SOA     dc01.manager.htb. hostmaster.manager.htb. 253 900 600 86400 3600

;; Query time: 68 msec
;; SERVER: 10.10.11.236#53(10.10.11.236) (UDP)
;; WHEN: Sat Oct 12 23:27:56 CST 2024
;; MSG SIZE  rcvd: 92
```
Nada, pero ahí aparece el nombre del servidor del **AD**.

Veamos si hay otro servidor registrado:
```bash
dig @10.10.11.236 manager.htb ns

; <<>> DiG 9.20.2-1-Debian <<>> @10.10.11.236 manager.htb ns
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49237
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;manager.htb.                   IN      NS

;; ANSWER SECTION:
manager.htb.            3600    IN      NS      dc01.manager.htb.

;; ADDITIONAL SECTION:
dc01.manager.htb.       3600    IN      A       10.10.11.236

;; Query time: 76 msec
;; SERVER: 10.10.11.236#53(10.10.11.236) (UDP)
;; WHEN: Sat Oct 12 23:28:02 CST 2024
;; MSG SIZE  rcvd: 75
```
Ninguno.

Por último, tratemos de aplicar el **ataque de transferencia de zona**:
```bash
dig @10.10.11.236 manager.htb axfr

; <<>> DiG 9.20.2-1-Debian <<>> @10.10.11.236 manager.htb axfr
; (1 server found)
;; global options: +cmd
; Transfer failed.
```
Nada tampoco. Pues solo comprobamos el dominio y el servidor del **AD**.

<h2 id="SMB">Enumeración de Servicio SMB</h2>


```bash
crackmapexec smb 10.10.11.236
SMB         10.10.11.236    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:manager.htb) (signing:True) (SMBv1:False)
```

```bash
smbclient -L //10.10.11.236// -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.236 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

```bash
smbmap -H 10.10.11.236

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.4 | Shawn Evans - ShawnDEvans@gmail.com<mailto:ShawnDEvans@gmail.com>
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 0 authenticated session(s)                                                          
[*] Closed 1 connections
```

<h2 id="kerberos">Enumeración de Servicio Kerberos</h2>

```bash
./kerbrute_linux_amd64 userenum --dc 10.10.11.236 -d manager.htb /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/12/24 - Ronnie Flathers @ropnop

2024/10/12 23:42:56 >  Using KDC(s):
2024/10/12 23:42:56 >   10.10.11.236:88

2024/10/12 23:42:57 >  [+] VALID USERNAME:       ryan@manager.htb
2024/10/12 23:43:00 >  [+] VALID USERNAME:       guest@manager.htb
2024/10/12 23:43:01 >  [+] VALID USERNAME:       cheng@manager.htb
2024/10/12 23:43:02 >  [+] VALID USERNAME:       raven@manager.htb
2024/10/12 23:43:12 >  [+] VALID USERNAME:       administrator@manager.htb
2024/10/12 23:43:28 >  [+] VALID USERNAME:       Ryan@manager.htb
2024/10/12 23:43:30 >  [+] VALID USERNAME:       Raven@manager.htb
2024/10/12 23:43:37 >  [+] VALID USERNAME:       operator@manager.htb
2024/10/12 23:44:39 >  [+] VALID USERNAME:       Guest@manager.htb
2024/10/12 23:44:40 >  [+] VALID USERNAME:       Administrator@manager.htb
2024/10/12 23:45:33 >  [+] VALID USERNAME:       Cheng@manager.htb
2024/10/12 23:48:00 >  [+] VALID USERNAME:       jinwoo@manager.htb
2024/10/12 23:48:24 >  [+] VALID USERNAME:       RYAN@manager.htb
2024/10/12 23:49:54 >  [+] VALID USERNAME:       RAVEN@manager.htb
2024/10/12 23:49:59 >  [+] VALID USERNAME:       GUEST@manager.htb
```


```bash

```

<br>

<h3 id="">Aplicando Fuerza Bruta con crackmapexec a Usuarios Encontrados</h3>

```bash

```

```bash

```

<h2 id="RPC">Enumeración de Servicio RPC</h2>

```bash
rpcclient -U "" 10.10.11.236 -N
rpcclient $>
```

```bash
rpcclient -U "" 10.10.11.236 -N
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED
rpcclient $> enumdomgroups
result was NT_STATUS_ACCESS_DENIED
rpcclient $> exit
```

```bash

```


<h2 id="LDAP">Enumeración de Servicio LDAP con ldapdomaindump</h2>

```bash

```

<h2 id="RID">Aplicando RID Cycling Attack para Enumeración de Usuarios con crackmapexec y rpcclient</h2>

```bash

```

```bash

```



<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="MSSQL">Enumeración de Servicio MSSQL</h2>


<h2 id=""></h2>


<h2 id=""></h2>




<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id=""></h2>


```batch
*Evil-WinRM* PS C:\Users\Raven\Documents> Import-Module .\adPEAS.ps1
*Evil-WinRM* PS C:\Users\Raven\Documents> Invoke-adPEAS

               _ _____  ______           _____
              | |  __ \|  ____|   /\    / ____|
      ____  __| | |__) | |__     /  \  | (___
     / _  |/ _  |  ___/|  __|   / /\ \  \___ \
    | (_| | (_| | |    | |____ / ____ \ ____) |
     \__,_|\__,_|_|    |______/_/    \_\_____/
                                            Version 0.8.24
                                                                                                                                                                                                                 
    Active Directory Enumeration
    by @61106960

    Legend
        [?] Searching for juicy information
        [!] Found a vulnerability which may can be exploited in some way
        [+] Found some interesting information for further investigation
        [*] Some kind of note
        [#] Some kind of secure configuration


[?] +++++ Searching for Juicy Active Directory Information +++++                                                                                                                                                 

[?] +++++ Checking General Domain Information +++++                                                                                                                                                              
[+] Found general Active Directory domain information for domain 'manager.htb':
Domain Name:                            manager.htb
Domain SID:                             S-1-5-21-4078382237-1492182817-2568127209
Domain Functional Level:                Windows 2016
Forest Name:                            manager.htb
Forest Children:                        No Subdomain[s] available
Domain Controller:                      dc01.manager.htb

[?] +++++ Searching for Vulnerable Certificate Templates +++++                                                                                                                                                   
adPEAS does basic enumeration only, consider using https://github.com/GhostPack/Certify or https://github.com/ly4k/Certipy

[?] +++++ Checking Template 'SubCA' +++++                                                                                                                                                                        
[!] Template 'SubCA' has Flag 'ENROLLEE_SUPPLIES_SUBJECT'
Template Name:                          SubCA
Template distinguishedname:             CN=SubCA,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=manager,DC=htb
Date of Creation:                       07/27/2023 10:31:05
EnrollmentFlag:                         0
[!] CertificateNameFlag:                ENROLLEE_SUPPLIES_SUBJECT
```


<br>
<br>
<div style="position: relative;">
  <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


* https://ppn.snovvcrash.rocks/pentest/infrastructure/ad/rid-cycling
* https://www.thehacker.recipes/ad/recon/ms-rpc
* https://github.com/61106960/adPEAS
* https://github.com/ly4k/Certipy
* https://www.reddit.com/r/hackthebox/comments/121sn0q/getting_time_skew_error_when_trying_to_run_psexec/
* https://medium.com/@danieldantebarnes/fixing-the-kerberos-sessionerror-krb-ap-err-skew-clock-skew-too-great-issue-while-kerberoasting-b60b0fe20069


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
