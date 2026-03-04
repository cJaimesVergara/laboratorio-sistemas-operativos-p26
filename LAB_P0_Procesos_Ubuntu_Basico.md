# LAB P0 — Primer acercamiento a procesos (Ubuntu CLI)

**Tema:** procesos en Linux, identidad, estado, señales, `/proc` y FDs

---

## Objetivo general

Que el alumno entienda **qué es un proceso** y qué información básica expone el sistema:

- **Identidad:** PID y PPID
- **Estado:** `STAT` (ps) y `State` (proc)
- **“Interior” básico:** `/proc/<PID>/status`, `/proc/<PID>/cmdline`
- **Variables de entorno (vista parcial):** `/proc/<PID>/environ`
- **Descriptores de archivo:** `/proc/<PID>/fd` (stdin/stdout/stderr)
- **Señales básicas:** `STOP` / `CONT` / `TERM` / `KILL`
- **Memoria (vista simple):** `/proc/<PID>/statm` (sin entrar a mapeos)

---

## Entregables (evidencia)

En `~/lab_p0_basico/evidencia/`:

- `salida_programa.txt`
- `pid.txt`
- `B1_ps.txt`
- `C1_status.txt`
- `C2_cmdline.txt`
- `C3_environ.txt`
- `D1_statm.txt`
- `E1_fd.txt`
- `F1_stop_cont.txt`
- `G1_fin.txt`

---

# 0) Preparación

**Propósito:** Crear el espacio de trabajo y una carpeta de evidencias.  
**Se espera obtener:** La ruta `~/lab_p0_basico/evidencia` lista para guardar archivos `.txt`.

```bash
mkdir -p ~/lab_p0_basico/evidencia            # Crea carpeta del lab y subcarpeta de evidencias (si ya existen, no pasa nada)
cd ~/lab_p0_basico                             # Entra al directorio del laboratorio
pwd                                            # Confirma en qué ruta estás (útil para evidencias)
ls -la                                         # Lista contenido actual (debe verse evidencia/)
```

---

# A) Crear un proceso sencillo (programa en C)

## A1) Crear el archivo `hola_proceso.c`

**Propósito:** Tener un programa mínimo que imprima PID/PPID y se mantenga vivo para inspeccionarlo.  
**Se espera obtener:** Un archivo `hola_proceso.c` con el código fuente en C.

```bash
cat > hola_proceso.c <<'EOF'                   # Crea el archivo hola_proceso.c pegando el contenido entre EOF...EOF
#include <stdio.h>                             # Librería estándar de entrada/salida (printf)
#include <unistd.h>                            # Librerías UNIX (getpid, getppid, sleep)

int main(void) {                               # Función principal del programa
    printf("Hola, soy un proceso.\n");         # Imprime un mensaje
    printf("PID: %d\n", getpid());             # Imprime el PID (identificador del proceso)
    printf("PPID: %d\n", getppid());           # Imprime el PPID (PID del proceso padre)

    printf("Voy a dormir 120 segundos. Inspeccióname con /proc.\n"); // Mantiene vivo el proceso para inspeccionarlo
    
    fflush(stdout);
    # Fuerza a que la salida se escriba (evita buffer cuando rediriges, asegúrate de que lo impreso ya esté físicamente escrito (en terminal/archivo/pipe) antes de seguir”)

    sleep(120);                                # Duerme 120s para dar tiempo de inspección
    return 0;                                  # Termina el programa con código 0 (éxito)
}
EOF                                            # Fin del contenido del archivo
```

---

## A2) Compilar el programa

**Propósito:** Convertir el código fuente en un ejecutable.  
**Se espera obtener:** Un archivo ejecutable llamado `hola_proceso`.

```bash
gcc hola_proceso.c -o hola_proceso             # Compila el C y produce el ejecutable hola_proceso
echo "Compilación exit code: $?"               # Muestra 0 si compiló bien; distinto de 0 si hubo error
ls -la hola_proceso                            # Verifica que el binario exista y tenga permisos de ejecución
```

> Si `gcc` no existe: `sudo apt update && sudo apt install build-essential`

---

## A3) Ejecutar en segundo plano, redirigir salida, guardar PID

**Propósito:** Crear un proceso real (instancia en ejecución) y conservar su PID para inspección.  
**Se espera obtener:**  
- Un proceso corriendo en background  
- Un PID guardado en variable `PID` y en `evidencia/pid.txt`  
- Salida del programa en `evidencia/salida_programa.txt`

```bash
./hola_proceso > evidencia/salida_programa.txt &  # Ejecuta, manda stdout a archivo, y lo pone en background
PID=$!                                            # Guarda el PID del último proceso lanzado al background
echo "PID=$PID" | tee evidencia/pid.txt            # Muestra y guarda el PID para usarlo en los siguientes pasos
```

---

# B) Identidad del proceso con `ps`

## B1) Ver PID/PPID/STAT/TTY/CMD

**Propósito:** Observar “desde afuera” la identidad y estado del proceso (lo que ve el sistema).  
**Se espera obtener:** Un registro con PID, PPID, estado (`STAT`), terminal y comando.

```bash
ps -o pid,ppid,stat,tty,cmd -p "$PID" | tee evidencia/B1_ps.txt  # Muestra columnas clave y guarda evidencia
```

---

# C) Explorar `/proc/<PID>` (básico)

> `/proc` es un pseudo-sistema de archivos: el kernel genera estos archivos para exponer info de procesos.

## C1) `status` (estructura general)

**Propósito:** Ver metadatos del proceso (estado, padre, hilos, memoria básica).  
**Se espera obtener:** Un extracto de `/proc/$PID/status` con datos como `Name`, `State`, `Pid`, `PPid`, `Threads`.

```bash
echo "=== /proc/$PID/status (primeras 30 líneas) ===" | tee evidencia/C1_status.txt  # Encabezado en evidencia
head -n 30 /proc/"$PID"/status | tee -a evidencia/C1_status.txt                      # Guarda las primeras 30 líneas
```

---

## C2) `cmdline` (argv separado por NUL `\0`)

**Propósito:** Leer el comando/argumentos reales (`argv [argument vector]`) tal como los guarda el kernel.  
**Se espera obtener:** Una “línea de comando” legible, reconstruida a partir de separadores NUL.

```bash
echo "=== /proc/$PID/cmdline ===" | tee evidencia/C2_cmdline.txt       # Encabezado
tr '\0' ' ' < /proc/"$PID"/cmdline | tee -a evidencia/C2_cmdline.txt  # Convierte separadores NUL a espacios y guarda
echo "" | tee -a evidencia/C2_cmdline.txt                               # Agrega salto de línea final (estético)
```

---

## C3) `environ` (variables de entorno, vista parcial)

**Propósito:** Comprobar que un proceso tiene un conjunto de variables de entorno heredadas/modificadas.  
**Se espera obtener:** Algunas variables tipo `PATH=...`, `LANG=...` (solo una parte por seguridad/volumen).

```bash
echo "=== /proc/$PID/environ (primeros 200 bytes) ===" | tee evidencia/C3_environ.txt  # Encabezado
head -c 200 /proc/"$PID"/environ | tr '\0' '\n' | tee -a evidencia/C3_environ.txt    # Muestra un pedazo, cambia NUL por saltos de línea
```

```bash
#Agrego variable de entorno
export MI_VAR="hola"
echo "$MI_VAR"
```


---

# D) Memoria (vista simple, sin mapas)

## D1) `statm` (memoria en números)

**Propósito:** Ver que el proceso tiene métricas de memoria sin entrar a direcciones ni mapeos.  
**Se espera obtener:** Una línea de números (páginas). No se interpreta a fondo aún; solo se observa que existe.

```bash
echo "=== /proc/$PID/statm ===" | tee evidencia/D1_statm.txt           # Encabezado
cat /proc/"$PID"/statm | tee -a evidencia/D1_statm.txt                # Imprime métricas simples de memoria (en páginas)
```

> Nota didáctica: `statm` da conteos en **páginas**. Más adelante se puede convertir a bytes multiplicando por el tamaño de página.

```bash
size resident shared text lib data dt

size: total de memoria virtual del proceso (todo lo que “cree” que tiene mapeado).

resident: cuántas páginas están realmente en RAM (lo que sí está residente).

shared: páginas que pueden estar compartidas (ej. librerías compartidas).

text: páginas usadas por código (segmento de instrucciones).

lib: hoy suele ser 0 o no muy útil en kernels modernos.

data: páginas de datos (heap + data segment; donde vive malloc, variables globales, etc.).

dt: “dirty pages” (histórico; hoy suele no usarse / 0 en muchos sistemas).

Nota: Linux ha cambiado detalles a lo largo del tiempo; lo importante didácticamente aquí es virtual vs residente.

Porque el kernel cuenta memoria en páginas de memoria. En x86_64 lo común es que una página sea de 4096 bytes (4 KiB), pero se confirma así:

getconf PAGESIZE

```



---

# E) Descriptores de archivo (FDs) del proceso

## E1) `/proc/<PID>/fd`

**Propósito:** Confirmar que stdin/stdout/stderr son FDs (0/1/2) y ver a qué apuntan.  
**Se espera obtener:** Un listado donde, por ejemplo, `1` apunte al archivo `evidencia/salida_programa.txt` (por la redirección).

```bash
echo "=== /proc/$PID/fd (symlinks) ===" | tee evidencia/E1_fd.txt      # Encabezado
ls -l /proc/"$PID"/fd | tee -a evidencia/E1_fd.txt                     # Lista enlaces simbólicos de FDs
```

Los FDs básicos: 0, 1, 2

Todo proceso normalmente inicia con estos 3 ya abiertos:

0 = stdin (entrada estándar)

1 = stdout (salida estándar)

2 = stderr (salida de error)

Ejemplo:

Cuando haces cat archivo.txt, cat lee de FD 0 (o del archivo que le pasas).

Cuando haces echo hola, se escribe a FD 1.

Errores típicamente van a FD 2.

---

# F) Señales: detener y reanudar (STOP/CONT)

## F1) STOP y CONT con evidencia del estado

**Propósito:** Ver que las señales cambian el estado del proceso sin matarlo.  
**Se espera obtener:** `STAT` cambiando típicamente a `T` (stopped) tras `STOP`, y regresando tras `CONT`.

```bash
echo "=== STOP ===" | tee evidencia/F1_stop_cont.txt                   # Encabezado STOP
kill -STOP "$PID"                                                      # Envía señal STOP: pausa el proceso
ps -o pid,stat,cmd -p "$PID" | tee -a evidencia/F1_stop_cont.txt       # Verifica STAT (debería mostrar T)

echo "=== CONT ===" | tee -a evidencia/F1_stop_cont.txt                # Encabezado CONT
kill -CONT "$PID"                                                      # Envía señal CONT: reanuda el proceso
ps -o pid,stat,cmd -p "$PID" | tee -a evidencia/F1_stop_cont.txt       # Verifica STAT (debe salir de T)
```

---

# G) Cierre: terminar el proceso (TERM, luego KILL si hace falta)

## G1) Terminar y dejar evidencia

**Propósito:** Finalizar el proceso limpiamente y evitar dejar procesos colgados.  
**Se espera obtener:** El proceso ya no aparece en `ps` y se genera `evidencia/G1_fin.txt`.

```bash
kill -TERM "$PID" 2>/dev/null                                          # Intenta terminar amablemente (TERM)
sleep 1                                                                # Da tiempo a que termine
ps -p "$PID" >/dev/null && kill -KILL "$PID" 2>/dev/null               # Si sigue vivo, lo mata a la fuerza (KILL)
echo "Proceso terminado: $PID" | tee evidencia/G1_fin.txt               # Evidencia final
```

---


## Nota rápida (muy importante)

Si el proceso termina antes de inspeccionarlo (por ejemplo, se acabó el `sleep`), entonces `/proc/<PID>/...` deja de existir y verás:

- `No such file or directory`

En ese caso, relanza el proceso (paso **A3**) y repite.

---


# `kill` en Linux: opciones (señales) y banderas útiles

> Nota: `kill` no significa “matar” necesariamente. Significa **enviar una señal** a un proceso.

---

## Señales más comunes (las que más usarás)

### `TERM` (15) — terminar “amable”
Permite que el proceso haga limpieza (cerrar archivos, guardar estado, etc.).

```bash
kill -TERM PID
# equivalente:
kill PID
```

---

### `KILL` (9) — matar a la fuerza
No se puede capturar ni ignorar. Úsalo solo si `TERM` no funcionó.

```bash
kill -KILL PID
# equivalente:
kill -9 PID
```

---

### `STOP` (19) — pausar el proceso
Pausa la ejecución (no lo mata). No se puede ignorar.

```bash
kill -STOP PID
```

---

### `CONT` (18) — reanudar el proceso
Reanuda un proceso pausado con `STOP`.

```bash
kill -CONT PID
```

---

### `HUP` (1) — “hangup” / recarga
Muchos servicios/daemons la usan para **recargar configuración** (depende del programa).

```bash
kill -HUP PID
```

---

### `INT` (2) — interrupción (como Ctrl+C)
Es la señal que normalmente envía `Ctrl+C` en una terminal.

```bash
kill -INT PID
```

---

## Ver todas las señales disponibles

```bash
kill -l
# o:
kill -L
```

---

## Usar señales por número

```bash
kill -9 PID     # KILL
kill -15 PID    # TERM
```

---

## “Opciones” (banderas) comunes del comando `kill`

- **Listar señales**:
  ```bash
  kill -l
  kill -L
  ```

- **Enviar señal a un job de tu shell** (si estás usando jobs, por ejemplo `%1`):
  ```bash
  kill %1
  ```

---

## Recordatorio clave

- `kill` = **enviar una señal**
- Algunas señales **no terminan** el proceso (`STOP`, `CONT`)
- Flujo recomendado para “terminar”:
  1) `kill PID` (TERM)
  2) si no responde: `kill -KILL PID` (KILL)

---
