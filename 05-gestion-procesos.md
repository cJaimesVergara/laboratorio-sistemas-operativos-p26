# Gestión de Procesos

## 1. ps - Listar Procesos

Muestra información sobre los procesos en ejecución.

```bash
ps [opciones]
```

**Opciones comunes:**
- `ps` : Procesos del usuario actual en la terminal actual
- `ps aux` : Todos los procesos con detalles
- `ps -ef` : Todos los procesos en formato completo
- `ps -u usuario` : Procesos de un usuario específico

**Ejemplos:**
```bash
$ ps
  PID TTY          TIME CMD
 1234 pts/0    00:00:00 bash
 5678 pts/0    00:00:00 ps

$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  16880  7520 ?        Ss   08:00   0:01 /sbin/init
usuario   1234  0.0  0.2  21456  9876 pts/0    Ss   09:00   0:00 -bash

$ ps aux | grep firefox
usuario   2345  5.2  8.3 2345678 654321 ?      Sl   09:30   2:45 firefox
```

**Columnas importantes:**
- **PID**: ID del proceso
- **%CPU**: Uso de CPU
- **%MEM**: Uso de memoria
- **VSZ**: Memoria virtual
- **RSS**: Memoria física (Resident Set Size)
- **STAT**: Estado del proceso

## 2. top - Monitor de Procesos en Tiempo Real

Muestra procesos en tiempo real con uso de recursos.

```bash
top [opciones]
```

**Controles interactivos en top:**
- `q`: Salir
- `k`: Matar proceso (pide PID)
- `M`: Ordenar por uso de memoria
- `P`: Ordenar por uso de CPU
- `h`: Ayuda
- `1`: Mostrar CPUs individuales

**Ejemplo:**
```bash
$ top
top - 10:30:25 up 5 days,  3:45,  2 users,  load average: 0.15, 0.20, 0.18
Tasks: 245 total,   1 running, 244 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.7 sy,  0.0 ni, 96.7 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7890.5 total,   3456.2 free,   2123.4 used,   2310.9 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5234.5 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 2345 usuario   20   0 2345678 654321 123456 S   5.2   8.3   2:45.67 firefox
```

## 3. htop - Monitor Mejorado (requiere instalación)

Similar a top pero con interfaz más intuitiva.

```bash
sudo apt install htop
htop
```

**Características:**
- Interfaz a color
- Uso del mouse
- Visualización de árboles de procesos
- Más fácil de usar que top

## 4. pgrep - Buscar Procesos por Nombre

Busca procesos por nombre y devuelve sus PIDs.

```bash
pgrep [opciones] patrón
```

**Opciones:**
- `pgrep -l` : Muestra PID y nombre
- `pgrep -u usuario` : Procesos de un usuario
- `pgrep -a` : Muestra línea de comando completa

**Ejemplos:**
```bash
$ pgrep firefox
2345

$ pgrep -l bash
1234 bash
5678 bash

$ pgrep -u usuario firefox
```

## 5. pidof - Obtener PID de un Programa

Similar a pgrep pero busca el nombre exacto.

```bash
pidof nombre_programa
```

**Ejemplo:**
```bash
$ pidof firefox
2345 6789

$ pidof -s firefox  # Solo el primer PID
2345
```

## 6. kill - Terminar Procesos

Envía señales a procesos (por defecto SIGTERM).

```bash
kill [opciones] PID
```

**Señales comunes:**
- `kill PID` o `kill -15 PID` : SIGTERM (terminación normal)
- `kill -9 PID` : SIGKILL (terminación forzada)
- `kill -1 PID` : SIGHUP (reiniciar)
- `kill -2 PID` : SIGINT (interrumpir, como Ctrl+C)

**Ejemplos:**
```bash
# Terminar proceso normalmente
$ kill 2345

# Forzar terminación
$ kill -9 2345

# Matar todos los procesos firefox
$ killall firefox

# Usando pkill (por nombre)
$ pkill firefox
$ pkill -u usuario firefox
```

## 7. killall - Matar Procesos por Nombre

Termina todos los procesos con el nombre especificado.

```bash
killall [opciones] nombre_proceso
```

**Ejemplos:**
```bash
$ killall firefox
$ killall -9 proceso_colgado
$ killall -u usuario firefox
```

## 8. nice y renice - Prioridad de Procesos

**nice** inicia un proceso con prioridad específica:
```bash
nice -n prioridad comando
```

**Valores de prioridad (niceness):**
- -20 a 19 (menor número = mayor prioridad)
- 0 es el valor por defecto
- Usuarios normales solo pueden aumentar niceness (reducir prioridad)

**renice** cambia la prioridad de un proceso en ejecución:
```bash
renice prioridad -p PID
```

**Ejemplos:**
```bash
# Iniciar con baja prioridad
$ nice -n 10 tar czf backup.tar.gz /home/

# Cambiar prioridad de proceso existente
$ renice 10 -p 2345

# Mayor prioridad (requiere sudo)
$ sudo renice -5 -p 2345
```

## 9. nohup - Ejecutar Proceso Inmune a Desconexión

Ejecuta un comando que continúa después de cerrar la terminal.

```bash
nohup comando &
```

**Ejemplo:**
```bash
$ nohup python3 script_largo.py &
[1] 3456
nohup: ignoring input and appending output to 'nohup.out'

# El proceso continúa aunque cierres la terminal
```

## 10. bg y fg - Trabajos en Background/Foreground

**bg** mueve proceso al background:
```bash
bg [%número_trabajo]
```

**fg** trae proceso al foreground:
```bash
fg [%número_trabajo]
```

**jobs** lista trabajos:
```bash
jobs
```

**Ejemplos:**
```bash
# Iniciar proceso
$ sleep 100

# Suspender con Ctrl+Z
^Z
[1]+  Stopped                 sleep 100

# Listar trabajos
$ jobs
[1]+  Stopped                 sleep 100

# Continuar en background
$ bg %1
[1]+ sleep 100 &

# Traer al foreground
$ fg %1
sleep 100
```

## 11. Operadores de Control de Procesos

### & - Ejecutar en Background
```bash
$ comando &
$ firefox &
[1] 2345
```

### && - AND Lógico
```bash
# Ejecuta comando2 solo si comando1 tiene éxito
$ comando1 && comando2
$ mkdir carpeta && cd carpeta
```

### || - OR Lógico
```bash
# Ejecuta comando2 solo si comando1 falla
$ comando1 || comando2
$ cd carpeta || mkdir carpeta
```

### ; - Ejecutar Secuencialmente
```bash
# Ejecuta ambos comandos sin importar el resultado
$ comando1 ; comando2
$ cd /tmp ; ls
```

## 12. watch - Ejecutar Comando Periódicamente

Ejecuta un comando repetidamente y muestra la salida.

```bash
watch [opciones] comando
```

**Opciones:**
- `watch -n segundos` : Intervalo de actualización
- `watch -d` : Resalta diferencias

**Ejemplos:**
```bash
# Actualizar cada 2 segundos (por defecto)
$ watch df -h

# Actualizar cada 5 segundos
$ watch -n 5 'ps aux | grep firefox'

# Monitorear archivo de log
$ watch -n 1 tail /var/log/syslog
```

## 13. time - Medir Tiempo de Ejecución

Mide cuánto tiempo tarda un comando en ejecutarse.

```bash
time comando
```

**Ejemplo:**
```bash
$ time ls -R /usr
real    0m2.543s
user    0m0.123s
sys     0m0.456s
```

## Ejercicios Prácticos

1. Usa `ps aux` para ver todos los procesos y filtra con grep
2. Ejecuta `top` y familiarízate con sus controles
3. Inicia un proceso en background con `&`
4. Usa Ctrl+Z para suspender un proceso, luego `bg` y `fg`
5. Practica matar procesos con `kill` y `killall`
6. Usa `watch` para monitorear un comando en tiempo real
7. Combina comandos: `pgrep firefox | xargs kill`

## Comandos Útiles Combinados

```bash
# Ver procesos de un usuario ordenados por uso de memoria
$ ps aux --sort=-%mem | grep usuario | head -10

# Matar todos los procesos de un programa
$ pkill -9 nombre_programa

# Monitorear uso de CPU de un proceso
$ watch -n 1 'ps aux | grep nombre_proceso'

# Ver procesos zombies
$ ps aux | grep 'Z'
```
