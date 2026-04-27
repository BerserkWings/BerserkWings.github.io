---
layout: single
title: Phantom - TheHackerLabs
excerpt: "."
date: 2026-04-28
classes: wide
header:
  teaser: /assets/images/THL-writeup-phantom/phantom.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Medium Machine
tags:
  - Windows
  - Active Directory
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-phantom/phantom.png)

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


<h2 id="Hosts">Descubrimiento de Hosts</h2>

Hagamos un descubrimiento de hosts para encontrar a nuestro objetivo.

Lo haremos primero con **nmap**:
```bash
nmap -sn 192.168.56.0/24
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 23:07 -0600
Nmap scan report for 192.168.56.1
Host is up (0.00042s latency).
MAC Address: XX (Unknown)
Nmap scan report for 192.168.56.100
Host is up (0.00053s latency).
MAC Address: XX (Oracle VirtualBox virtual NIC)
Nmap scan report for 192.168.56.102
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 1.91 seconds
```

Vamos a probar la herramienta **arp-scan**:
```bash
arp-scan -I eth0 -g 192.168.56.0/24
Interface: eth0, type: EN10MB, MAC: 08:00:27:8d:c1:60, IPv4: 192.168.56.102
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	XX	PCS Systemtechnik GmbH
192.168.56.100	XX	PCS Systemtechnik GmbH

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.143 seconds (119.46 hosts/sec). 2 responded
```

Encontramos nuestro objetivo y es: `192.168.56.100`.

<br>

<h2 id="Ping">Traza ICMP</h2>

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 192.168.56.100
PING 192.168.56.100 (192.168.56.100) 56(84) bytes of data.
64 bytes from 192.168.56.100: icmp_seq=1 ttl=128 time=0.804 ms
64 bytes from 192.168.56.100: icmp_seq=2 ttl=128 time=1.00 ms
64 bytes from 192.168.56.100: icmp_seq=3 ttl=128 time=0.971 ms
64 bytes from 192.168.56.100: icmp_seq=4 ttl=128 time=0.874 ms

--- 192.168.56.100 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3002ms
rtt min/avg/max/mdev = 0.804/0.912/1.001/0.078 ms
```
Por el TTL sabemos que la máquina usa **Windows**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.56.100 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 23:09 -0600
Initiating ARP Ping Scan at 23:09
Scanning 192.168.56.100 [1 port]
Completed ARP Ping Scan at 23:09, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 23:09
Scanning 192.168.56.100 [65535 ports]
Discovered open port 135/tcp on 192.168.56.100
Discovered open port 139/tcp on 192.168.56.100
Discovered open port 53/tcp on 192.168.56.100
Discovered open port 445/tcp on 192.168.56.100
Discovered open port 3269/tcp on 192.168.56.100
Discovered open port 59503/tcp on 192.168.56.100
Discovered open port 59466/tcp on 192.168.56.100
Discovered open port 593/tcp on 192.168.56.100
Discovered open port 59458/tcp on 192.168.56.100
Discovered open port 88/tcp on 192.168.56.100
Discovered open port 9389/tcp on 192.168.56.100
Discovered open port 464/tcp on 192.168.56.100
Discovered open port 636/tcp on 192.168.56.100
Discovered open port 3268/tcp on 192.168.56.100
Discovered open port 59464/tcp on 192.168.56.100
Discovered open port 59487/tcp on 192.168.56.100
Discovered open port 5985/tcp on 192.168.56.100
Discovered open port 49664/tcp on 192.168.56.100
Discovered open port 59478/tcp on 192.168.56.100
Discovered open port 389/tcp on 192.168.56.100
Completed SYN Stealth Scan at 23:09, 26.32s elapsed (65535 total ports)
Nmap scan report for 192.168.56.100
Host is up, received arp-response (0.00060s latency).
Scanned at 2026-04-26 23:09:21 CST for 26s
Not shown: 65515 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 128
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
59458/tcp open  unknown          syn-ack ttl 128
59464/tcp open  unknown          syn-ack ttl 128
59466/tcp open  unknown          syn-ack ttl 128
59478/tcp open  unknown          syn-ack ttl 128
59487/tcp open  unknown          syn-ack ttl 128
59503/tcp open  unknown          syn-ack ttl 128
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.50 seconds
           Raw packets sent: 131053 (5.766MB) | Rcvd: 23 (996B)
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

Vemos muchos puertos abiertos, entre ellos el **puerto 88**, que nos indica el uso del **servicio Kerberos**, por lo que nos enfrentaremos a una máquina **Active Directory**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49664,59458,59464,59466,59478,59487,59503 192.168.56.100 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 23:10 -0600
Nmap scan report for 192.168.56.100
Host is up (0.0016s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-27 05:25:32Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: PHANTOM.THL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-27T05:26:59+00:00; +15m15s from scanner time.
| ssl-cert: Subject: commonName=DC01.PHANTOM.THL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.PHANTOM.THL
| Not valid before: 2026-02-21T23:24:38
|_Not valid after:  2027-02-21T23:24:38
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
59458/tcp open  msrpc         Microsoft Windows RPC
59464/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
59466/tcp open  msrpc         Microsoft Windows RPC
59478/tcp open  msrpc         Microsoft Windows RPC
59487/tcp open  msrpc         Microsoft Windows RPC
59503/tcp open  msrpc         Microsoft Windows RPC
MAC Address: XX (Oracle VirtualBox virtual NIC)
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 15m15s, deviation: 0s, median: 15m14s
|_nbstat: NetBIOS name: DC01, NetBIOS user: <unknown>, NetBIOS MAC: XX (Oracle VirtualBox virtual NIC)
| smb2-time: 
|   date: 2026-04-27T05:26:20
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.34 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Gracias a este escaneo, podemos ver muchas cosas interesantes:
* Podemos ver cual es el dominio del **AD** siendo `PHANTOM.THL` y `DC01.PHANTOM.THL`, siendo que podemos guardarlos en el `/etc/hosts`:
```bash
echo "192.168.56.100 PHANTOM.THL DC01.PHANTOM.THL" >> /etc/hosts
```
* 
* 
* 
* 


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Analisis" style="text-align:center;">Análisis de Vulnerabilidades</h1>
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

<p align="center">
<img src="/assets/images/THL-writeup-/Captura#.png">
</p>


<a href="link" target="_blank">titulo_del_link</a>


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
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
