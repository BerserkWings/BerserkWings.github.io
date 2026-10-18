---
title: "NombreMaquina - TheHackersLabs"
date: 20XX-XX-XX
# Opcional: unlock_date: 20XX-XX-XX
draft: false
slug: "THL-writeup-NombreMaquina"
excerpt: "Descripción de solución"
categories: ["TheHackerLabs", "Dificultad Machine"]
tags: ["OSCP Style"]
tools_kali: ["ping", "nmap"]
tools_target: ["whoami"]
links: ["links de investigación"]
showtoc: true
header:
  teaser: "/assets/images/THL-writeup-NombreMaquina/imagenMaquina.png"
---


  <!-- ======== ENUMERACIÓN ======== -->


## Recopilación de Información {#Recopilacion}

### Traza ICMP {#Ping}

Vamos a realizar un ping para saber si la máquina está activa y en base al TTL veremos qué sistema operativo (SO) opera en la máquina.
```bash
ping
```
Por el TTL sabemos que la máquina usa **SO**, hagamos los escaneos de puertos y servicios.

### Escaneo de Puertos {#Puertos}

```bash
nmap puertos
```

| Parámetros | Descripción |
|------------|-------------|
| *-p-*      | Para indicarle un escaneo en todos los puertos. |
| *--open*   | Para indicar que aplique el escaneo en los puertos abiertos. |
| *-sS*      | Para indicar un TCP Syn Port Scan para que nos agilice el escaneo. |
| *--min-rate* | Para indicar una cantidad de envío de paquetes de datos no menor a la que indiquemos (en nuestro caso pedimos 5000). |
| *-vvv*     | Para indicar un triple verbose, un verbose nos muestra lo que vaya obteniendo el escaneo. |
| *-n*       | Para indicar que no se aplique resolución DNS para agilizar el escaneo. |
| *-Pn*      | Para indicar que se omita el descubrimiento de hosts. |
| *-oG*      | Para indicar que el output se guarde en un fichero grepeable. Lo nombre allPorts. |

Resumen...

### Escaneo de Servicios {#Servicios}

```bash
nmap servicios
```

| Parámetros | Descripción |
|------------|-------------|
| *-sC*      | Para indicar un lanzamiento de scripts básicos de reconocimiento. |
| *-sV*      | Para identificar los servicios/versión que están activos en los puertos que se analicen. |
| *-p*       | Para indicar puertos específicos. |
| *-oN*      | Para indicar que el output se guarde en un fichero. Lo llame targeted. |

Resumen...


  <!-- ======== ANÁLISIS VULNERABILIDADES ======== -->


## Análisis de Vulnerabilidades {#Analisis}

Contenido...

<p align="center">
<img src="/assets/images/THL-writeup-NombreMaquina/Captura1.png">
</p>


  <!-- ======== EXPLOTACIÓN ======== -->


## Explotación de Vulnerabilidades {#Explotacion}

Contenido..

| **Concepto** |
|:--------------------------------------:|
| *Descripción* |

```bash
comando
```


  <!-- ======== POST EXPLOTACIÓN ======== -->


## Post Explotación {#Post}

Contenido...

```bash
comando
```

Ya solo busquemos la última flag:
```bash
cat /root/root.txt
...
```
Y con esto terminamos la máquina.

  <!-- ======== FIN ======== -->
