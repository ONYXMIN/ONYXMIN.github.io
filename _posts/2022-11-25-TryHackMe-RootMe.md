---
title: TryHackMe - RootMe
cover:
    image: "images/rootme-tryhackme/rootme.png"
categories: [Linux]
tags: [TryHackMe, eJPT, Linux]
---

***En este post voy a explicar paso por paso como hacer la máquina [RootMe](https://tryhackme.com/room/rrootme) de TryHackMe.***

* SO: Linux

* Dificultad: Fácil

* Dirección IP: 10.10.143.118

# Reconocimiento

Para empezar lo primero que vamos a hacer es comprobar si la máquina está actualmente activa y que SO tiene.

```
ping -c 1 10.10.143.118 
```
En este caso el TTL es de 63
![](https://i.postimg.cc/pdh7snb5/ping.png)

> TTL=64: Linux

> TTL=128: Windows

Por proximidad podemos decir que esta máquina es Linux.

### Escaneo de puertos

```
nmap -p- --open -sS --min-rate 5000 -n -vvv -Pn 10.10.143.118 -oG allPorts
```
Para extraer los puertos abiertos del fichero allPorts usaremos la [función de extractPorts](https://pastebin.com/tYpwpauW) de [s4vitar](https://github.com/s4vitar).

![](https://i.postimg.cc/7YRnmtxX/extract-Ports.png)

Ahora con nmap vamos a intentar buscar las versiones y servicios de los puertos abiertos.

```
nmap -p22,80 -sCV 10.10.143.118 -oN targeted
```
![](https://i.postimg.cc/J0vHcgbk/versiones-Yservicios.png)
Podemos observar que en el puerto 80 hay un servidor web, vamos a echarle un ojo

![](https://i.postimg.cc/k4D51sMF/pagina.png)

Como podemos ver, no hay gran cosa en la página, vamos a usar la herramienta wfuzz para descubrir directorios.

```
wfuzz -c --hc=404 -t 150 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.143.118/FUZZ
```

![](https://i.postimg.cc/QdspMFqG/wfuzz.png)

Al buscar directorios con wfuzz, podemos ver que hay 2 que son bastante llamativos.

* panel -> En este directorio tenemos un panel para subir archivos.
* uploads -> En este directorio se almacenan los archivos que subamos

Intentemos subir una reverse shell con extensión .php para ver si nos deja.

![](https://i.postimg.cc/yx4DrGQ4/php-blocked.png)

Al parecer hay extensiones que no están permitidas, vamos a hacer fuerza bruta con burpsuite para saber cuáles están permitidas. 
Vamos a capturar la petición de la subida del archivo y la mandaremos al Intruder.

![](https://i.postimg.cc/MHqWBJ3T/intruder.png)

Una vez en el intruder marcaremos el lugar al cual vamos a hacer fuerza bruta y le pondremos los payloads.


![](https://i.postimg.cc/pLhtpdVC/positions.png)

![](https://i.postimg.cc/T3T4v48x/attack.png)

![](https://i.postimg.cc/P5ZFgvrg/result.png)

Como podemos ver hay 2 extensiones que nos devuelven distintos caracteres como respuestas del servidor, por lo que podemos intentar subir archivos con esas extensiones, lo que subiremos es un [reverse shell](https://raw.githubusercontent.com/nop-tech/Pentesting/main/Reverse-Shells/php-reverse-shell.php) de .php a la cual le cambiaremos la extensión a .php5, tambien recuerden modificar el campo de la IP por la de ustedes.

Una vez subida nos pondremos en escucha con netcat y nos dirigiremos al directorio /uploads en el cual va a estar almacenada la reverse shell, le daremos clic y nos tendría que llegar la shell.

### Preguntas
1. Scan the machine, how many ports are open? - ```2```
2. What version of Apache is running? - ```2.4.29```
3. What service is running on port 22? - ```ssh```
4. What is the hidden directory? - ```/panel/```


## Reverse Shell 

![](https://i.postimg.cc/K8HBD02F/shell.png)
Una vez obtenida la reverse shell, vamos a hacer un [tratamiento de la TTY](/posts/tratamiento-de-la-TTY).

```
find / -type f -name "user.txt" 2>/dev/null | xargs cat
```
![](https://i.postimg.cc/sD2MH5K2/user-flag.png)

### Preguntas
1. user.txt - ```THM{y0u_g0t_a_sh3ll}```


## Escalada de Privilegios

Esta máquina ya nos adelanta que se escala abusando de un archivo que tiene permisos SUID, vamos a buscar nuevamente con el comando find.

```
find / -perm -4000 2>/dev/null
```
![](https://i.postimg.cc/fWqzLpjw/python.png)

Al buscar por ficheros con permisos SUID, vemos que nos reporta /usr/bin/python, vamos a buscarlo en [GTFOBins](https://gtfobins.github.io/gtfobins/python/#suid).


![](https://i.postimg.cc/05KkzZtD/gtfobins.png)

Como vemos hay una forma de explotarlo, vamos a probar ese comando.

![](https://i.postimg.cc/jjCVGdmP/explotacion.png)

Genial! ya escalamos nuestro privilegio y somos el usuario root.

### Preguntas

1. root.txt - ```THM{pr1v1l3g3_3sc4l4t10n}```
