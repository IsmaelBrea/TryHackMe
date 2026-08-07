
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


## Escalate 




## Cracking




## Find flags!



 Máquina completada: Blue ✅

