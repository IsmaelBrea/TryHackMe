
# Máquina Lo-Fi

Dificultad -> Fácil

Enlace a la máquina -> [Lo-Fi](https://tryhackme.com/room/lofi)

El objetivo de la máquina es encontrar 1 flag.

## Despliegue del laboratorio

Desplegar VPN y comprobar la interfaz tun0 con `ip a`.


<br>
<br>


En este desafío, nos saltamos el escaneo Nmap. Dado que la descripción de la habitación ya nos pide que visitemos una página web, y debemos probar la inclusión de archivos locales aquí. En la página Índice tenemos enlaces a diferentes géneros.

<img width="1200" height="604" alt="imagen" src="https://github.com/user-attachments/assets/aa2c5bac-bc5a-4b6e-beb8-c8f1d833ca59" />


Tendremos que usar 2 técnicas:

- **LF IPath Traversal:**

Es una vulnerabilidad que permite acceder a archivos o directorios fuera de la carpeta permitida utilizando rutas como ../.

Por ejemplo, si una aplicación permite abrir archivos:

?page=manual.pdf

un atacante podría intentar:

?page=../../../etc/passwd

Los ../ hacen que se suba de directorio hasta llegar a archivos del sistem.

<br>

- **File Inclusion:**

Es una vulnerabilidad en la que una aplicación incluye o carga un archivo cuyo nombre o ruta depende de la entrada del usuario sin validarla correctamente. Ejemplo en PHP:
```php
include($_GET['page']);
```

Si el usuario controla el parámetro page, puede conseguir que la aplicación cargue archivos no previstos.
  

Por tanto al acceder a la web y ver el parámetro ?page sabemos que puede ser vulnerable a LFLI, Podemos probar de forma manual a poner /etc/passwd después de ?page. Esto es lo que sale:
<img width="1849" height="620" alt="imagen" src="https://github.com/user-attachments/assets/df837f12-a6a0-4ecb-ae0a-0174d047a112" />

<br>

Tenemos que probar a volver directorios hacia atrás hasta llegar al directorio raíz como indica el enunciado. Ahí estará el fichero con la flag. A mano probando encuentro /etc/passwd en la siguiente url:

http://10.128.166.181/?page=../../../../etc/passwd


Esto se podría haber automatizado con herramientas de fuzzing y con un diccionario especial para LFI. Hay algunos en Kali ya incluidos en /usr/share/wordlists/seclists/LFI. Podemos usar ffuf o wfuzz:
```bash
ffuf -w /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt \
-u "http://10.128.166.181/?page=FUZZ" \
-fl 124
```

La flag se puede encontrar en:

http://MACHINE_IP/?page=../../../flag.txt

 Máquina completada: Pickle Rick ✅

