
# Máquina OhSINT

Dificultad -> Fácil

Enlace a la máquina -> [OhSINT](https://tryhackme.com/room/ohsint)

El objetivo de la máquina es encontrar distintas flags a partir de una imagen.

## Despliegue del laboratorio

Descargar imagen en local para operar a partir de ella


<br>

--- 
A partir de la imagen podemos usar exiftool, que es una herramienta para obtener info de los metadatos de la misma. 

<img width="793" height="576" alt="imagen" src="https://github.com/user-attachments/assets/a5d02ecb-bc32-416e-b4f5-53a2f7cf08a2" />

Aquí obtenemos información importante sobre la misma. Podemos ver el nombre del Copyright: OWoodflint que os puede dar info valiosa a la hora de buscar en Internet. Otra información relevante es la del GPS y sus coordenadas.

**Flag 1**

Nos pide obtener de donde es el avatar del usuario. El usuario sabemos que puede ser el del copyrigth, por lo que podemos buscarlo en Internet. 

SI lo buscamos aparece bastante información sobre él. Así lo más importante que encontramos, es su perfil de Github, de Twitter y un blog en Worpress.

si nos fijamos en el avatar de cada foto de perfil, vemos que en Twitter tiene un gato. Por tanto la flag 1 será cat.

<br>

**Flag 2**

Nos pide saber de que ciudad es el usuario. Si miramos su perfil de Github ya nos dice que es de London:

<img width="657" height="561" alt="imagen" src="https://github.com/user-attachments/assets/477ac546-f671-446a-a3db-b73f61fd7fca" />

<br>

**Flag 3**

Nos pide saber a que SSID the una WAP está conectado. 

En su primer tweet, del 3 de Marzo de 2019, nos comenta que ha logrado obtener WiFi gratis desde su casa y nos comparte el BSSID de la misma. Perfecto! Vamos a indagar en Wigle y ver si logramos dar con dicha red.

WiGLE (Wireless Geographic Logging Engine) es una base de datos colaborativa que almacena información sobre redes Wi-Fi y torres de telefonía móvil de todo el mundo. Su principal uso en OSINT es buscar una red Wi-Fi a partir de su BSSID (dirección MAC) o de su SSID (nombre de la red).

Si buscamos el BSSDI en Wigle obtenemos lo siguiente:

<img width="546" height="181" alt="imagen" src="https://github.com/user-attachments/assets/a1e205eb-d3d3-4b44-967f-4140f76b2ee2" />

Nuestro tercer flag es UnileverWiFi.

<br>

**Flag4**

La cuarta flag nos pide saber donde encontramos el email del usuario. Como vimos en la primera imagen, lo muestra en Github.

<br>


**Flag5**

Aquí pregunta donde está de vacaciones. Eso nos lo dice en su blog en Wordpress:

<img width="908" height="640" alt="imagen" src="https://github.com/user-attachments/assets/bbd1d594-d1f5-4594-a134-982d16e35856" />

Está en New York.

<br>

**Flag6**

Nos pregunta cual es la contraseña del usuario.

Esta es la flag más difícil de encontrar. He mirado muchas cosas, pero finalemente la obtenemos analizando el código fuente del worpress en HTML:

<img width="708" height="75" alt="imagen" src="https://github.com/user-attachments/assets/f77d5057-34cc-48a6-a74d-b3d0508896f7" />


<br>

 Máquina completada: OhSINT ✅

