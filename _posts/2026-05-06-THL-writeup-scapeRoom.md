---
layout: single
title: ScapeRoom - TheHackerLabs
excerpt: "."
date: 2026-05-07
classes: wide
header:
  teaser: /assets/images/THL-writeup-scapeRoom/ScapeRoom.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Linux
  - SSH
  - MySQL
  - Web Enumeration
  - Fuzzing
  - Base64 Decoding
  - Analysing Metadata in Image
  - MySQL Enumeration
  - MySQL Decrypting Password
  - 
  - OSCP Style
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
				<li><a href="#HTTP">Analizando Servicio HTTP</a></li>
				<li><a href="#fuzz">Fuzzing</a></li>
				<li><a href="#DNS">Descubriendo Dominio y Subdominio Ocultos en Cabeceras HTTP con Nikto</a></li>
				<ul>
					<li><a href="#metadatos">Analizando Metadatos de una Imagen y Ganando Acceso al Servicio MySQL</a></li>
				</ul>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#MySQL">Enumeración del Servicio MySQL y Desencriptando Contraseña Almacenada en Base de Datos</a></li>
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
nmap -sn 192.168.110.0/24
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-06 12:52 -0600
...
Nmap scan report for thlpwn.thl (192.168.110.10)
Host is up (0.00090s latency).
MAC Address: XX (Oracle VirtualBox virtual NIC)
...
Nmap done: 256 IP addresses (9 hosts up) scanned in 12.61 seconds
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.110.0/24
Interface: eth0, type: EN10MB, MAC: XX, IPv4: Tu_IP
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
...
...
192.168.100.22	XX  PCS Systemtechnik GmbH

9 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.142 seconds (119.51 hosts/sec). 9 responded
```

Encontramos nuestro objetivo y es: `192.168.110.10`.

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.110.10
PING 192.168.110.10 (192.168.110.10) 56(84) bytes of data.
64 bytes from 192.168.110.10: icmp_seq=1 ttl=64 time=1.53 ms
64 bytes from 192.168.110.10: icmp_seq=2 ttl=64 time=0.866 ms
64 bytes from 192.168.110.10: icmp_seq=3 ttl=64 time=1.72 ms
64 bytes from 192.168.110.10: icmp_seq=4 ttl=64 time=0.562 ms

--- 192.168.110.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3034ms
rtt min/avg/max/mdev = 0.562/1.169/1.719/0.472 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.110.10 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-06 12:53 -0600
Initiating ARP Ping Scan at 12:53
Scanning 192.168.110.10 [1 port]
Completed ARP Ping Scan at 12:53, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:53
Scanning 192.168.110.10 [65535 ports]
SYN Stealth Scan Timing: About 50.00% done; ETC: 12:55 (0:00:37 remaining)
Discovered open port 80/tcp on 192.168.110.10
Discovered open port 22/tcp on 192.168.110.10
Discovered open port 3306/tcp on 192.168.110.10
Completed SYN Stealth Scan at 12:55, 75.76s elapsed (65535 total ports)
Nmap scan report for 192.168.110.10
Host is up, received arp-response (0.00087s latency).
Scanned at 2026-05-06 12:53:50 CST for 75s
Not shown: 53054 filtered tcp ports (no-response), 12478 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
80/tcp   open  http    syn-ack ttl 64
3306/tcp open  mysql   syn-ack ttl 64
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 75.96 seconds
           Raw packets sent: 124832 (5.493MB) | Rcvd: 12484 (499.368KB)
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

Tenemo 3 puertos abiertos, pero es curioso ver el **puerto 3306**, que es del **servicio MySQL**, que este expuesto.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80,3306 192.168.110.10 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-06 12:55 -0600
Nmap scan report for thlpwn.thl (192.168.110.10)
Host is up (0.00100s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:72:6f:9a:ba:7f:73:1d:bc:30:36:ee:9d:62:e8:88 (RSA)
|   256 cd:0b:5b:55:41:13:bc:1c:d1:27:81:77:ff:6d:33:15 (ECDSA)
|_  256 63:29:44:28:7d:5a:db:39:29:47:2f:a5:1d:17:fc:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Lecturas de Sensores
3306/tcp open  mysql   MySQL 8.0.40-0ubuntu0.20.04.1
| ssl-cert: Subject: commonName=MySQL_Server_8.0.39_Auto_Generated_Server_Certificate
| Not valid before: 2024-11-01T16:51:24
|_Not valid after:  2034-10-30T16:51:24
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.40-0ubuntu0.20.04.1
|   Thread ID: 26
|   Capabilities flags: 65535
|   Some Capabilities: InteractiveClient, LongPassword, FoundRows, ConnectWithDatabase, SupportsTransactions, IgnoreSpaceBeforeParenthesis, Speaks41ProtocolNew, SupportsCompression, SupportsLoadDataLocal, ODBCClient, LongColumnFlag, Support41Auth, IgnoreSigpipes, DontAllowDatabaseTableColumn, SwitchToSSLAfterHandshake, Speaks41ProtocolOld, SupportsAuthPlugins, SupportsMultipleResults, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: B\x0C\x12lF+6	b%!fzrR%X\x0Bi,
|_  Auth Plugin Name: caching_sha2_password
|_ssl-date: TLS randomness does not represent time
MAC Address: 08:00:27:7C:59:CE (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.93 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Gracias al escaneo de servicios, vemos que la página web activa en el **puerto 80** no utiliza un dominio y vemos información sobre la versión del **servicio MySQL**, etc.

Entonces, empeemos por la página web.


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
<img src="/assets/images/THL-writeup-scapeRoom/Captura1.png">
</p>

Es una página que monitorea varios sensores.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-scapeRoom/Captura2.png">
</p>

No hay mucho que nos ayude.

Revisando el código fuente, encontraremos una cadena en **base64**:

<p align="center">
<img src="/assets/images/THL-writeup-scapeRoom/Captura3.png">
</p>

Al decodificarla con el comando **base64**, vemos solo un mensaje:
```bash
echo -n "QSB2ZWNlcywgbG8gbcOhcyBvYnZpbyBlcyBsbyBxdWUgc2UgcGFzYSBwb3IgYWx0by4gVHUgw7puaWNhIHNhbGlkYSBlcyB2ZXIgbG8gcXVlIG90cm9zIG5vIHZlbi4gRWwgdGllbXBvIGNvcnJlIH
kgY2FkYSBwaXN0YSBlcyB1bmEgcGllemEgY2xhdmUgZGVsIHJvbXBlY2FiZXphcy4K" | base64 -d
A veces, lo más obvio es lo que se pasa por alto. Tu única salida es ver lo que otros no ven. El tiempo corre y cada pista es una pieza clave del rompecabezas.
```
No encontraremos algo más, así que podemos aplicar **Fuzzing** para buscar directorios y/o archivos ocultos.

<br>

<h2 id="fuzz">Fuzzing</h2>

Primero probemos con la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt:FUZZ -u http://192.168.110.10/FUZZ -t 300 -mc 200,301 -e .php,.txt,.html

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.110.10/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200,301
________________________________________________

administracion          [Status: 301, Size: 325, Words: 20, Lines: 10, Duration: 7ms]
css                     [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 2ms]
index.php               [Status: 200, Size: 4268, Words: 1608, Lines: 133, Duration: 6ms]
index.php               [Status: 200, Size: 4268, Words: 1608, Lines: 133, Duration: 26ms]
.                       [Status: 200, Size: 4268, Words: 1608, Lines: 133, Duration: 287ms]
:: Progress: [513480/513480] :: Job [1/1] :: 236 req/sec :: Duration: [0:07:58] :: Errors: 641 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*	     | Para indicar la cantidad de hilos a usar. |
| *-mc*	     | Para aplicar un filtro que solo muestre resultados con un código de estado específico. |
| *-e*	     | Para específicar la extensión de un archivo a buscar. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://192.168.110.10 -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt -t 300 -x php,txt,html -s 200,301 -b "" --ne
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:            http://192.168.110.10
[+] Method:         GET
[+] Threads:        300
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt
[+] Status codes:   200,301
[+] User Agent:     gobuster/3.8.2
[+] Extensions:     html,php,txt
[+] Timeout:        10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
administracion       (Status: 301) [Size: 325] [--> http://192.168.110.10/administracion/]
css                  (Status: 301) [Size: 314] [--> http://192.168.110.10/css/]
index.php            (Status: 200) [Size: 4273]
index.php            (Status: 200) [Size: 4273]
.                    (Status: 200) [Size: 4273]
Progress: 513480 / 513480 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-s*	     | Para aplicar un filtro que muestre solo los códigos de estado específicos. |
| *-b*	     | Para que funcione el filtro anterior. |
| *--ne*     | Para que no muestre errores. |
| *-x*	     | Para específicar la extensión de un archivo a buscar. |

<br>

Encontramos un directorio llamado **administracion**, pero al entrar en este, no tendremos permisos suficientes:

<p align="center">
<img src="/assets/images/THL-writeup-scapeRoom/Captura4.png">
</p>

Sin embargo, puede que podamos ver el contenido de este directorio, por lo que también le aplicaremos **Fuzzing**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt:FUZZ -u http://192.168.110.10/administracion/FUZZ -t 300 -mc 200,301 -e .php,.txt,.html

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.110.10/administracion/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200,301
________________________________________________

login.php               [Status: 200, Size: 2714, Words: 1004, Lines: 107, Duration: 49ms]
:: Progress: [513480/513480] :: Job [1/1] :: 232 req/sec :: Duration: [0:08:35] :: Errors: 1075 ::
```

Muy bien, encontramos una página llamada **login.php**:

<p align="center">
<img src="/assets/images/THL-writeup-scapeRoom/Captura5.png">
</p>

Por ahora no tenemos credenciales, así que tenemos que buscar una forma de encontrarlas.

<br>

<h2 id="DNS">Descubriendo Dominio y Subdominio Ocultos en Cabeceras HTTP con Nikto</h2>

Al no encontrar alguna pista más, utilizamos la herramienta **nikto** para comprobar si había algo que se nos escapaba.

Observa el resultado:
```bash
nikto -host 192.168.110.10
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.110.10
+ Target Hostname:    192.168.110.10
+ Target Port:        80
+ Start Time:         2026-05-06 13:17:04 (GMT-6)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: Uncommon header 'x-hint' found, with contents: Subdominios.... info.
+ /: Uncommon header 'x-contact' found, with contents: info@sensores.thl.
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.41 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /css/: Directory indexing found.
+ /css/: This might be interesting.
+ 8102 requests: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2026-05-06 13:17:51 (GMT-6) (47 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
Podemos ver que hay 2 cabeceras que nos indican un dominio y un subdominio.

Podemos consultarlas con **curl**, indicando que nos muestre las cabeceras:
```bash
curl -i http://192.168.110.10 | head -n 8
HTTP/1.1 200 OK
Date: Thu, 07 May 2026 00:22:05 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
X-Hint: Subdominios.... info
X-Contact: info@sensores.thl
Content-Length: 4269
Content-Type: text/html; charset=UTF-8
```

Guardemos ese dominio y subdominio en el `/etc/hosts`:
```bash
echo "192.168.110.10 sensores.thl info.sensores.thl" >> /etc/hosts
```
Y entremos en estos para ver que encontramos.

Primero el dominio **sensores.thl**:

<p align="center">
<img src="/assets/images/THL-writeup-scapeRoom/Captura6.png">
</p>

Si te das cuenta, no cambia nada con la página principal que habíamos visto antes.

Ahora el subdominio **info.sensores.thl**:

<p align="center">
<img src="/assets/images/THL-writeup-scapeRoom/Captura7.png">
</p>

Ahora si que cambia, pues podemos ver más información agregada, pero no hay algo que nos ayude.

<br>

<h3 id="metadatos">Analizando Metadatos de una Imagen y Ganando Acceso al Servicio MySQL</h3>

Algo que podemos hacer es analizar los metadatos de la imagen que encontramos:
```bash
wget http://info.sensores.thl/images/sensor_overview.png
```

Para ver los metadatos, usaremos la herramienta **exiftool**:
```bash
exiftool sensor_overview.png
ExifTool Version Number         : 13.44
File Name                       : sensor_overview.png
Directory                       : .
File Size                       : 1931 kB
File Modification Date/Time     : 2024:11:03 11:02:04-06:00
File Access Date/Time           : 2026:05:06 13:20:25-06:00
File Inode Change Date/Time     : 2026:05:06 13:20:25-06:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 1147
Image Height                    : 1147
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Comment                         : YWN1dGU6SVM0eUJ2Znd4cFhVWnNCeGhDWHI1bXV2M2RYZFFnIQo=
Image Size                      : 1147x1147
Megapixels                      : 1.3
```
Observa que tiene un comentario agregado como una cadena en **base64**.

Vamos a decodificarla:
```bash
echo -n "YWN1dGU6SVM0eUJ2Znd4cFhVWnNCeGhDWHI1bXV2M2RYZFFnIQo=" | base64 -d
acute:IS4yBvfwxpXUZsBxhCXr5muv3dXdQg!
```
Parece que encontramos credenciales de acceso.

Se intento utilizar en el login que encontramos en la página web, pero no funcionara. Sin embargo, al probarlo para entrar al **servicio MySQL** nos dará acceso:
```bash
mysql -h 192.168.110.10 -u 'acute' -p --skip-ssl
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 24
Server version: 8.0.40-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="MySQL">Enumeración del Servicio MySQL y Desencriptando Contraseña Almacenada en Base de Datos</h2>

```bash
MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| SensorData         |
| information_schema |
| performance_schema |
+--------------------+
3 rows in set (0.003 sec)
```

```bash
MySQL [(none)]> use SensorData;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

```bash
MySQL [SensorData]> show tables;
+------------------------+
| Tables_in_SensorData   |
+------------------------+
| login                  |
| sensor_readings        |
| sensor_readings_backup |
+------------------------+
3 rows in set (0.002 sec)
```

```bash
MySQL [SensorData]> select * from login;
+----+---------------+------------------------------------------------------------------+
| id | usuario       | password                                                         |
+----+---------------+------------------------------------------------------------------+
|  1 | administrador | BD9D09C524BC8A234E0B75D4FA60AF8A71D124915A3D44BC17B761186D0EB00E |
+----+---------------+------------------------------------------------------------------+
1 row in set (0.002 sec)
```

```bash
MySQL [SensorData]> show events from SensorData;
+------------+--------------------+----------------+-----------+-----------+------------+----------------+----------------+---------------------+------+---------+------------+----------------------+----------------------+--------------------+
| Db         | Name               | Definer        | Time zone | Type      | Execute at | Interval value | Interval field | Starts              | Ends | Status  | Originator | character_set_client | collation_connection | Database Collation |
+------------+--------------------+----------------+-----------+-----------+------------+----------------+----------------+---------------------+------+---------+------------+----------------------+----------------------+--------------------+
| SensorData | insert_login_data  | acute@%        | SYSTEM    | RECURRING | NULL       | 1              | MINUTE         | 2024-11-02 22:46:13 | NULL | ENABLED |          1 | utf8mb3              | utf8mb3_general_ci   | utf8mb4_0900_ai_ci |
| SensorData | sensor_data_update | root@localhost | SYSTEM    | RECURRING | NULL       | 1              | MINUTE         | 2024-11-01 18:02:42 | NULL | ENABLED |          1 | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
+------------+--------------------+----------------+-----------+-----------+------------+----------------+----------------+---------------------+------+---------+------------+----------------------+----------------------+--------------------+
```

```bash
MySQL [SensorData]> SELECT * FROM information_schema.EVENTS;
...
...    
    IF (SELECT COUNT(*) FROM login) = 0 THEN
        SET @password_plain = SUBSTRING(MD5(RAND()), 1, 16);
        INSERT INTO login (usuario, password)
        VALUES ('administrador', HEX(AES_ENCRYPT(@password_plain, 'encryption_key')));
    END IF;
END
...                                                                                                                                                                                                
```

```bash
MySQL [SensorData]> SELECT AES_DECRYPT(UNHEX(password), 'encryption_key') FROM login;
+------------------------------------------------+
| AES_DECRYPT(UNHEX(password), 'encryption_key') |
+------------------------------------------------+
| e381025ee5804e9a                               |
+------------------------------------------------+
1 row in set (0.002 sec)
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
