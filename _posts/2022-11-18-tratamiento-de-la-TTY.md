---
title: Tratamiento de la TTY
cover:
    image: "images/Otros/tratamiento-de-la-tty/foto-portada.jpg"
categories: [Linux, Config, TTY]
tags: [Linux, TTY]
---

Bienvenido a mi primer post, en el cual voy a enseñar a como hacer un buen tratamiento de la TTY. **^^**


## Antes de empezar, ¿Qué es una TTY?

**La TTY es la consola que nos permite en GNU/Linux acceder a nuestro sistema operativo fuera de su entorno gráfico.**

**Dicho esto, empecemos.**

Luego de haber ganado acceso a un sistema, vamos a tener que estar en una consola en la cual podamos movernos cómodamente, para hacer el tratamiento de la TTY vamos a seguir estos pasos:


![Captura-de-pantalla-2022-10-21-145658.png](https://i.postimg.cc/R0qVSPCz/Captura-de-pantalla-2022-10-21-145658.png)

Luego de hacer `script /dev/null -c bash` se nos lanza una Pseudo consola

Presionamos ctrl + z para suspender la Shell

![ctrl-z.png](https://i.postimg.cc/XJP0239Z/ctrl-z.png)

Una vez hecho eso, el proceso se va a quedar en segundo plano.

Lo restableceremos poniendo lo siguiente 

![stty.png](https://i.postimg.cc/T2kZJRTf/stty.png)

Luego de poner eso se reiniciara la configuración de la terminal, escribiremos `reset` y cuando nos pregunte el tipo de terminal que queremos le pondremos `xterm`

![stty.png](https://i.postimg.cc/jdtBTLvH/stty.png)

Pondremos `export TERM=xterm` y `export SHELL=bash`.

Con esto le diremos que como tipo de terminal queremos usar una xterm y como tipo de Shell una bash.

![Captura-de-pantalla-2022-10-21-152937.png](https://i.postimg.cc/ZnVG2mkt/Captura-de-pantalla-2022-10-21-152937.png)

Para terminar vamos a terminar de configurar las proporciones, en nuestra propia máquina escribiremos 'stty size' para ver el tamaño de nuestra Shell

![size.png](https://i.postimg.cc/Vvx9G5YH/size.png)

Y las seteamos en la máquina víctima (en mi caso 51 filas y 189 columnas)

![Captura-de-pantalla-2022-10-21-154225.png](https://i.postimg.cc/ZKDPD7qy/Captura-de-pantalla-2022-10-21-154225.png)

Listo, ya podemos utilizar nuestra Shell 100% interactiva.

**Happy Hacking! ^^**
