# Información del Sistema

## 1. uname - Información del Sistema

Muestra información básica del sistema.

```bash
uname [opciones]
```

**Opciones comunes:**
- `uname -a` : Toda la información del sistema
- `uname -s` : Nombre del kernel
- `uname -r` : Versión del kernel
- `uname -m` : Arquitectura del hardware

**Ejemplos:**
```bash
$ uname -a
Linux ubuntu 5.15.0-58-generic #64-Ubuntu SMP x86_64 GNU/Linux

$ uname -r
5.15.0-58-generic
```

## 2. hostname - Nombre del Host

Muestra o establece el nombre del sistema.

```bash
hostname [opciones]
```

**Ejemplos:**
```bash
$ hostname
mi-servidor-ubuntu

$ hostname -I  # Muestra direcciones IP
192.168.1.100
```

## 3. uptime - Tiempo de Actividad

Muestra cuánto tiempo lleva el sistema encendido.

```bash
uptime
```

**Ejemplo:**
```bash
$ uptime
 10:30:25 up 5 days,  3:45,  2 users,  load average: 0.15, 0.20, 0.18
```

## 4. whoami y who - Información de Usuario

**whoami** muestra tu nombre de usuario:
```bash
$ whoami
usuario
```

**who** muestra quién está conectado:
```bash
$ who
usuario  tty7         2024-02-11 08:00 (:0)
```

**w** muestra información detallada de usuarios:
```bash
$ w
 10:30:25 up 5 days,  3:45,  2 users,  load average: 0.15, 0.20, 0.18
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
usuario  tty7     :0               08:00    3:45m  2:30   0.05s -bash
```

## 5. df - Espacio en Disco

Muestra el espacio usado y disponible en los sistemas de archivos.

```bash
df [opciones]
```

**Opciones comunes:**
- `df -h` : Formato legible (human-readable)
- `df -T` : Muestra el tipo de sistema de archivos
- `df -i` : Muestra información de inodos

**Ejemplos:**
```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   20G   28G  42% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/sda2       100G   45G   50G  48% /home

$ df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/sda1      ext4       50G   20G   28G  42% /
```

## 6. du - Uso de Disco por Directorio

Muestra el espacio usado por archivos y directorios.

```bash
du [opciones] [directorio]
```

**Opciones comunes:**
- `du -h` : Formato legible
- `du -s` : Solo muestra el total (summary)
- `du -a` : Incluye archivos individuales
- `du --max-depth=1` : Limita la profundidad

**Ejemplos:**
```bash
$ du -h
4.0K    ./carpeta1
8.0K    ./carpeta2
12K     .

$ du -sh *
4.0K    archivo.txt
8.0K    carpeta1
16K     carpeta2

$ du -h --max-depth=1 /home/usuario
```

## 7. free - Memoria RAM y Swap

Muestra el uso de memoria del sistema.

```bash
free [opciones]
```

**Opciones:**
- `free -h` : Formato legible
- `free -m` : En megabytes
- `free -g` : En gigabytes

**Ejemplo:**
```bash
$ free -h
              total        used        free      shared  buff/cache   available
Mem:          7.7G        2.3G        3.2G        234M        2.2G        4.9G
Swap:         2.0G          0B        2.0G
```

## 8. lscpu - Información del CPU

Muestra información detallada del procesador.

```bash
lscpu
```

**Ejemplo:**
```bash
$ lscpu
Architecture:        x86_64
CPU(s):              4
Model name:          Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz
Thread(s) per core:  2
```

## 9. lsblk - Listar Dispositivos de Bloque

Muestra información sobre dispositivos de almacenamiento.

```bash
lsblk [opciones]
```

**Ejemplo:**
```bash
$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 238.5G  0 disk 
├─sda1   8:1    0    50G  0 part /
└─sda2   8:2    0   100G  0 part /home
```

## 10. lsusb y lspci - Dispositivos USB y PCI

**lsusb** lista dispositivos USB:
```bash
$ lsusb
Bus 002 Device 003: ID 0bda:0129 Realtek Semiconductor Corp.
Bus 001 Device 002: ID 8087:0024 Intel Corp.
```

**lspci** lista dispositivos PCI:
```bash
$ lspci
00:00.0 Host bridge: Intel Corporation
00:02.0 VGA compatible controller: Intel Corporation
00:14.0 USB controller: Intel Corporation
```

## 11. dmidecode - Información del Hardware

Muestra información detallada del hardware (requiere sudo).

```bash
sudo dmidecode [opciones]
```

**Ejemplos:**
```bash
$ sudo dmidecode -t memory
$ sudo dmidecode -t processor
$ sudo dmidecode -t bios
```

## 12. date - Fecha y Hora

Muestra o establece la fecha y hora del sistema.

```bash
date [opciones] [formato]
```

**Ejemplos:**
```bash
$ date
Tue Feb 11 10:30:25 AM -05 2024

$ date "+%Y-%m-%d"
2024-02-11

$ date "+%Y-%m-%d %H:%M:%S"
2024-02-11 10:30:25
```

## 13. cal - Calendario

Muestra un calendario.

```bash
cal [mes] [año]
```

**Ejemplos:**
```bash
$ cal
    February 2024
Su Mo Tu We Th Fr Sa
             1  2  3
 4  5  6  7  8  9 10
11 12 13 14 15 16 17
18 19 20 21 22 23 24
25 26 27 28 29

$ cal 12 2024  # Diciembre 2024
```

## Ejercicios Prácticos

1. Ejecuta `uname -a` y analiza la salida
2. Verifica el espacio disponible en disco con `df -h`
3. Identifica qué directorios ocupan más espacio con `du`
4. Revisa el uso de memoria con `free -h`
5. Investiga las características de tu CPU con `lscpu`
6. Combina comandos: `df -h | grep /dev/sda`
