---
title: Crocodile Writeup
date: 2023-05-21
categories: [Writeups, Hackthebox]
tags: [Starting Point, Linux, Very Easy]
---
![img](/assets/img/post/crocodile/crocodile.jpg)
Resolucion de la maquina crocodile del Starting Point de HackThebox. 

## Reconocimiento 
Confirmamos que la maquina Victima este activa:
```bash
❯ ping -c 2 10.129.128.219
PING 10.129.128.219 (10.129.128.219) 56(84) bytes of data.
64 bytes from 10.129.128.219: icmp_seq=1 ttl=63 time=60.0 ms
64 bytes from 10.129.128.219: icmp_seq=2 ttl=63 time=59.0 ms

--- 10.129.128.219 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 59.036/59.525/60.014/0.489 ms
```

Enviamos 2 paquetes y los recibimos devuelta, por lo que la maquina se encuentra activa.
### Escaneo de puertos
Vamos a Escanear la maquina victima, para ver posibles puertos abiertos o servicios vulnerables:
```bash
❯ nmap -sV -sC 10.129.128.219
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-18 01:01 AST
Nmap scan report for 10.129.128.219 (10.129.128.219)
Host is up (0.055s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.212
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.59 seconds
```

## FTP Login
Miramos en nuestro escaneo, los servicios que corren en esta maquina, si se fijan en el puerto 21 (ftp), nos lista 2 archivos :
```text
ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
```
En una maquina anterior, ya habiamos usado este metodo, que seria hacer un login mendiante ftp, con el usuario por defecto, que seria "anonymous"

```bash
ftp 10.129.11.176
```
Luego, que logramos conexion mediante este servicio, vamos a extraer(Descargar), los 2 archivos anteriormente mencionados.

Primeramente Listamos el contenido, para ver que otras cosas podemos ver ahi dentro:

```bash 
ftp> dir
229 Entering Extended Passive Mode (|||43944|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
```

Necesitamos descargar esto 2 archivos, para ver su contenido, para eso utilizamos "get"

### Get Files
 
```bash
ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
229 Entering Extended Passive Mode (|||44046|)
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
100% |*******************************************************************************************************************************************|    33       19.32 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.56 KiB/s)
```
> >
```bash
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
229 Entering Extended Passive Mode (|||42178|)
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
100% |*******************************************************************************************************************************************|    62      651.04 KiB/s    00:00 ETA
226 Transfer complete.
62 bytes received in 00:00 (1.08 KiB/s)
```

Miramos el contenido de cada archivo, que ya debemos de tener descargado en nuestra maquina.

```bash
❯ cat allowed.userlist
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: allowed.userlist
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ aron
   2   │ pwnmeow
   3   │ egotisticalsw
   4   │ admin
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ cat allowed.userlist.passwd
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: allowed.userlist.passwd
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ root
   2   │ Supersecretpassword1
   3   │ @BaASD&9032123sADS
   4   │ rKXM59ESxesUFHAd
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
## HTTP  (Web - Puerto 80)

Pasamos a explorar el otro puerto abierto encontrado en el escaneo.

Si vamos al navegador y entramos a la pagina de la maquina victima podemos ver esta pagina.

![img](/assets/img/post/crocodile/page.png)

### Brute Force (Gobuster)

Vamos a aplicar fuerza bruta a la url de la maquina victima, para ver si podemos listar archivos de esta.

Utilizaremos Gobuster.

Gobuster es una herramienta utilizada para realizar fuerza bruta a: URIs (directorios y archivos) en sitios web, subdominios DNS (con soporte de comodines), y nombres de hosts virtuales en los servidores web.

```bash
❯ gobuster dir -u 10.129.11.176 -w /usr/share/dirb/wordlists/common.txt -x .php
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.11.176
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2023/01/18 13:35:02 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/.hta.php             (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htaccess.php        (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htpasswd.php        (Status: 403) [Size: 278]
/assets               (Status: 301) [Size: 315] [--> http://10.129.11.176/assets/]
/config.php           (Status: 200) [Size: 0]
/css                  (Status: 301) [Size: 312] [--> http://10.129.11.176/css/]
/dashboard            (Status: 301) [Size: 318] [--> http://10.129.11.176/dashboard/]
/fonts                (Status: 301) [Size: 314] [--> http://10.129.11.176/fonts/]
/index.html           (Status: 200) [Size: 58565]
/js                   (Status: 301) [Size: 311] [--> http://10.129.11.176/js/]
/login.php            (Status: 200) [Size: 1577]
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/server-status        (Status: 403) [Size: 278]
Progress: 9175 / 9230 (99.40%)===============================================================
2023/01/18 13:35:56 Finished
===============================================================
```
En nuestro output, miramos varios archivos encontrados, pero en este caso vamos a ir primero por el archivo login.php.

### Login.php (file)
Ya que en el puerto ftp, encontramos unos archivos con listas de usuarios, vamos a tratar de iniciar secion con estos.

Vamos al navegador y en el dominio, agregamos el login.php, para intentar acceder:

![img](/assets/img/post/crocodile/login1.png)

Nos carga esta pagina:

![img](/assets/img/post/crocodile/login2.png)

Aqui Probamos con los users que tenemos en los archivos descargados anteriormente.

Ya con al iniciar secion tenemos la flag, que buscamos.

![img](/assets/img/post/crocodile/flag.png)

## Task (Preguntas a responder)

Ahora pasamos a responder todas las preguntas, para completar esta maquina.

### Task 1.
What nmap scanning switch employs the use of default scripts during a scan?
```text
-sC
```
### Task 2. 
What service version is found to be running on port 21?
```text
vsftpd 3.0.3
```
### Task 3. 
What FTP code is returned to us for the "Anonymous FTP login allowed" message?
```text
230
```
### Task 4.
What command can we use to download the files we find on the FTP server?
```text
get
```
### Task 5.
What is one of the higher-privilege sounding usernames in the list we retrieved?
```text
admin
```
### Task 6.
What version of Apache HTTP Server is running on the target host?
```text
2.4.41
```
### Task 7.
What is the name of a handy web site analysis plug-in we can install in our browser?
```text
Wappalyzer
```
### Task 8.
What switch can we use with gobuster to specify we are looking for specific filetypes?
```text
-x
```
### Task 9.
What file have we found that can provide us a foothold on the target?
```text
login.php
```






