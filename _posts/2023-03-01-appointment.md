---
title: Appointment Writeup
date: 2023-05-17 
categories: [Writeups, Hackthebox]
tags: [Starting Point, Linux, Very Easy]
image:
  path: ../../assets/img/post/appointment/appointment.png
  width: 800
  height: 500
  alt:
---

En este articulo vamos a estar resolviendo la maquina appointment del starting point de HackTheBox  

## Task 1. What does the acronym SQL stand for?

*¿Qué significa el acrónimo SQL?*

Esto con una simple busqueda en google lo tenemos:

```text 
Structured Query Language
```

## Task 2. What is one of the most common type of SQL vulnerabilities?

*¿Cuál es uno de los tipos más comunes de vulnerabilidades de SQL?*

```text
SQL Injection
```

Este tema es muy interesante, mas adelante voy a dedicar un articulo completo, a inyecciones SQL.

## Task 3.  What does PII stand for?

*¿Qué significa PII?*

```text 
Personally identifiable information
```

## Task 4. What does the OWASP Top 10 list name the classification for this vulnerability?

*¿Cuál es el nombre de la lista OWASP Top 10 para la clasificación de esta vulnerabilidad?*

```text
A03:2021-Injection
```

## Task 5. What service and version are running on port 80 of the target?

*¿Qué servicio y versión se ejecutan en el puerto 80 del destino?*

Vamos a realizar un escaneo con la herramienta nmap a la maquina victima: 
```bash 
❯ sudo nmap -sC -sV --min-rate 5000 10.129.111.208
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-29 15:16 AST
Nmap scan report for 10.129.111.208
Host is up (0.17s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.67 seconds
```
Como podemos ver en nuestro resultado del escaneo tenemos el servicio y version que corre en la maquina victima.

```text
Apache httpd 2.4.38 ((Debian))
```

## Task 6. What is the standard port used for the HTTPS protocol?

*¿Cuál es el puerto estándar utilizado para el protocolo HTTPS?*

Si hacemos una busqueda en google, esta nos da como resultado:

By default, these two protocols are on their standard port number of 80 for HTTP and 443 for HTTPS.

```text
443
```
O puede ser Puerto 80.

## Task 7. What is one luck-based method of exploiting login pages? 

*¿Cuál es un método basado en la suerte para explotar las páginas de inicio de sesión?*

```text 
brute-forcing
```

## Task 8. What is a folder called in web-application terminology?

*¿Cómo se llama una carpeta en la terminología de aplicaciones web?*

```text
directory
```

## Task 9. What response code is given for "Not Found" errors?

*¿Qué código de respuesta se da para los errores 'No encontrado'?*

Esto es muy comun en paginas webs y seguramente ya lo habias escuchado, se refiere al error:
```text
404
```
## Task 10. What switch do we use with Gobuster to specify we're looking to discover directories, and not subdomains?

*¿Qué interruptor usamos con Gobuster para especificar que buscamos descubrir directorios y no subdominios?*

```text
dir
```

Vamos a intentar acceder al login de la maquina victima, anteriormente habiamos visto que la maquina corria con el servicio http.

Entoces vamos a intentar ver la web en el navegador:

```text 
http://10.129.21.212
```

![login](/assets/img/post/appointment/login.png)

Probamos iniciar sesion. aqui algunos usarios de los que probe 
```text
user
root
admin
```

Para poder aplicar fuerza bruta tenemos que filtar con este user:
```text 
admin'#
```
Y en la parte de la password, no nos deja iniciar con el espacio en blanco, asi que ponemos cualquier caracter y iniciamos sesion.


## What symbol do we use to comment out parts of the code?

*¿Qué símbolo usamos para comentar partes del código?*

```text
#
```

Como lo habiamos usado en la pregunta anterior.

## Submit root flag

Al iniciar sesion nos muestra la flag.

```text
Congratulations!

Your flag is: e3d0796d002a446c0e622226f42*****
```
