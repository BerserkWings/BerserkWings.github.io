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
					<li><a href="#"></a></li>
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

* **Primero**: Identificar si un archivo encontrado es un binario.
* **Segundo**: Estudiar y analizar su funcionamiento.
* **Tercero**: Confirmar vulnerabilidad con herramienta **gdb**.
* **Cuarto**: Identificar el **offset**.
* **Quinto**: Creación y ejecución de payload en el binario.

Aca te dejo unos links con buena información sobre **Buffer Overflow**:
* <a href="https://keepcoding.io/blog/que-es-un-buffer-overflow/" target="_blank">¿Qué es un buffer overflow?</a>
* <a href="https://www.fortinet.com/lat/resources/cyberglossary/buffer-overflow" target="_blank">Desbordamiento de búfer</a>
* <a href="https://www.cloudflare.com/es-es/learning/security/threats/buffer-overflow/" target="_blank">¿Qué es el desbordamiento del búfer?</a>

Para este punto, ya hemos completado el punto 1 y 2, así que sigamos con los otros 3.

<br>

<h3 id="">Confirmando Buffer Overflow con gdb</h3>


<br>

<h3 id=""></h3>



<br>

<h3 id=""></h3>




<h2 id=""></h2>


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
