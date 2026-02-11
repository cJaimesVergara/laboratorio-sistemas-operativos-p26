# LAB 01 — Linux incremental + Búsqueda/Filtrado/Pipelines/Procesos/Disco

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
mkdir -p ~/linux-lab/lab1/{sandbox,evidencia}
cd ~/linux-lab/lab1
pwd
ls -la
```

**Evidencia 01**
```bash
{
  echo "=== Identidad ==="
  whoami
  hostname
  date
  echo "HOME=$HOME"
  echo "PWD=$(pwd)"
  echo "Kernel: $(uname -r)"
} > evidencia/01_identidad.txt
```

---

## 1) Dataset (simular archivos reales de trabajo)

En este laboratorio vas a generar un conjunto de archivos que simula logs y documentos.

```bash
cd ~/linux-lab/lab1/sandbox
mkdir -p logs docs out tmp
```

### 1.1 Genera un log (texto)
```bash
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
echo "Manual de usuario - versión 1" > docs/manual.txt
echo "Guía rápida - versión 1" > docs/guia.txt
seq 1 120 > docs/numeros.txt
```

**Evidencia 02**
```bash
{
  echo "=== Estructura sandbox ==="
  pwd
  ls -la
  echo
  echo "=== logs/app.log (head) ==="
  head -n 5 logs/app.log
  echo
  echo "=== docs ==="
  ls -la docs
} > ~/linux-lab/lab1/evidencia/02_dataset.txt
```

---

## 2) grep: buscar y filtrar (ya con opciones útiles)

### 2.1 Buscar por palabra
```bash
cd ~/linux-lab/lab1/sandbox
grep "ERROR" logs/app.log
```

### 2.2 Buscar sin importar mayúsculas
```bash
grep -i "warn" logs/app.log
```

### 2.3 Contar coincidencias y mostrar líneas
```bash
grep -c "ERROR" logs/app.log
grep -n "ERROR" logs/app.log
```

### 2.4 Mostrar contexto (líneas alrededor)
```bash
grep -n -C 1 "ERROR" logs/app.log
```

**Evidencia 03**
```bash
{
  echo "=== grep: ERROR (líneas) ==="
  grep -n "ERROR" logs/app.log
  echo
  echo "=== grep: WARN (case-insensitive) ==="
  grep -ni "warn" logs/app.log
  echo
  echo "=== Conteos ==="
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
grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq
```

### 3.2 Conteo por usuario (ranking)
```bash
grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq -c | sort -nr
```

### 3.3 Solo errores 4xx/5xx (filtra por código)
```bash
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
mkdir -p out
grep -n "ERROR" logs/app.log | tee out/errores_encontrados.txt
```

**Evidencia 05**
```bash
{
  echo "=== tee: errores_encontrados.txt ==="
  cat out/errores_encontrados.txt
} > ~/linux-lab/lab1/evidencia/05_tee.txt
```

---

## 5) find: localizar archivos por nombre/tamaño (más práctico)

### 5.1 Buscar por nombre
```bash
find . -name "*.txt"
```

### 5.2 Buscar por tamaño (archivos “pesados”)
Crea un archivo grande:
```bash
dd if=/dev/zero of=tmp/grande.bin bs=1M count=5 status=none
```

Ahora busca archivos mayores a 1MB:
```bash
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
ps -u "$USER" -f | head
```

### 6.2 Levantar un proceso “dummy” y localizarlo
```bash
sleep 300 &
echo "PID sleep: $!"
pgrep -a sleep
```

### 6.3 Terminarlo (controlado)
```bash
kill $(pgrep -n sleep)
```

**Evidencia 07**
```bash
{
  echo "=== Procesos (ps) ==="
  ps -u "$USER" -f | head -n 15
  echo
  echo "=== pgrep sleep (antes) ==="
  (sleep 60 &); echo "PID=$!"
  pgrep -a sleep || true
  echo
  echo "=== kill sleep ==="
  kill $(pgrep -n sleep) 2>/dev/null || true
  echo "pgrep después:"
  pgrep -a sleep || echo "OK: no hay sleep corriendo"
} > ~/linux-lab/lab1/evidencia/07_procesos.txt
```

---

## 7) Disco: entender espacio real (df/du)

```bash
df -h
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
grep "NO_EXISTE" logs/app.log
echo $?

grep "ERROR" logs/app.log
echo $?
```

**Evidencia 09**
```bash
{
  echo "=== exit codes ==="
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
  wc -l logs/app.log
  echo

  echo "2) Conteo por nivel:"
  echo -n "INFO:  "; grep -c " INFO "  logs/app.log
  echo -n "WARN:  "; grep -c " WARN "  logs/app.log
  echo -n "ERROR: "; grep -c " ERROR " logs/app.log
  echo

  echo "3) Ranking usuarios:"
  grep "user=" logs/app.log | cut -d' ' -f4 | sort | uniq -c | sort -nr
  echo

  echo "4) Archivos .txt:"
  find . -name "*.txt"
} > out/reporte_lab1.txt

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
cd ~/linux-lab/lab1
tar -czf lab1_incremental_02XXXXX.tgz evidencia
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
