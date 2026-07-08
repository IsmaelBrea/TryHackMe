
# Máquina Pickle Rick 

Dificultad -> Fácil

Enlace a la máquina -> [TryHackMe]([https://dockerlabs.es/](https://tryhackme.com/room/picklerick))

El objetivo de la máquina es encontrar 3 flags, que son 3 ingredientes.

## Despliegue del laboratorio

Desplegar VPN y comprobar la interfaz tun0 con `ip a`.


## Reconocimiento

Tenemos la Ip de la máquina víctima. El primer paso realizado fue un escaneo completo de nmap para ver puertos y servicios:
```bash
nmap -sS -sV -p- -min-rate 5000 -vvv 10.130.133.241
```

Es un escaneo SYN scan (-sS), es decir, no llega a completar el handshake aunque es indiferente puesto que es un laboratorio. Detectamos versiones de los servicios con -sV y con `--min-rate 5000`  aumenta la velocidad enviando al menos 5000 paquetes por segundo.


Podemos comprobar que están abiertos los puertos 22 (ssh) h 80 (http):

<img width="849" height="78" alt="imagen" src="https://github.com/user-attachments/assets/582d4220-26f3-4f10-b2cc-15bb61007974" />

<br>

En este punto ya tenemos dos vectores de ataque por donde podemos empezar. 

En primer caso es recomendable ver que contiene el puerto 80 así que lo pegamos con su IP en el navegador para ver que contiene. En este caso no funciona, no lleva a ningún lado. Aun así probamos un curl para ver que nos devuelve:
```bash
curl http://10.130.133.241
```

Esto sí que funciona y nos devuelve un código HTML que contiene algún dato interesante. Lo más destacado es un nombre al final: 
```html
Username: R1ckRul3s
```
 y algún dato como que tenemos que ayudar a Morty a encontrar ingredientes para que deje de ser un pepinillo.

## Enumeración Web - Fuzzing

A partir de lo anterior, tenemos varias opciones por donde tirar. Tenemos ya un nombre de usuario con el que podemos probar varias cosas. Lo primero que hice al obtenerlo y teniendo en cuenta que teníamos el puerto 22 abierto fue prpbar fuerza bruta con hydra para ver si podía obtener alguna contraseña con la que acceder al SSH. Para ello usé:
```bash
hydra -l R1ckRul3s - P /usr/share/wordlists/rockyou.txt ssh://10.130.133.241
```

`-l` -> login (usuario a probar)
`-P` -> password list (Indica un archivo de contraseñas)

Sin embargo obtuve lo siguiente: does not support password authentication. Lo que indica que ssh no tiene contraseña, funcionaba solo por clave pública  y por tanto no podemos usar hydra. Esto lo confirmamos con:
```bash
ssh -v R1ckRul3s@10.130.133.241
```

>  Authentications that can continue: publickey

Por tanto Hydra no servía.



 ### Gobuster

Otra posibilidad a probar es el fuzzing, que consiste en enumerar directorios y ficheros secretos. Para ello usamos la herraienta gobuster como principal. 
He probado distintas combinaciones en gobuster para ver si encontraba algo y he encontrado un php y un txt importantes:
```bash
gobuster dir -u http://10.130.133.241 -w /usr/share/wordlists/dirb/common.txt
```

`-u` -> url
`-w` -> wordlist (diccionario)

Este comando utiliza una wordlist que le proporcionamos con el parámetro -w para probar posibles directorios y archivos en la web.
El parámetro -t indica el número de hilos concurrentes (aunque no hace falta), acelerando el proceso de búsqueda.
La opción -x permite probar diferentes extensiones (por ejemplo, .php, .html) sobre cada palabra de la wordlist. Eso lo usé luego.

Aquí solo encontré un .txt (robots.txt) que contenía lo que parecía una contraseña: Wubbalubbadubdub

Volví a probar gobuster con más opciones y encontré un archivo .php: 
```bash
gobuster dir -u http://10.130.133.241 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

<img width="866" height="684" alt="imagen" src="https://github.com/user-attachments/assets/447464af-3c8e-4d4f-bee2-c8246ac4592c" />

<br>

El archivo importante que encontramos ahora es el .php. Lo probamos en el navegador y a su vez probamos un curl:
```bash
curl http://10.130.133.241/login.php
```

Aquí veíamos el html y era un formulario paa rellenar usuario y nombre. Desde el navegador probamos a acceder con el usuario y contraseña obtenidos y entramos a un panel que permite la ejecución de comandos. Aquí estarán las 3 flags.


## Explotación

Solo tenemos un vector de ataque y es saber usar bien los comandos de Linux desde la web esta para poder encontrar las flags. Es lo que se conoce como una web shell. Lo primero que se nos puede ocurrir es probar lo siguiente:
```
whoami  # www-data
pwd     # /var/www/html
ls -la  # index.html, login.php, portal.php, robots.txt, Sup3rS3cretPickl3Ingred.txt
```

**Ingrediente 1**

Después del ls, lo normal es probar cat en todos los archivos. El cat estaba bloqueado, por lo que había que usar otro comando de lectura. Probé less, que es un visor de archivos de texto y obtuve la primera flag en el archivo Sup3rS3cretPickl3Ingred.txt
```bash
less Sup3rS3cretPickl3Ingred.txt
```

<br>

**Ingrediente 2**
Para el ingrediente 2, después de ir comprobando cosas, vi que en el directorio home había dos directorios más, uno de rick (/home/rick) y otro de ubuntu (/home/ubuntu).

Comprobé en ambos y en el directorio de rick encontré un archivo llamado `second ingredients`:
```bash
ls -la /home/rick
less "/home/rick/second ingredients"
```

Con less obtuve el segundo flag.

<br>

**Ingrediente 3 - sudo mal configurado**

Esta flag estaba relacionado con la **escalada de privilegios**. Para ver que podía hacer root hice lo siguiente:
```bash
sudo -l
```

Esto muestra qué comandos puede ejecutar un usuario con sudo.

Resultado:
```bash
User www-data may run:
(ALL) NOPASSWD: ALL
```

Esto significa:

- Usuario actual: www-data
- Puede ejecutar cualquier comando
- Como cualquier usuario
- Sin contraseña

Por tanto comprobé:
```bash
sudo root
whoami # root
```

Al ser root busqué en sus archivos: 
```bash
sudo ls -la /root/
```
 Y encontre un archivo 3rd.txt que parecía el último flag:
 ```bash
sudo less /root/3rd.txt
```
 Y efectivamente. 


 Máquina completada: Pickle Rick ✅

