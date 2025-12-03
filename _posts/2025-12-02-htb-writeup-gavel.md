---
layout: single
title: Gavel - Hack The Box
excerpt: "."
date: 2025-12-02
classes: wide
header:
  teaser: /assets/images/htb-writeup-gavel/gavel.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Medium Machine
tags:
  - Linux
  - 
  - 
  - OSCP Style
---
![](/assets/images/htb-writeup-gavel/gavel.png)

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
ping -c 4 10.129.45.152
PING 10.129.45.152 (10.129.45.152) 56(84) bytes of data.
64 bytes from 10.129.45.152: icmp_seq=1 ttl=63 time=68.4 ms
64 bytes from 10.129.45.152: icmp_seq=2 ttl=63 time=71.5 ms
64 bytes from 10.129.45.152: icmp_seq=3 ttl=63 time=69.7 ms
64 bytes from 10.129.45.152: icmp_seq=4 ttl=63 time=71.5 ms

--- 10.129.45.152 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3012ms
rtt min/avg/max/mdev = 68.373/70.278/71.525/1.324 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.45.152 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-02 12:40 CST
Initiating SYN Stealth Scan at 12:40
Scanning 10.129.45.152 [65535 ports]
Discovered open port 22/tcp on 10.129.45.152
Discovered open port 80/tcp on 10.129.45.152
Completed SYN Stealth Scan at 12:41, 19.64s elapsed (65535 total ports)
Nmap scan report for 10.129.45.152
Host is up, received user-set (0.078s latency).
Scanned at 2025-12-02 12:40:58 CST for 20s
Not shown: 55237 closed tcp ports (reset), 10296 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 19.74 seconds
           Raw packets sent: 97302 (4.281MB) | Rcvd: 60390 (2.416MB)
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

Solo hay dos puertos abiertos. Supongo que la intrusión será por el **puerto 80**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80 10.129.45.152 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-02 12:42 CST
Nmap scan report for 10.129.45.152
Host is up (0.070s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1f:de:9d:84:bf:a1:64:be:1f:36:4f:ac:3c:52:15:92 (ECDSA)
|_  256 70:a5:1a:53:df:d1:d0:73:3e:9d:90:ad:c1:aa:b4:19 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://gavel.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: gavel.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.89 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

El escaneo de la página web del **puerto 80**, nos indica que hay una redirección a un dominio al momento de visitarla.

Registremos ese dominio en el `/etc/hosts`:
```bash
echo "10.129.45.152 gavel.htb" >> /etc/hosts
```
Visitemos la página para saber a que nos enfrentamos.


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
<img src="/assets/images/htb-writeup-gavel/Captura1.png">
</p>

Parece ser una página un poco extraña sobre compra de articulos curiosos.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/htb-writeup-gavel/Captura2.png">
</p>

No hay mucho que destacar.

Del lado izquierdo, podemos ver que nos podemos loguear y crear un usuario:

<p align="center">
<img src="/assets/images/htb-writeup-gavel/Captura3.png">
</p>

Entra a la opción de crear un usuario y crea uno para entrar a la página web:

<p align="center">
<img src="/assets/images/htb-writeup-gavel/Captura4.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-gavel/Captura5.png">
</p>

Estamos dentro y del lado izquierdo tenemos dos páginas más.

Si entramos a la opción **Bidding**, encontraremos varios articulos que parecen estar dentro de una subasta:

<p align="center">
<img src="/assets/images/htb-writeup-gavel/Captura6.png">
</p>

Debemos ofrecer la cantidad que viene indicada en el mensaje de cada articulo para poder comprarlo:

<p align="center">
<img src="/assets/images/htb-writeup-gavel/Captura7.png">
</p>

Una vez que pasa el tiempo para subastar, el articulo pasa a nuestro inventario, que podemos ver en la opción **Inventory**:

<p align="center">
<img src="/assets/images/htb-writeup-gavel/Captura8.png">
</p>

De momento, es todo lo que podemos hacer con nuestro usuario.

Apliquemos **Fuzzing** para saber si hay algún directorio o archivo oculto.

<br>

<h2 id="fuzz">Fuzzing</h2>

Primero probemos con la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt:FUZZ -u http://gavel.htb/FUZZ -t 300 -e .php,.txt,.html -mc 200,302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://gavel.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200,302
________________________________________________

.git/logs/              [Status: 200, Size: 1128, Words: 77, Lines: 18, Duration: 155ms]
.git/config             [Status: 200, Size: 136, Words: 13, Lines: 9, Duration: 158ms]
.git/HEAD               [Status: 200, Size: 23, Words: 2, Lines: 2, Duration: 167ms]
admin.php               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 134ms]
admin.php               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 150ms]
.git/index              [Status: 200, Size: 224718, Words: 313, Lines: 355, Duration: 5401ms]
index.php               [Status: 200, Size: 14108, Words: 4631, Lines: 223, Duration: 122ms]
index.php               [Status: 200, Size: 14071, Words: 4626, Lines: 223, Duration: 122ms]
inventory.php           [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 93ms]
login.php               [Status: 200, Size: 4281, Words: 1817, Lines: 79, Duration: 117ms]
logout.php              [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 200ms]
register.php            [Status: 200, Size: 4485, Words: 1571, Lines: 85, Duration: 150ms]
:: Progress: [18984/18984] :: Job [1/1] :: 219 req/sec :: Duration: [0:00:33] :: Errors: 24 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-e*       | Para indicar una busqueda de archivos específicos. |
| *-mc*      | Para indicar que muestre solo resultados con códigos de estados específicos. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://gavel.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -t 300 -x php,txt,html --ne
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://gavel.htb
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.git/config          (Status: 200) [Size: 136]
/admin.php            (Status: 302) [Size: 0] [--> index.php]
/assets               (Status: 301) [Size: 307] [--> http://gavel.htb/assets/]
/includes             (Status: 301) [Size: 309] [--> http://gavel.htb/includes/]
/index.php            (Status: 200) [Size: 14014]
/index.php            (Status: 200) [Size: 14054]
/inventory.php        (Status: 302) [Size: 0] [--> index.php]
/login.php            (Status: 200) [Size: 4281]
/logout.php           (Status: 302) [Size: 0] [--> index.php]
/register.php         (Status: 200) [Size: 4485]
/rules                (Status: 301) [Size: 306] [--> http://gavel.htb/rules/]
/server-status        (Status: 403) [Size: 274]
Progress: 18984 / 18984 (100.00%)
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
| *--ne*     | Para que no muestre errores. |

<br>

Encontramos algunos archivos como **admin.php**, pero lo más interesante es ver que hay un repositorio de **Git** expuesto, así que vamos a descargarlo.

<br>

<h2 id="DumpGit">Dumpeando Repositorio de Git Expuesto y Analizando su Contenido</h2>

Para dumpear el repositoio de **Git**, utilizaremos la siguiente herramienta:
* <a href="https://github.com/arthaud/git-dumper" target="_blank">Repositorio de arthaud: git-dumper</a>
 
Una vez que lo descargues, instala los requerimientos y ya podras usar la herramienta:
```bash
pip install -r requirements.txt

python3 git_dumper.py
usage: git-dumper [options] URL DIR
git_dumper.py: error: the following arguments are required: URL, DIR
```

Crea un directorio donde se guarde el repo e indicale a **git-dumper** que lo almacene ahí:
```bash
mkdir gavel-git

python3 git-dumper/git_dumper.py http://gavel.htb/.git/ gavel-git
[-] Testing http://gavel.htb/.git/HEAD [200]
[-] Testing http://gavel.htb/.git/ [200]
...
[-] Running git checkout .
Actualizadas 1849 rutas desde el índice
```
Excelente, tenemos el repositorio completo.

Analicemos su contenido.

<br>

<h3 id="admin">Analizando Script admin.php e Identificando Posibles Vulnerabilidades</h3>

```bash

```

```bash

```

```bash

```

```bash

```

<br>

<h3 id="inventory">Analizando Script inventory.php e Identificando Posibles Vulnerabilidades</h3>

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
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="SQLi">Aplicando Inyección SQL Union-Based con Filtros y Crackeando Hash Bcrypt</h2>



```bash

```

```bash

```

```bash

```

```bash
john -w:/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
midnight1        (?)     
1g 0:00:00:27 DONE (2025-12-02 19:07) 0.03698g/s 113.8p/s 113.8c/s 113.8C/s iamcool..milena
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

<br>

<h2 id="RCE"></h2>

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
