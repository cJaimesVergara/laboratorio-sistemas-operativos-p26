# LAB 01 — Linux incremental + Búsqueda/Filtrado/Pipelines/Procesos/Disco (COMANDOS COMENTADOS)

**Entrega:** `lab1_incremental_02XXXXX.tgz` con carpeta `evidencia/`

---

## Prerrequisito (continuidad)
Este laboratorio asume que ya practicaste en **LAB 00 y LAB 00.5**:

- Navegación y manejo de archivos
- Globbing básico (`*`, `?`)
- Redirecciones (`>`, `>>`, `2>`) y pipes (`|`)
- Resumen con `wc`, `sort`, `uniq`

Aquí subimos el nivel con **búsqueda real**, **parsing de logs**, **find más útil**, **procesos** y **disco**.

---

## Objetivo
Al final podrás:

- Buscar y filtrar información: `grep` (más opciones), `find` (tamaño, tipo)
- Transformar/extraer: `cut` (y pipelines más largos)
- Guardar evidencia sin perder pantalla: `tee`
- Procesos: `ps`, `pgrep`, `kill` (controlado)
- Sistema/Disco: `df`, `du`
- Crear una entrega reproducible: `tar`
- (Plus) Entender **códigos de salida**: `echo $?`

> **Regla:** No uses interfaz gráfica para crear/editar archivos (solo para abrir la terminal).

---

## 0) Preparación

```bash
# Crea la estructura base del laboratorio:
# -p: crea directorios padres si no existen
# {sandbox,evidencia}: expansión de llaves para crear ambas carpetas
mkdir -p ~/linux-lab/lab1/{sandbox,evidencia}

# Entra a la carpeta del laboratorio
cd ~/linux-lab/lab1

# Imprime el directorio actual (ruta completa)
pwd

# Lista todo con detalle:
# -l: formato largo
# -a: incluye ocultos
ls -la
```

**Evidencia 01**
```bash
{
  # Texto decorativo
  echo "=== Identidad ==="

  # Usuario actual
  whoami

  # Nombre del host
  hostname

  # Fecha/hora actual
  date

  # Variable HOME
  echo "HOME=$HOME"

  # $(pwd): ejecuta pwd y lo inserta
  echo "PWD=$(pwd)"

  # uname -r: versión del kernel
  echo "Kernel: $(uname -r)"

# > guarda stdout del bloque al archivo (sobrescribe)
} > evidencia/01_identidad.txt
```

---

## 1) Dataset (simular archivos reales de trabajo)

En este laboratorio vas a generar un conjunto de archivos que simula logs y documentos.

```bash
# Entra al sandbox
cd ~/linux-lab/lab1/sandbox

# Crea carpetas de trabajo
mkdir -p logs docs out tmp
```

### 1.1 Genera un log (texto)
```bash
# Here-doc (<<) para escribir un bloque al archivo app.log
# 'EOF' evita expansión de variables dentro del bloque
cat > logs/app.log << 'EOF'
2026-02-10 10:00:01 INFO  user=carlos action=login ip=10.0.0.10
2026-02-10 10:00:03 INFO  user=ana    action=login ip=10.0.0.11
2026-02-10 10:01:10 WARN  user=carlos action=upload ip=10.0.0.10 file=doc1.pdf
2026-02-10 10:02:22 ERROR user=ana    action=upload ip=10.0.0.11 code=500
2026-02-10 10:03:01 INFO  user=luis   action=logout ip=10.0.0.12
2026-02-10 10:03:44 ERROR user=carlos action=upload ip=10.0.0.10 code=403
2026-02-10 10:04:10 INFO  user=ana    action=login ip=10.0.0.11
2026-02-10 10:05:50 WARN  user=luis   action=download ip=10.0.0.12 file=manual.pdf
EOF
```

### 1.2 Genera “documentos”
```bash
# echo + > crea/sobrescribe archivos de texto con una línea
echo "Manual de usuario - versión 1" > docs/manual.txt
echo "Guía rápida - versión 1" > docs/guia.txt

# seq genera números del 1 al 120
# > redirige stdout al archivo numeros.txt
seq 1 120 > docs/numeros.txt
```

**Evidencia 02**
```bash
{
  echo "=== Estructura sandbox ==="

  # pwd muestra ruta actual
  pwd

  # lista contenido del sandbox
  ls -la

  echo
  echo "=== logs/app.log (head) ==="

  # head -n 5 muestra las primeras 5 líneas del log
  head -n 5 logs/app.log

  echo
  echo "=== docs ==="

  # lista archivos en docs
  ls -la docs

} > ~/linux-lab/lab1/evidencia/02_dataset.txt
```

---

## 2) grep: buscar y filtrar (ya con opciones útiles)

### 2.1 Buscar por palabra
```bash
# Entra al sandbox (por si cambiaste de carpeta)
cd ~/linux-lab/lab1/sandbox

# grep "ERROR" imprime las líneas que contienen ERROR
grep "ERROR" logs/app.log
```

### 2.2 Buscar sin importar mayúsculas
```bash
# -i: ignora mayúsculas/minúsculas
grep -i "warn" logs/app.log
```

### 2.3 Contar coincidencias y mostrar líneas
```bash
# -c: cuenta coincidencias (número de líneas que matchean)
grep -c "ERROR" logs/app.log

# -n: muestra número de línea + contenido
grep -n "ERROR" logs/app.log
```

### 2.4 Mostrar contexto (líneas alrededor)
```bash
# -C 1: muestra 1 línea antes y 1 después de cada match
# -n: también incluye números de línea
grep -n -C 1 "ERROR" logs/app.log
```

**Evidencia 03**
```bash
{
  echo "=== grep: ERROR (líneas) ==="
  # -n: numera líneas
  grep -n "ERROR" logs/app.log

  echo
  echo "=== grep: WARN (case-insensitive) ==="
  # -n: numera, -i: ignora mayúsculas
  grep -ni "warn" logs/app.log

  echo
  echo "=== Conteos ==="
  # echo -n no agrega salto de línea
  echo -n "ERROR: "; grep -c "ERROR" logs/app.log
  echo -n "INFO: ";  grep -c "INFO" logs/app.log

  echo
  echo "=== Contexto (-C 1) ==="
  grep -n -C 1 "ERROR" logs/app.log

} > ~/linux-lab/lab1/evidencia/03_grep_util.txt
```

---

## 3) Pipelines: extraer y resumir (parsing de log)

### 3.1 ¿Qué usuarios aparecen en el log?
Extrae `user=...`, ordénalo y deja únicos.

```bash
# grep filtra líneas con "user="
# cut corta columnas:
# -d' ' usa espacio como delimitador
# -f4 toma el campo 4 (en este log, "user=...")
# sort ordena para que uniq funcione
# uniq deja únicos
grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq
```

### 3.2 Conteo por usuario (ranking)
```bash
# uniq -c cuenta repeticiones (requiere lista ordenada)
# sort -nr ordena numéricamente en descendente
grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq -c | sort -nr
```

### 3.3 Solo errores 4xx/5xx (filtra por código)
```bash
# grep "ERROR" filtra solo líneas de error
# grep -E usa regex extendida
# "code=4|code=5" encuentra code=4xx o code=5xx (por prefijo)
grep "ERROR" logs/app.log | grep -E "code=4|code=5"
```

**Evidencia 04**
```bash
{
  echo "=== Usuarios únicos ==="
  grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq

  echo
  echo "=== Conteo por usuario (desc) ==="
  grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq -c | sort -nr

  echo
  echo "=== Errores con code=4xx/5xx ==="
  grep "ERROR" logs/app.log | grep -E "code=4|code=5"

} > ~/linux-lab/lab1/evidencia/04_pipelines_parsing.txt
```

---

## 4) tee: guardar evidencia sin perder pantalla

```bash
# mkdir -p asegura que exista la carpeta out (no falla si ya existe)
mkdir -p out

# tee escribe simultáneamente:
# - a pantalla (stdout)
# - y al archivo out/errores_encontrados.txt
grep -n "ERROR" logs/app.log | tee out/errores_encontrados.txt
```

**Evidencia 05**
```bash
{
  echo "=== tee: errores_encontrados.txt ==="
  # cat muestra lo guardado por tee
  cat out/errores_encontrados.txt
} > ~/linux-lab/lab1/evidencia/05_tee.txt
```

---

## 5) find: localizar archivos por nombre/tamaño (más práctico)

### 5.1 Buscar por nombre
```bash
# find . busca desde el directorio actual (.)
# -name "*.txt" filtra por nombre/ patrón
find . -name "*.txt"
```

### 5.2 Buscar por tamaño (archivos “pesados”)
Crea un archivo grande:
```bash
# dd copia bytes:
# if=/dev/zero: fuente de ceros (genera datos)
# of=...: archivo destino
# bs=1M: bloque de 1 MiB
# count=5: 5 bloques => ~5 MiB
# status=none: no imprime progreso
dd if=/dev/zero of=tmp/grande.bin bs=1M count=5 status=none
```

Ahora busca archivos mayores a 1MB:
```bash
# -type f: solo archivos (no directorios)
# -size +1M: mayor a 1 MiB
# -ls: imprime en formato tipo ls largo
find . -type f -size +1M -ls
```

**Evidencia 06**
```bash
{
  echo "=== find: *.txt ==="
  find . -name "*.txt"

  echo
  echo "=== find: >1MB ==="
  find . -type f -size +1M -ls

} > ~/linux-lab/lab1/evidencia/06_find.txt
```

---

## 6) Procesos: observar y controlar (sin romper nada)

### 6.1 Ver procesos del usuario
```bash
# ps lista procesos
# -u "$USER" filtra por tu usuario
# -f formato completo
# | head muestra los primeros (para no saturar)
ps -u "$USER" -f | head
```

### 6.2 Levantar un proceso “dummy” y localizarlo
```bash
# sleep 300 duerme 300 segundos
# & lo manda a background (regresa el prompt)
sleep 300 &

# $! es el PID del último proceso en background
echo "PID sleep: $!"

# pgrep busca procesos por nombre
# -a muestra PID + comando completo
pgrep -a sleep
```

### 6.3 Terminarlo (controlado)
```bash
# pgrep -n sleep obtiene el proceso sleep más reciente
# $(...) sustituye comando por su salida (PID)
# kill envía señal (por defecto SIGTERM) para terminar el proceso
kill $(pgrep -n sleep)
```

**Evidencia 07**
```bash
{
  echo "=== Procesos (ps) ==="
  ps -u "$USER" -f | head -n 15

  echo
  echo "=== pgrep sleep (antes) ==="
  # Levanta un sleep corto y captura su PID
  (sleep 60 &); echo "PID=$!"

  # Lista sleeps; || true evita que falle si no encuentra
  pgrep -a sleep || true

  echo
  echo "=== kill sleep ==="
  # 2>/dev/null descarta errores si ya no existe
  kill $(pgrep -n sleep) 2>/dev/null || true

  echo "pgrep después:"
  # Si no hay sleep, imprime OK
  pgrep -a sleep || echo "OK: no hay sleep corriendo"

} > ~/linux-lab/lab1/evidencia/07_procesos.txt
```

---

## 7) Disco: entender espacio real (df/du)

```bash
# df -h: espacio en discos/particiones (human-readable)
df -h

# du -h: tamaño por carpeta/archivo (human-readable)
# --max-depth=1: solo 1 nivel de profundidad desde el directorio actual
du -h --max-depth=1 .
```

**Evidencia 08**
```bash
{
  echo "=== df -h ==="
  df -h

  echo
  echo "=== du (sandbox) ==="
  du -h --max-depth=1 .

} > ~/linux-lab/lab1/evidencia/08_disco.txt
```

---

## 8) Códigos de salida (bonus pro)

```bash
# grep busca un texto que NO existe: normalmente no imprime nada
# luego echo $? imprime el código de salida del último comando
grep "NO_EXISTE" logs/app.log
echo $?

# grep busca un texto que sí existe: imprime matches
# y echo $? típicamente será 0 (éxito)
grep "ERROR" logs/app.log
echo $?
```

**Evidencia 09**
```bash
{
  echo "=== exit codes ==="
  # ; permite ejecutar echo aunque grep no encuentre
  grep "NO_EXISTE" logs/app.log; echo "exit=$?"
  grep "ERROR" logs/app.log;    echo "exit=$?"

} > ~/linux-lab/lab1/evidencia/09_exit_codes.txt
```

---

## 9) Reto final (integrador)

Genera un reporte llamado `out/reporte_lab1.txt` con:

1) Total de líneas del log  
2) Total de eventos por nivel (INFO/WARN/ERROR)  
3) Ranking de usuarios por apariciones  
4) Lista de archivos `.txt` encontrados con `find`  
5) Lista de errores (con `tee`) guardada en `out/errores_encontrados.txt`

```bash
{
  echo "=== REPORTE LAB1 ==="
  echo "Fecha: $(date)"
  echo

  echo "1) Total líneas log:"
  # wc -l cuenta líneas del archivo
  wc -l logs/app.log
  echo

  echo "2) Conteo por nivel:"
  # grep -c cuenta coincidencias
  # Observa que se usa " INFO " con espacios para evitar falsos positivos
  echo -n "INFO:  "; grep -c " INFO "  logs/app.log
  echo -n "WARN:  "; grep -c " WARN "  logs/app.log
  echo -n "ERROR: "; grep -c " ERROR " logs/app.log
  echo

  echo "3) Ranking usuarios:"
  grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq -c | sort -nr
  echo

  echo "4) Archivos .txt:"
  # find lista rutas de .txt desde el directorio actual
  find . -name "*.txt"

# > guarda el reporte al archivo (sobrescribe)
} > out/reporte_lab1.txt

# tee guarda errores y también los muestra en pantalla
grep -n "ERROR" logs/app.log | tee out/errores_encontrados.txt
```

**Evidencia 10**
```bash
{
  echo "=== reporte_lab1 ==="
  cat out/reporte_lab1.txt

  echo
  echo "=== errores_encontrados ==="
  cat out/errores_encontrados.txt

} > ~/linux-lab/lab1/evidencia/10_reto_final.txt
```

---

## 10) Cierre y entrega

**Resumen**
```bash
# Here-doc para crear el resumen del alumno
cat > ~/linux-lab/lab1/evidencia/11_resumen.txt << 'EOF'
Resumen (2–6 líneas)
- 3 comandos nuevos que aprendiste en este lab:
- ¿Qué hace un pipeline (|) en tus palabras?
- ¿Para qué sirve grep y find en la vida real?
- 1 error que te pasó y cómo lo resolviste:
EOF
```

**Comprimir entrega - Colocar tu número de id**
```bash
# Entra a la carpeta del lab
cd ~/linux-lab/lab1

# tar comprime la carpeta evidencia:
# -c create, -z gzip, -f nombre del archivo
tar -czf lab1_incremental_02XXXXX.tgz evidencia

# Verifica que existe
ls -la lab1_incremental_02XXXXX.tgz
```

**Entrega:** `lab1_incremental_02XXXXX.tgz`

---

## Cheatsheet mini
- Búsqueda/filtrado: `grep`, `cut`, `sort`, `uniq`, `wc`
- Archivos: `find`
- Pipes/redirecciones: `|`, `>`, `>>`, `2>`, `tee`
- Procesos: `ps`, `pgrep`, `kill`
- Disco: `df -h`, `du -h`
- Entrega: `tar -czf`
