# LAB 00.5 — Linux reforzamiento + Redirecciones/Pipelines/Globbing

**Entrega:** `lab0_5_refuerzo_02XXXXX.tgz` con carpeta `evidencia/`

---

## Objetivo
Al final podrás reforzar lo del LAB 00 y además:

- Globbing y patrones: `*`, `?`, `[]` (selección de archivos)
- Crear estructura más rápido: `mkdir -p`, `cp -r`
- Redirecciones: `>`, `>>`, `2>`, `2>&1`
- Pipes: `|` (combinación de comandos)
- Resumen/Conteo: `wc`, `sort`, `uniq`, `head`, `tail`
- Inspección de archivos: `file`, `stat`
- Comprimir entrega: `tar`

> **Regla:** No uses interfaz gráfica para crear/editar archivos (solo para abrir la terminal).

---

## 0) Preparación

```bash
mkdir -p ~/linux-lab/lab0_5/{sandbox,evidencia}
cd ~/linux-lab/lab0_5
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
  echo "Shell=$SHELL"
} > evidencia/01_identidad.txt
```

---

## 1) Construir una estructura “real” de trabajo (refuerzo de LAB00)

```bash
cd ~/linux-lab/lab0_5/sandbox
mkdir -p proyecto/{docs,src,bin,logs,tmp,output}
ls -la
```

Crea algunos archivos con contenido (sin editor gráfico):

```bash
cat > proyecto/docs/README.txt << 'EOF'
Proyecto de práctica LAB 00.5
Objetivo: reforzar comandos básicos y aprender redirecciones/pipes.
EOF

echo "alpha"  > proyecto/src/a.txt
echo "beta"   > proyecto/src/b.txt
echo "gamma"  > proyecto/src/c.txt
seq 1 80      > proyecto/docs/numeros.txt
```

**Evidencia 02**
```bash
{
  echo "=== Árbol (sin tree) ==="
  cd ~/linux-lab/lab0_5/sandbox
  ls -R proyecto
  echo
  echo "=== head README ==="
  head -n 5 proyecto/docs/README.txt
  echo
  echo "=== tail numeros ==="
  tail -n 5 proyecto/docs/numeros.txt
} > ~/linux-lab/lab0_5/evidencia/02_estructura_y_archivos.txt
```

---

## 2) Globbing/patrones: trabajar “por lotes”

### 2.1 Listar por patrones
```bash
cd ~/linux-lab/lab0_5/sandbox/proyecto/src
ls *.txt
ls ?.txt
```

### 2.2 Copiar por patrones
Copia todos los `.txt` a `tmp/`:

```bash
cp *.txt ../tmp/
ls -la ../tmp
```

### 2.3 Renombrar con `mv` (uno por uno, pero pensando en patrones)
Renombra `a.txt` a `a_v2.txt`:

```bash
mv a.txt a_v2.txt
ls -la
```

**Evidencia 03**
```bash
{
  cd ~/linux-lab/lab0_5/sandbox/proyecto/src
  echo "=== src (después de copiar/renombrar) ==="
  ls -la
  echo
  echo "=== tmp ==="
  ls -la ../tmp
} > ~/linux-lab/lab0_5/evidencia/03_globbing.txt
```

---

## 3) Inspección: ¿qué tipo de archivo es? ¿cuánto pesa?

```bash
cd ~/linux-lab/lab0_5/sandbox/proyecto
file docs/README.txt
stat docs/README.txt
```

**Evidencia 04**
```bash
{
  cd ~/linux-lab/lab0_5/sandbox/proyecto
  echo "=== file ==="
  file docs/README.txt docs/numeros.txt src/b.txt
  echo
  echo "=== stat README ==="
  stat docs/README.txt
} > ~/linux-lab/lab0_5/evidencia/04_file_stat.txt
```

---

## 4) Redirecciones: guardar salida (y errores) de forma controlada

### 4.1 `>` vs `>>`
```bash
cd ~/linux-lab/lab0_5/sandbox/proyecto
ls -la > output/listado.txt
echo "----" >> output/listado.txt
date >> output/listado.txt
```

### 4.2 stderr: mandar errores a otro archivo
Provoca un error controlado:

```bash
ls no_existe 1> output/out.txt 2> output/err.txt
```

### 4.3 Mezclar stdout+stderr en un solo archivo (útil para logs)
```bash
(ls -la && ls no_existe) > output/todo.txt 2>&1
```

**Evidencia 05**
```bash
{
  cd ~/linux-lab/lab0_5/sandbox/proyecto
  echo "=== listado.txt ==="
  cat output/listado.txt
  echo
  echo "=== out.txt ==="
  cat output/out.txt 2>/dev/null || true
  echo
  echo "=== err.txt ==="
  cat output/err.txt 2>/dev/null || true
  echo
  echo "=== todo.txt (head) ==="
  head -n 20 output/todo.txt
} > ~/linux-lab/lab0_5/evidencia/05_redirecciones.txt
```

---

## 5) Pipes: combinar comandos (sin entrar aún a grep/find avanzados)

### 5.1 Contar líneas/palabras/caracteres
```bash
cd ~/linux-lab/lab0_5/sandbox/proyecto
wc -l docs/numeros.txt
wc -w docs/README.txt
```

### 5.2 Pipeline: cuenta cuántos archivos hay en `src` (sin contar `.` y `..`)
```bash
ls -1 src | wc -l
```

### 5.3 Resumen con `sort` y `uniq` (preparación para LAB 01)
Crea un “listado con repetidos”:

```bash
cat > logs/usuarios.txt << 'EOF'
ana
carlos
ana
luis
carlos
carlos
EOF
```

Ahora genera ranking:

```bash
sort logs/usuarios.txt | uniq -c | sort -nr
```

**Evidencia 06**
```bash
{
  cd ~/linux-lab/lab0_5/sandbox/proyecto
  echo "=== wc ==="
  wc -l docs/numeros.txt
  wc -w docs/README.txt
  echo
  echo "=== archivos en src ==="
  ls -1 src | wc -l
  echo
  echo "=== ranking usuarios ==="
  sort logs/usuarios.txt | uniq -c | sort -nr
} > ~/linux-lab/lab0_5/evidencia/06_pipes_wc_sort_uniq.txt
```

---

## 6) Reto integrador 

Crea un reporte `output/reporte_lab0_5.txt` que contenga:

1) Fecha y usuario  
2) Total de archivos `.txt` dentro de `proyecto/` (en cualquier subcarpeta)  
3) Las **10 últimas líneas** de `docs/numeros.txt`  
4) El ranking de `logs/usuarios.txt` (ya ordenado)

> Hint: usa `find` **solo** en modo básico (por nombre) o, si aún no lo has visto, resuélvelo con `ls` dentro de cada carpeta.  
> Si ya conoces `find`, úsalo así: `find proyecto -name "*.txt" | wc -l`

Comando sugerido (puedes modificarlo):

```bash
cd ~/linux-lab/lab0_5/sandbox

{
  echo "=== REPORTE LAB0.5 ==="
  echo "Fecha: $(date)"
  echo "Usuario: $(whoami)"
  echo

  echo "1) Total .txt en proyecto:"
  find proyecto -name "*.txt" | wc -l
  echo

  echo "2) Últimas 10 líneas de numeros.txt:"
  tail -n 10 proyecto/docs/numeros.txt
  echo

  echo "3) Ranking usuarios:"
  sort proyecto/logs/usuarios.txt | uniq -c | sort -nr
} > proyecto/output/reporte_lab0_5.txt
```

**Evidencia 07**
```bash
{
  cd ~/linux-lab/lab0_5/sandbox/proyecto
  echo "=== reporte ==="
  cat output/reporte_lab0_5.txt
} > ~/linux-lab/lab0_5/evidencia/07_reto_final.txt
```

---

## 7) Cierre y entrega

**Resumen**
```bash
cat > ~/linux-lab/lab0_5/evidencia/08_resumen.txt << 'EOF'
Resumen (2–6 líneas)
- 2 cosas que reforzaste del LAB00:
- 2 comandos nuevos que te parecieron más útiles:
- ¿Cuál es la diferencia entre > y >> ?
- ¿Para qué sirve 2> ?
- Un error que te pasó y cómo lo resolviste:
EOF
```

**Comprimir entrega - Colocar tu número de id**
```bash
cd ~/linux-lab/lab0_5
tar -czf lab0_5_refuerzo_02XXXXX.tgz evidencia
ls -la lab0_5_refuerzo_02XXXXX.tgz
```

**Entrega:** `lab0_5_refuerzo_02XXXXX.tgz`

---

## Cheatsheet mini
- Patrones: `*`, `?`
- Redirecciones: `>`, `>>`, `2>`, `2>&1`
- Pipes: `|`
- Resumen: `wc`, `sort`, `uniq`, `head`, `tail`
- Inspección: `file`, `stat`
- Entrega: `tar -czf`
