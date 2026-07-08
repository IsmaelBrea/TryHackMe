# TryHackMe

Writeups de las máquinas resueltas de TryHackMe

Las máquinas de TryHackMe tiene dos formas básicas de acceder (con el plan básico gratis) a la máquina víctima. Una de ellas es con una máquina atacante con escritorio remoto que nos proporciona la plataforma, pero con una duración de una hora por día, por lo que bajo mi punto de vista **no es recomendable**. La otra forma es acceder a la máquina mediante tu propio host o una VM en tu host como puede ser Kali Linux que sería lo ideal. Para acceder a la red de la máquina tenemos que descargarnos un archivo .ovpn que nos proporciona la plataforma de TryHackMe a cada usuario. Luego desde nuestra máquina atacante en mi caso Kali tendremos que activar el túnel Ovpn para poder acceder a la misma red donde está la máquina. Usaremos el siguiente comando:
```bash
sudo openvpn name.ovpn
```



1-Pickle Rick - nivel fácil (escaneo, eumeración de puertos y enumeración web y uso de comandos Linux).

2-Basic Pentesting - nivel fácil (fuerza bruta, cracking de hashes, enumeración de servicios, enumeración linux)
