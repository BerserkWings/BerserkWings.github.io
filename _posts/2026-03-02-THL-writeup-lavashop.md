---
layout: single
title: Lavashop - TheHackerLabs
excerpt: "."
date: 2026-03-02
classes: wide
header:
  teaser: /assets/images/THL-writeup-lavashop/lavashop.png
  teaser_home_page: true
  icon: /assets/images/thehackerlabs.jpeg
categories:
  - TheHackerLabs
  - Easy Machine
tags:
  - Linux
  - 
  - 
  - OSCP Style
---
![](/assets/images/THL-writeup-lavashop/lavashop.png)

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
ping -c 4 192.168.100.210
PING 192.168.68.60 (192.168.100.210) 56(84) bytes of data.
64 bytes from 192.168.100.210: icmp_seq=1 ttl=64 time=1.87 ms
64 bytes from 192.168.100.210: icmp_seq=2 ttl=64 time=0.988 ms
64 bytes from 192.168.100.210: icmp_seq=3 ttl=64 time=1.08 ms
64 bytes from 192.168.100.210: icmp_seq=4 ttl=64 time=0.928 ms

--- 192.168.100.210 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3141ms
rtt min/avg/max/mdev = 0.928/1.217/1.874/0.382 ms
```
Por el TTL sabemos que la máquina usa **Linux**, hagamos los escaneos de puertos y servicios.

<br>

<h2 id="Puertos">Escaneo de Puertos</h2>

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.100.210 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-02 10:06 -0600
Initiating ARP Ping Scan at 10:06
Scanning 192.168.100.210 [1 port]
Completed ARP Ping Scan at 10:06, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:06
Scanning 192.168.100.210 [65535 ports]
Discovered open port 22/tcp on 192.168.100.210
Discovered open port 80/tcp on 192.168.100.210
Discovered open port 1337/tcp on 192.168.100.210
Completed SYN Stealth Scan at 10:06, 7.23s elapsed (65535 total ports)
Nmap scan report for 192.168.100.210
Host is up, received arp-response (0.00059s latency).
Scanned at 2026-03-02 10:06:52 CST for 7s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 64
80/tcp   open  http    syn-ack ttl 64
1337/tcp open  waste   syn-ack ttl 64
MAC Address: XX (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 7.44 seconds
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

<br>

Vemos que hay 3 puertos abiertos, aunque nuestro interés se enfoca en el **puerto 80**.

<br>

<h2 id="Servicios">Escaneo de Servicios</h2>

```bash
nmap -sCV -p 22,80,1337 192.168.100.210 -oN targeted
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-02 10:07 -0600
Nmap scan report for 192.168.100.210
Host is up (0.00077s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 af:79:a1:39:80:45:fb:b7:cb:86:fd:8b:62:69:4a:64 (ECDSA)
|_  256 6d:d4:9d:ac:0b:f0:a1:88:66:b4:ff:f6:42:bb:f2:e5 (ED25519)
80/tcp   open  http    Apache httpd 2.4.62
|_http-title: Did not follow redirect to http://lavashop.thl/
|_http-server-header: Apache/2.4.62 (Debian)
1337/tcp open  waste?
MAC Address: XX (Oracle VirtualBox virtual NIC)
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.82 seconds
```

| Parámetros | Descripción |
|--------------------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

<br>

Gracias a este escaneo vemos que la página web activa en el **puerto 80**, utiliza un dominio, que podemos registrar en el `/etc/hosts`:
```bash
echo "192.168.100.210 lavashop.thl" >> /etc/hosts
```
Además, no vemos algo de información sobre el **puerto 1337**, pero enfoquemonos en el **puerto 80**.


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
<img src="/assets/images/THL-writeup-lavashop/Captura1.png">
</p>

Parece ser una tienda que vende lamparas de lava.

Veamos qué nos dice **Wappalizer**:

<p align="center">
<img src="/assets/images/THL-writeup-lavashop/Captura2.png">
</p>

<br>

<h2 id="fuzz">Fuzzing</h2>


```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt:FUZZ -u http://lavashop.thl/FUZZ -t 300 -e .php,.txt,.html -mc 200,301,302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://lavashop.thl/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200,301,302
________________________________________________

assets                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 4ms]
includes                [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 10ms]
index.php               [Status: 200, Size: 1539, Words: 192, Lines: 45, Duration: 3ms]
index.php               [Status: 200, Size: 1539, Words: 192, Lines: 45, Duration: 4ms]
pages                   [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 3ms]
.                       [Status: 200, Size: 1539, Words: 192, Lines: 45, Duration: 12ms]
:: Progress: [513480/513480] :: Job [1/1] :: 153 req/sec :: Duration: [0:01:39] :: Errors: 0 ::
```


```bash

```

```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt:FUZZ -u http://lavashop.thl/pages/FUZZ -t 300 -e .php,.txt,.html -mc 200,301,302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://lavashop.thl/pages/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt
 :: Extensions       : .php .txt .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200,301,302
________________________________________________

about.php               [Status: 200, Size: 208, Words: 33, Lines: 6, Duration: 2ms]
contact.php             [Status: 200, Size: 139, Words: 8, Lines: 5, Duration: 3ms]
home.php                [Status: 200, Size: 195, Words: 22, Lines: 6, Duration: 4ms]
products.php            [Status: 200, Size: 1017, Words: 189, Lines: 27, Duration: 2ms]
:: Progress: [513480/513480] :: Job [1/1] :: 284 req/sec :: Duration: [0:01:37] :: Errors: 0 ::
```

```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt:FUZZ -u http://lavashop.thl/pages/products.php?FUZZ=test -t 300 -fs 1017

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://lavashop.thl/pages/products.php?FUZZ=test
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 300
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 1017
________________________________________________

file                    [Status: 200, Size: 1370, Words: 221, Lines: 32, Duration: 3ms]
:: Progress: [128370/128370] :: Job [1/1] :: 169 req/sec :: Duration: [0:00:44] :: Errors: 0 ::
```

```bash

```


<br>
<br>
<hr>
<div style="position: relative;">
  <h1 id="Explotacion" style="text-align:center;">Explotación de Vulnerabilidades</h1>
</div>
<br>


<h2 id=""></h2>

```bash
<?php if (isset($_GET['file'])): ?>
  <section style="margin-bottom:1.5rem; padding:1rem; background:rgba(0,0,0,.25); border:1px solid rgba(255,255,255,.1); border-radius:12px;">
    <p style="margin:0 0 .75rem;">Incluyendo: <code><?= htmlspecialchars($_GET['file'], ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8') ?></code></p>
    <pre style="white-space:pre-wrap; overflow:auto; padding:.75rem; background:rgba(0,0,0,.35); border-radius:8px;">
<?php
    $target = $_GET['file'];
    @include $target;
?>
    </pre>
  </section>
<?php endif; ?>

<section>
  <h2>Productos destacados</h2>
  <div class="product-grid">
    <article class="product">
      <img src="/assets/images/lamp1.jpg" alt="Lámpara de lava azul">
      <h3>Lava Classic - Azul</h3>
      <p>Lámpara clásica de lava azul, llega a ser hipnotizante.</p>
    </article>
    <article class="product">
      <img src="/assets/images/lamp2.jpg" alt="Lámpara de lava roja">
      <h3>Lava Premium - Rojo</h3>
      <p>Lámpara clásica de lava roja, llega a ser hipnotizante.</p>
    </article>
    <article class="product">   
      <img src="/assets/images/lamparasal.jpg" alt="Lámpara de sal del Himalaya">
      <h3>Lámpara de sal</h3>
      <p>Lámpara de sal del Himalaya, te otorgará calma para los CTFs avanzados.</p>
    </article>
    <article class="product">   
      <img src="/assets/images/lamparaplasma.jpg" alt="Lámpara de plasma">
      <h3>Lámpara de plasma</h3>
      <p>Lámpara de plasma, dará ese toque hacker a tu habitación.</p>
    </article>
  </div>
</section>

```

```bash
<?php
error_reporting(E_ALL);
ini_set('display_errors', 1);

$BASE = '';

$page = isset($_GET['page']) ? $_GET['page'] : 'home';

if ($page === 'products' && isset($_GET['file'])) {
    $path = $_GET['file'];
    header('Content-Type: text/plain; charset=UTF-8');

    $forbidden_substrings = ['php://', 'data://', 'expect://', 'filter://', "\0", '%00'];
    foreach ($forbidden_substrings as $bad) {
        if (strpos($path, $bad) !== false) {
            http_response_code(400);
            echo "Ruta inválida\n";
            exit;
        }
    }

    $logs_prefixes = ['/var/log', '/var/logs', '/logs', __DIR__ . '/logs'];
    foreach ($logs_prefixes as $pref) {
        if (strpos($path, $pref) === 0) {
            http_response_code(403);
            echo "Acceso denegado\n";
            exit;
        }
    }

    $real = @realpath($path);
    $target = $real !== false ? $real : $path;

    if (@is_dir($target)) {
        http_response_code(400);
        echo "No es un archivo\n";
        exit;
    }

    $MAX_BYTES = 2 * 1024 * 1024;
    if (@file_exists($target) && @filesize($target) !== false && @filesize($target) > $MAX_BYTES) {
        http_response_code(413);
        echo "Archivo demasiado grande\n";
        exit;
    }

    $resolved = @realpath($target);
    if ($resolved !== false) {
        foreach ($logs_prefixes as $pref) {
            if (strpos($resolved, $pref) === 0) {
                http_response_code(403);
                echo "Acceso denegado\n";
                exit;
            }
        }
    }

    $contents = @file_get_contents($target);
    if ($contents === false) {
        http_response_code(404);
        echo "No se pudo leer\n";
        exit;
    }

    echo $contents;
    exit;
}
?>
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>LavaShop</title>
  <link rel="stylesheet" href="/assets/css/styles.css?v=1">
</head>
<body>

<?php require __DIR__ . '/includes/header.php'; ?>

<?php
if ($page === 'home' || $page === '' || $page === null) {
    ?>
    <section class="hero" style="padding: 3rem 0; text-align:center;">
      <h2>Bienvenido a LavaLamps Shop</h2>
      <p>Las mejores lámparas de lava para diseñar tu espacio.</p>
      <p style="margin-top:1rem;">
        <a class="cta" href="/index.php?page=products" style="display:inline-block;background:#ff445a;color:#fff;padding:.75rem 1.1rem;border-radius:10px;text-decoration:none;font-weight:700;">
          Ver catálogo
        </a>
      </p>
    </section>
    <?php
    require __DIR__ . '/includes/footer.php';
    echo "</body></html>";
    exit;
}

if (preg_match('/^[A-Za-z0-9_-]+$/', $page)) {
    $to_include = __DIR__ . '/pages/' . $page . '.php';
    if (file_exists($to_include)) {
        include $to_include;
        require __DIR__ . '/includes/footer.php';
        echo "</body></html>";
        exit;
    }
}

http_response_code(404);
echo '<section class="hero" style="padding:3rem 1rem; text-align:center;"><h2>404</h2><p>Página no encontrada.</p></section>';

require __DIR__ . '/includes/footer.php';
?>
</body>
</html>
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
