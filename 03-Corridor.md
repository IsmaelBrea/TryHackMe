
# Máquina Corridor

Dificultad -> Muy Fácil

Enlace a la máquina -> [Corridor](https://tryhackme.com/room/picklerick](https://tryhackme.com/room/corridor))

El objetivo de la máquina es encontrar 1 flag.

## Despliegue del laboratorio

Desplegar VPN y comprobar la interfaz tun0 con `ip a`.


## Reconocimiento

Usamos nmap para ver puertos abiertos. Cualquier comando de nmap que permita ver puertos sirve aquí. Encontramos que solo tiene el puerto 80 abierto. Copiamos la IP en el navegador y nos lleva a una web que contiene una imagen. 

<img width="1126" height="651" alt="imagen" src="https://github.com/user-attachments/assets/18edd4e5-ccdd-4436-a6f5-3d1b1745bd25" />

<br>

Cada puerta tiene una URL distinta. Es una cadena de caractere. Si nos fijamos bien y leyendo las bases de la máquina nos damos cuenta que es un hash. Sabemos que es MD5 porque tiene 128 bits (32 caracteres hexadecimales). 

<img width="1452" height="262" alt="imagen" src="https://github.com/user-attachments/assets/f99a9680-b011-4f2e-86c5-88a9eb78b048" />

<br>

128 bits = 16 bytes = 32 caracteres hexadecimales

Podemos copiar estos hashes en alguna página como hashes.com para ver cuál es su valor real. Por ejemplo si quremos obtener c4ca4238a0b923820dcc509a6f75849b en md5 obtenemos como resultado 1. Esto lo que quiere decir es que las puertas posiblemente tengan un orden númerico pero están hasheadas. 

> Aquí hay que corregir una cosa: realmente no desciframos MD5. Lo que hacemos es comprobar valores posibles.

Lo que yo hice fue ir escribiendo los hashes uno a uno desde la izquierda en hashes.com para ver si veía algo raro en alguno. Pero no, simplemente estaban todos numerados del 1 al 13, que era el número de puertas que había. Lo que se me ocurrió por tanto fue probar a hashear el 0 y el 14, los números anterior y siguiente al que no estaban ahí y probar a ponerlos en la URL. Eso precisamente es un IDOR. La aplicación permite acceder a recursos cambiando un identificador que debería estar protegido.

Para ello utilicé el siguiente comando en Linux:
```bash
echo -n "0" | md5sum
```

cfcd208495d565ef66e7dff9f98764da

Y copie la salida del hash en la URL y... obtuve la FLAG!!

<img width="1724" height="586" alt="imagen" src="https://github.com/user-attachments/assets/f9cc2c70-155b-45b4-934f-9ebbe1505f02" />

<br>

 Máquina completada: Corridor ✅

