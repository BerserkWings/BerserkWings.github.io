---
layout: single
title: GoodGames - Hack The Box
excerpt: "."
date: 2024-10-14
classes: wide
header:
  teaser: /assets/images/htb-writeup-goodgames/GoodGames.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Easy Machine
tags:
  - Linux
  - 
  - 
  - 
---
![](/assets/images/htb-writeup-goodgames/GoodGames.png)

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
ping -c 4 10.10.11.130
PING 10.10.11.130 (10.10.11.130) 56(84) bytes of data.
64 bytes from 10.10.11.130: icmp_seq=1 ttl=63 time=67.8 ms
64 bytes from 10.10.11.130: icmp_seq=2 ttl=63 time=67.8 ms
64 bytes from 10.10.11.130: icmp_seq=3 ttl=63 time=68.3 ms
64 bytes from 10.10.11.130: icmp_seq=4 ttl=63 time=68.1 ms

--- 10.10.11.130 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 67.771/68.001/68.271/0.213 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.11.130 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-14 12:42 CST
Initiating SYN Stealth Scan at 12:42
Scanning 10.10.11.130 [65535 ports]
Discovered open port 80/tcp on 10.10.11.130
Completed SYN Stealth Scan at 12:42, 27.57s elapsed (65535 total ports)
Nmap scan report for 10.10.11.130
Host is up, received user-set (0.083s latency).
Scanned at 2024-10-14 12:42:20 CST for 28s
Not shown: 49224 closed tcp ports (reset), 16310 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 27.69 seconds
           Raw packets sent: 137121 (6.033MB) | Rcvd: 52326 (2.093MB)
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

Que extraño, solo nos dio un puerto abierto.

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 80 10.10.11.130 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-14 12:43 CST
Nmap scan report for 10.10.11.130
Host is up (0.069s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
|_http-title: GoodGames | Community and Store
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
Service Info: Host: goodgames.htb

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.36 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Parece que nos enfrentamos otra vez a **Werkzeug y Flask**. Vayamos a ver esa página web.


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
<img src="/assets/images/htb-writeup-goodgames/Captura1.png">
</p>

Se ve chula la página.

Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura2.png">
</p>

Parece que estan ocupando muchas tecnologias.

Probemos si sale algo extra con **whatweb**:
```bash
whatweb http://10.10.11.130
http://10.10.11.130 [200 OK] Bootstrap, Country[RESERVED][ZZ], Frame, HTML5, HTTPServer[Werkzeug/2.0.2 Python/3.9.2], IP[10.10.11.130], JQuery, Meta-Author[_nK], PasswordField[password], Python[3.9.2], Script, Title[GoodGames | Community and Store], Werkzeug[2.0.2], X-UA-Compatible[IE=edge]
```
Me da curiosidad eso del **PasswordField**. 

Por lo mientras, vamos a ver con que nos encontramos en la página web.

Si le damos al botón **Blog**, nos mostrara varias publicaciones y veremos algunos usuarios:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura3.png">
</p>

También podemos visitar los productos, pero solo sale lo siguiente:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura4.png">
</p>

Investigando por ahí, podemos ver un blog que si podemos visitar:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura5.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura6.png">
</p>

Resulta que es una publicación del administrador, entonces, aquí esta registrado un administrador. Lo tendremos en cuenta para más adelante.

Existe un login en el que nos podemos autenticar y registrar:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura7.png">
</p>

Vamos a intentar registrarnos con un usuario random:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura8.png">
</p>

Obtuvimos un registro exitoso.

Ahora nos autenticamos:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura9.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura10.png">
</p>

Bien, pudimos entrar.

Veamos si hay algo por ahí que solo un usuario puede ver:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura11.png">
</p>

Nada, solamente podemos ver la información de nuestro usuario.

Veamos si no hay algo que se nos este escapando, aplicando **Fuzzing**.

<h2 id="fuzz">Fuzzing</h2>

```bash
wfuzz -c -L --hc=404 -t 200 --hh 9265 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.11.130/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.11.130/FUZZ
Total requests: 220545

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                         
=====================================================================

000000072:   200        266 L    545 W      9267 Ch     "profile"                                                                                                                                       
000000039:   200        266 L    553 W      9294 Ch     "login"                                                                                                                                         
000000018:   200        908 L    2572 W     44206 Ch    "blog"                                                                                                                                          
000000203:   200        727 L    2070 W     33387 Ch    "signup"                                                                                                                                        
000001211:   200        1734 L   5548 W     85093 Ch    "logout"                                                                                                                                        
000012936:   200        729 L    2069 W     32744 Ch    "forgot-password"                                                                                                                               
000044928:   200        286 L    620 W      10524 Ch    "coming-soon"                                                                                                                                   
000045226:   200        1734 L   5548 W     85093 Ch    "http://10.10.11.130/"                                                                                                                          

Total time: 0
Processed Requests: 88573
Filtered Requests: 88565
Requests/sec.: 0
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *-L*	     | Para que siga las redirecciones del HTTP que encuentre. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *--hh*     | Para no mostrar resultados que tenga N cantidad de caracteres. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://10.10.11.130/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 30 --exclude-length=9265 -r
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.130/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          9265
[+] User Agent:              gobuster/3.6
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/blog                 (Status: 200) [Size: 44212]
/login                (Status: 200) [Size: 9294]
/profile              (Status: 200) [Size: 9267]
/signup               (Status: 200) [Size: 33387]
/logout               (Status: 200) [Size: 85107]
/forgot-password      (Status: 200) [Size: 32744]
/coming-soon          (Status: 200) [Size: 10524]
/server-status        (Status: 403) [Size: 277]
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
| *--exclude-length=* | Para excluir resultados con N cantidad de caracteres. |
| *-r*       | Para que siga las redirecciones que encuentre. |

<br>

Pues no encontramos algo útil que nos ayude.

<h2 id="SQL1">Probando Inyección SQL en Login de Página Web con BurpSuite</h2>

Algo que podemos hacer, es aplicar una inyección SQL en el login:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura12.png">
</p>

Pero obtenemos este problema.

Quizá se este aplicando una expresión que evite las **SQLi**:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura13.png">
</p>

Parece que no.

Aplicaremos la inyección con **BurpSuite**.

Captura la petición del login con **BurpSuite**:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura22.png">
</p>

Enviala al **Repeater** y lanza la petición:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura14.png">
</p>

Observa como obtenemos un error.

Intentemos aplicar una **SQLi** común en el campo del **email**:
```sql
' or 1=1-- -
```

Envía la petición:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura15.png">
</p>

Excelente, parece que la **SQLi** funciono, ya que se obtuvo una cookie de sesión y aparece el mensaje de un login exitoso.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="SQL2">Aplicando Inyección SQL a Login y Ganando Acceso a la Página como Administrador</h2>

Ya vimos que se puede aplicar una inyección SQL al login.

Vamos a capturarlo de nuevo y modificaremos el campo del email, para meter nuestra **SQLi**:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura16.png">
</p>

Dale a **Forward** para que enviar la petición con nuestra **SQLi**.

Observa el resultado que obtenemos:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura17.png">
</p>

Ganamos acceso como administrador, que raro.

El perfil nos dice la información del administrador:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura18.png">
</p>

Parece que la extensión del correo es **goodgames.htb**.

Curiosamente, apareción un botón de configuración:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura19.png">
</p>

Entremos para ver de que se trata:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura20.png">
</p>

Parece que no nos esta resolviendo la página.

Esto nos indica que se esta aplicando **virtual hosting**.

Vamos a agregar esa ruta que no nos resuelve y **goodgames.htb** que es la extensión de correo que vimos al **/etc/hosts**:
```bash
nano /etc/hosts
10.10.11.130 internal-administration.goodgames.htb goodgames.htb
```

Listo, vuelve a cargar la página y veamos si ahora si resuelve:

<p align="center">
<img src="/assets/images/htb-writeup-goodgames/Captura21.png">
</p>

Muy bien, parece que llegamos al login de **Flask Volt**.




<h2 id="SQL3">Aplicando Inyecciones SQL para Dumpear Base de Datos Manualmente con BurpSuite</h2>


<h2 id="SQL4">Utilizando SQLMAP para Dumpear Base de Datos</h2>


<h2 id="SQL5">Aplicando Inyecciones SQL con curl</h2>


<h2 id="MD5">Crackeando Hash Obtenido de la Base de Datos con JohnTheRipper</h2>

```bash
echo -n "2b22337f218b2d82dfc3b6f77e7cb8ec" | wc -c
32
```

```bash
hash-identifier
/usr/share/hash-identifier/hash-id.py:13: SyntaxWarning: invalid escape sequence '\ '
  logo='''   #########################################################################
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: 2b22337f218b2d82dfc3b6f77e7cb8ec

Possible Hashs:
[+] MD5
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

Least Possible Hashs:
[+] RAdmin v2.x
[+] NTLM
[+] MD4
[+] MD2
[+] MD5(HMAC)
[+] MD4(HMAC)
[+] MD2(HMAC)
[+] MD5(HMAC(Wordpress))
[+] Haval-128
[+] Haval-128(HMAC)
[+] RipeMD-128
[+] RipeMD-128(HMAC)
[+] SNEFRU-128
[+] SNEFRU-128(HMAC)
[+] Tiger-128
[+] Tiger-128(HMAC)
[+] md5($pass.$salt)
[+] md5($salt.$pass)
```

```bash
john -w:/usr/share/wordlists/rockyou.txt hash
Warning: detected hash type "LM", but the string is also recognized as "dynamic=md5($p)"
Use the "--format=dynamic=md5($p)" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "HAVAL-128-4"
Use the "--format=HAVAL-128-4" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "MD2"
Use the "--format=MD2" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "mdc2"
Use the "--format=mdc2" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "mscash"
Use the "--format=mscash" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "mscash2"
Use the "--format=mscash2" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "NT"
Use the "--format=NT" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "Raw-MD4"
Use the "--format=Raw-MD4" option to force loading these as that type instead
Warning: detected hash type "LM", but the string is also recognized as "Raw-MD5"
Use the "--format=Raw-MD5" option to force loading these as that type instead
```

```bash
john -w:/usr/share/wordlists/rockyou.txt hash --format=Raw-MD5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 SSE2 4x3])
Warning: no OpenMP support for this hash type, consider --fork=5
Press 'q' or Ctrl-C to abort, almost any other key for status
superadministrator (?)     
1g 0:00:00:00 DONE (2024-10-14 15:09) 4.166g/s 14484Kp/s 14484Kc/s 14484KC/s superarely1993..super_haven
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```


<h2 id=""></h2>


<h2 id=""></h2>



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
