
# Máquina Pickle Rick 

Dificultad -> Muy Fácil

Enlace a la máquina -> [Neighbour](https://tryhackme.com/room/neighbour/)

El objetivo de la máquina es encontrar 1 flag.

## Despliegue del laboratorio

Desplegar VPN y comprobar la interfaz tun0 con `ip a`.


Es una máquina muy breve y fácil. Según el resultado no hace falta ni reconocimiento, basta con pegar la IP en el navegador y operar a partir de ahí: http://IP

Bien, una vez accedemos a la web tenemos un panel de login. La máquina con recomienda ver el código fuente (Ctrl+U). Por tanto será lo primero que hagamos. Ahí tenemos cierta información, principalmente esta línea:
```html
<!-- use guest:guest credentials until registration is fixed. "admin" user account is off limits!!!!! -->
```

Por tanto, tal y como nos indica ahí y nos registramos con guest:guest. 
<img width="1753" height="304" alt="imagen" src="https://github.com/user-attachments/assets/809e648e-8f0f-4a0d-a824-bb5041279f6f" />

<br>


Una vez registrados, a nivel visual no hay nada relevante. Por tanto volvemos a mirar el código fuente (Ctrl+U). Ahí encontramos esto ahora:
```html
<!-- admin account could be vulnerable, need to update -->
```

Por tanto tenemos que probar un IDOR tal como indica el enunciado con el usuario admin. Como vemos en la imagen anterior, con guest sale arriba esto: http://10.130.171.231/profile.php?user=guest. Si cambiamos aquí guest por admin, tendremos acceso a su cuenta debido a una mala configuración. Ya como admin no sale la flag en la pantalla:

<img width="1870" height="341" alt="imagen" src="https://github.com/user-attachments/assets/d5bc495d-bee3-44bb-86f1-5034b427f4fd" />

<br>

 Máquina completada: Neighbour ✅

