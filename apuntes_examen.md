# 📑 CHULETA DEFINITIVA: EXAMEN PRÁCTICO DE ASO

Este documento agrupa la estructura de comandos, archivos críticos y los patrones exactos de preguntas resueltas extraídos de los exámenes reales (**2020, 2021, 2022 y 2024**) para las tres familias: **Linux** (Devuan/Void), **BSD** (FreeBSD/NetBSD/DragonFly) y **Solaris** (OpenIndiana).


> Para acceder a la tty de una máquina: Ctl + Fn + F1  || Ctl + Alt + Fn + F5 


**Acceder a una máquina en la que no sabemos usuario ni contraseña**


ANTES DE NADA: Comprobamos que tipo de sistema es:
```bash
uname -a
cat /etc/os-release
hostnamectl
```

Entonces se usa el modo recovery desde GRUB:

Reinicia la VM

En GRUB selecciona Devuan

Pulsa e

Busca la línea que empieza por linux

Al final añade:
```bash
init=/bin/bash
```

Arranca con Ctrl + X

Esto te da una shell como root sin contraseña.

Luego:
```bash
mount -o remount,rw /
passwd usuario
sync
reboot
```

---

# TEMA 1: INTRODUCCIÓN (DISCOS, PARTICIONES Y SWAP)

## 💻 Comandos de Diagnóstico por Familia

- **Linux:** `fdisk -l` o `cat /proc/partitions`
- **BSD:** `gpart show` o `disklabel <dispositivo>`
  - Ejemplo:
    ```bash
    disklabel wd0
    disklabel ad0s4
    ```
- **Solaris:** `format`
  - Herramienta interactiva para ver discos, particiones y *slices*.

---

## 🔍 Ficheros de Configuración Críticos

- `/etc/fstab` → Montajes automáticos en **Linux** y **BSD**.
- `/etc/vfstab` → Equivalente de montaje automático en **Solaris**.

---

## 🎯 Joyas del Examen (Patrones que siempre caen)

### 1. Identificar y medir el espacio SWAP

- **Linux:** `swapon --show` o `cat /proc/swaps`
- **BSD:** `swapinfo -k`
  - Muestra el tamaño en bloques de 1K.
- **Solaris:** `swap -l`
  - Suele mostrar si es un volumen ZFS como:
    ```text
    /dev/zvol/dsk/rpool/swap
    ```

#### Pregunta típica

> "¿Es fichero o partición? Da el nombre y tamaño"

- Si la ruta apunta a `/var/swap` → es **fichero**
- Si apunta a `/dev/...` → es **partición/dispositivo**

---

### 2. Calcular el espacio máximo para una nueva partición (NetBSD / FreeBSD)

#### Método

Ejecuta:

```bash
disklabel wd0
```

Busca:

- La partición `c`
  - Representa la totalidad del disco o slice asignado al SO.
- Su tamaño total (`size`).
- La última partición de datos usada (`e`, `g`, etc.).

Después calcula la diferencia entre offsets y límites para saber cuántos sectores quedan libres.

---

### 3. Montar almacenamiento temporal (Imágenes ISO) sin reiniciar

#### BSD

Usa dispositivos de memoria (`md`):

```bash
mdconfig -a -t vnode -f /data/ASO/imagen.iso
# Devuelve un dispositivo, ej: md1

mount -t cd9660 /dev/md1 /mnt
```

#### Solaris

Usa *loopback* (`lofiadm`):

```bash
lofiadm -a /data/ASO/imagen.iso
# Devuelve un dispositivo, ej: /dev/lofi/1

mount -F hsfs /dev/lofi/1 /mnt
```

---

### 4. Montaje persistente de sistemas Windows/MSDOS

#### BSD (`/etc/fstab`)

```text
/dev/ada0s3  /data  msdosfs  rw  0  0
```

#### Solaris (`/etc/vfstab`)

```text
/dev/dsk/c1d0p3  /dev/rdsk/c1d0p3  /data  pcfs  yes  -
```

---

# TEMA 2: CARGADORES DE ARRANQUE (GRUB Y REFIND)

## 🔍 Ficheros de Configuración Críticos

- `/boot/grub/grub.cfg`
  - Configuración generada automáticamente.
  - ❌ No editar directamente.
- `/etc/grub.d/40_custom`
  - Lugar correcto para entradas persistentes.
- `/etc/lilo.conf`
  - Configuración de LILO (Linux antiguos).

---

## 🎯 Joyas del Examen

### 1. Añadir un SO al menú de GRUB

Editar:

```text
/etc/grub.d/40_custom
```

Ejemplo:

```text
menuentry "FreeBSD" {
    set root=(hd0,msdos4)
    chainloader +1
}
```

Aplicar cambios:

```bash
update-grub
```

o

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

### 2. Restricciones de Hardware: ¿Se puede instalar rEFInd?

#### Pregunta típica

> "¿Podríamos poner rEFInd como cargador y desde él seleccionar qué operativo arranca?"

#### Respuesta

❌ No, si la máquina virtual usa BIOS clásico.

✅ rEFInd funciona únicamente en plataformas **EFI/UEFI**.

---

# TEMA 3: ADMINISTRACIÓN BÁSICA DE USUARIOS Y SEGURIDAD PAM

## 💻 Comandos esenciales

### Linux

```bash
useradd -m <user>
usermod
groupadd
```

### BSD

```bash
pw useradd -m -n <user>
pw usermod
vipw
```

### Solaris

```bash
useradd -m <user>
usermod -R root <user>
```

---

## 🔍 Ficheros Críticos

- `/etc/pam.d/su`
- `/etc/pam.d/login`
- `/etc/pam.d/lightdm`
- `/etc/group`

---

## 🎯 Joyas del Examen

### 1. Averiguar quién puede hacerse root

#### Caso `pam_wheel.so`

Si aparece:

```text
auth requisite pam_wheel.so group=bin root_only
```

➡ Solo usuarios del grupo `bin` pueden usar `su`.

Mirar:

```text
/etc/group
```

---

#### Caso `pam_succeed_if.so`

```text
auth requisite pam_succeed_if.so uid = 1501
```

➡ Solo el usuario con UID 1501 puede escalar privilegios.

Buscar en:

```text
/etc/passwd
```

---

### 2. Reparar logins automáticos sin contraseña

#### Problema

El sistema entra directamente sin pedir clave.

#### Solución

Editar:

```text
/etc/pam.d/lightdm
```

Buscar líneas `sufficient` asociadas a:

- `pam_shells.so`
- `pam_permit.so`

Comentarlas o eliminarlas para forzar `pam_unix.so`.

---

### 3. Resolver usuarios bloqueados

#### Shell inválida

```bash
usermod -s /bin/bash usuario
```

O añadir shell a:

```text
/etc/shells
```

---

#### Cuenta bloqueada

```bash
passwd -u usuario
```

En BSD avanzado revisar con:

```bash
vipw
```

---

#### Cuenta caducada

Modificar con:

```bash
usermod -e
```

o comandos `pw` en BSD.

---

### 4. Creación masiva de usuarios

#### Linux (Debian/Devuan)

```bash
for I in $(seq -w 1000 2999); do
    useradd -m -p $(mkpasswd --method=yescrypt password$I) user$I
done
```

---

#### BSD (FreeBSD/DragonFly)

```bash
for I in $(seq -w 1000 2999); do
    echo "password$I" | pw useradd -m -s /usr/local/bin/bash -h 0 -n user$I
done
```

---

#### Solaris (OpenIndiana)

```bash
for I in $(seq -w 1000 2999); do
    useradd -m user$I
    /root/pass user$I
done
```

---

# TEMA 4: PROCESOS Y PAQUETES DE SOFTWARE

## 💻 Tabla de paquetería

| Sistema | Ver repositorio | Instalar | Ver paquetes |
|---|---|---|---|
| Linux (Debian/Devuan) | `cat /etc/apt/sources.list` | `apt install` | `dpkg --list \| wc -l` |
| Linux (Void) | `/share/xbps.d/` | `xbps-install` | `xbps-query -l \| wc -l` |
| BSD | `/usr/local/etc/pkg/repos` | `pkg install` | `pkg info \| wc -l` |
| Solaris | `pkg publisher` | `pkg install` | `pkg list \| wc -l` |

---

## 🎯 Joyas del Examen

### ¿Instalado por Ports o por paquetes?

#### Pregunta

> "¿El programa X se instaló desde ports o desde pkg?"

#### Respuesta

Ir al árbol de ports:

```text
/usr/ports/games/xbill
```

- Si existe `work/` → compilado desde ports.
- Si no existe → instalado con `pkg install`.

Los fuentes descargados quedan en:

```text
/usr/ports/distfiles
```

---

# TEMA 5: AUTOMATIZACIÓN DE TAREAS (ROTACIÓN DE LOGS)

## 🔍 Herramientas

### BSD

- `newsyslog`
- Configuración:
  ```text
  /etc/newsyslog.conf
  ```

### Solaris

- `logadm`
- Configuración:
  ```text
  /etc/logadm.conf
  ```

---

## 🎯 Joyas del Examen

### Rotar logs cada 10 minutos

---

### BSD (`newsyslog`)

Archivo:

```text
/etc/rotacion-auth.conf
```

Contenido:

```text
# logfile          owner:group    mode count size time flags
/var/log/auth      root:wheel     600  5     * * Z
```

Cron:

```text
*/10 * * * * newsyslog -f /etc/rotacion-auth.conf
```

---

### Solaris (`logadm`)

Cron:

```text
10 15 * * * logadm -C8 -a 'kill -HUP `cat /var/run/syslog.pid`' /var/log/syslog
```

---

# TEMA 6: REDES (IP ESTÁTICA Y WRAPPERS)

# 💻 Configuración de IP persistente

## 1. Linux (Devuan)

Archivo:

```text
/etc/network/interfaces
```

Alias:

```text
auto eth1:1

iface eth1:1 inet static
    address 192.168.22.101/24
```

---

## 2. BSD

### FreeBSD / DragonFly

Archivo:

```text
/etc/rc.conf
```

Configuración:

```text
ifconfig_em1="inet 192.168.20.101 netmask 255.255.255.0"
ifconfig_em1_alias0="inet 192.168.22.101 netmask 255.255.255.0"
```

---

### NetBSD

Archivo:

```text
/etc/ifconfig.wm1
```

Configuración:

```text
inet 192.168.10.101 netmask 255.255.255.0
inet alias 192.168.15.101 netmask 255.255.255.0
```

---

## 3. Solaris (OpenIndiana)

Usa `ipadm`:

```bash
ipadm create-addr -T static -a 192.168.10.101 e1000g1/v4address

ipadm create-addr -T static -a 192.168.11.101 e1000g1/v4alias
```

---

# 🔍 TCP Wrappers e inetd

- `/etc/hosts.allow`
- `/etc/hosts.deny`
- `/etc/inetd.conf`

---

## 🎯 Joyas del Examen

### 1. Restringir acceso a telnet

#### `/etc/hosts.allow`

```text
in.telnetd: 192.168.10. : allow
```

#### `/etc/hosts.deny`

```text
in.telnetd: ALL : deny
```

---

### Solaris: activar wrappers

```bash
inetadm -m telnet tcp_wrappers=TRUE
```

---

### 2. "Tengo ALL:ALL pero sigue entrando"

#### Causa

`inetd` llama directamente a `telnetd` sin pasar por `tcpd`.

#### Incorrecto

```text
telnet stream tcp ... /usr/sbin/telnetd in.telnetd
```

#### Correcto

```text
telnet stream tcp ... /usr/sbin/tcpd in.telnetd
```

---

### 3. Problema del Default Router

#### Síntoma

DNS funciona pero no hay salida a internet.

#### Causa

Falta gateway por defecto.

#### BSD (`/etc/rc.conf`)

```text
defaultrouter="10.0.2.2"
```