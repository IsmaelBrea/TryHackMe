
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

Sin embargo obtuve lo siguiente: does not support password authentication. Lo que indica que ssh no tiene contraseña  y por tanto no podemos usar hydra. Esto lo confirmarmos con:
```bash
ssh -v R1ckRul3s@10.130.133.241
```

>  Authentications that can continue: publickey

Por tanto Hydra no servía.

`-l` -> login (usuario a probar)
`-P` -> password list (Indica un archivo de contraseñas)

 ### Gobuster
He probado distintas combinaciones en gobuster para ver si encontraba algo y he encontrado un php con el siguiente comando:
 ```bash
 gobuster dir -u http://172.17.0.2/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -t 20 -x html,php,txt,php.bak
---------------------------------------------------------------------------------
/index.html           (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]  
/server-status        (Status: 403) [Size: 275
 ```

Este comando utiliza una wordlist que le proporcionamos con el parámetro -w para probar posibles directorios y archivos en la web.
El parámetro -t indica el número de hilos concurrentes, acelerando el proceso de búsqueda.
La opción -x permite probar diferentes extensiones (por ejemplo, .php, .html) sobre cada palabra de la wordlist.

En conjunto, Gobuster intenta encontrar archivos o directorios en la URL o IP que le indicamos, que en este caso corresponde a la máquina objetivo donde está alojada la página web y la plantilla de Apache.

![Gobuster](/images/gobuster_1.png)

Al acceder a /secret.php podemos ver lo siguiente:

![PHP](/images/secret_php.png)


## Explotación
Solo tenemos un vector de ataque con la información que tenemos. Sabemos que está abierto el puerto 22 y que hay un usuario llamada Mario. Por tanto vamos a realizar fuerza bruta a este puerto utilizando **hydra**.
Tenemos que usar la wordlist rockyou que viene instalada ya en Kali. Suele venir instalado pero viene en un zip por lo que hay que descomprimirlo una vez. 
Localiza el archivo:
```bash
ls /usr/share/wordlists/rockyou.txt.gz
```

Debería estar en:
```swift
/usr/share/wordlists/rockyou.txt.gz
```

Descomprimimos el archivo  (solo lo tenemos que hacer una vez):
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz

Ahora ya podemos usar la wordlist:
```swift
/usr/share/wordlists/rockyou.txt
```

Bien, ahora ya podemos usar hyndra para obtener ka contraseña del usuario Mario para realizar luego ssh
Usaremos el siguiene comando:
```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.19.0.2 -t 4
```


hydra → ejecuta la herramienta Hydra.

-l mario → indica el usuario único que queremos atacar, en este caso mario.

-P /usr/share/wordlists/rockyou.txt → indica la lista de contraseñas que Hydra va a probar para ese usuario.

Hydra probará cada contraseña del archivo rockyou.txt contra el usuario mario.

ssh://172.19.0.2 → especifica el servicio y la dirección del objetivo:

ssh → protocolo que se va a atacar.

172.19.0.2 → IP de la máquina donde Hydra intentará conectarse.

-t 4 → número de hilos concurrentes, es decir, Hydra hará 4 intentos al mismo tiempo para acelerar el proceso.


![Hydra1](/images/hydra1.png)

Obtenemos la contraseña, que como vemos es chocolate.

Ya podemos realizar ssh para entrar como Mario

```bash
ssh mario@172.19.0.2
```

### Tratamiento de la tty

Realizaremos un breve **tratamiento de la tty** para poder operar de forma cómoda sobre la consola. Los comandos a ejecutar:

```shell
script /dev/null -c bash 
```
(hacemos  **ctrl  +  Z**)

```shell
stty raw -echo; fg
reset xterm
stty rows 62 columns 248
export TERM=xterm
export SHELL=bash
```

Pondremos en rows y columns las columnas y filas que correspondan a la pantalla de nuestra máquina.
Una vez hecho esto podemos maniobrar con comodidad, pudiendo hacer Ctrl+L para limpiar la pantalla así como Ctrl+C.

## Escalada de privilegios

Usamos **sudo -l** para ver si podemos ejecutar algo como root:

```shell
sudo -l
-----------------------
User mario may run the following commands on 78a58e094bf9:
    (ALL) /usr/bin/vim
```

Podemos ejecutar **vim**. Si no sabemos como explotarlo para escalar privilegios, siempre podemos consultar -> [GTFOBins](https://gtfobins.github.io/)

```shell
sudo vim -c ':!/bin/bash'
```

```shell
whoami
----------------
root
```

Hemos alcanzado el nivel de privilegios máximos en el sistema!







