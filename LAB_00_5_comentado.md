# LAB 00.5 — Linux reforzamiento + Redirecciones/Pipelines/Globbing (COMENTADO)

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
# Crea (si no existen) las carpetas sandbox y evidencia dentro de lab0_5
mkdir -p ~/linux-lab/lab0_5/{sandbox,evidencia}

# Entra a la carpeta del laboratorio
cd ~/linux-lab/lab0_5

# Muestra la ruta absoluta (para confirmar dónde estás parado)
pwd

# Lista archivos con detalle (incluye ocultos, permisos, propietario, tamaños)
ls -la
```

**Evidencia 01**
```bash
# Agrupa varios comandos en un bloque y redirige TODO su stdout a un archivo
{
  # Título del bloque
  echo "=== Identidad ==="

  # Muestra el usuario actual
  whoami

  # Muestra el nombre del equipo
  hostname

  # Muestra fecha y hora actuales
  date

  # Imprime la variable HOME (tu carpeta personal)
  echo "HOME=$HOME"

  # Imprime la ruta actual (PWD) calculada en ese momento
  echo "PWD=$(pwd)"

  # Imprime el shell configurado (bash, zsh, etc.)
  echo "Shell=$SHELL"
# Redirección: crea/sobrescribe el archivo de evidencia con la salida del bloque
} > evidencia/01_identidad.txt
```

---

## 1) Construir una estructura “real” de trabajo (refuerzo de LAB00)

```bash
# Entra a la carpeta sandbox (zona de trabajo)
cd ~/linux-lab/lab0_5/sandbox

# Crea estructura de carpetas de un "proyecto" de ejemplo
# -p permite crear padres y no falla si ya existen
# {docs,src,...} crea varias carpetas en una sola expresión
mkdir -p proyecto/{docs,src,bin,logs,tmp,output}

# Verifica la estructura creada
ls -la
```

Crea algunos archivos con contenido (sin editor gráfico):

```bash
# Crea README.txt escribiendo un bloque de texto multi-línea
# "cat > archivo" crea/sobrescribe el archivo con lo que escribas hasta EOF
# << 'EOF' (con comillas) evita expansiones de variables dentro del bloque
cat > proyecto/docs/README.txt << 'EOF'
Proyecto de práctica LAB 00.5
Objetivo: reforzar comandos básicos y aprender redirecciones/pipes.
EOF

# Crea/sobrescribe a.txt con "alpha" (stdout de echo va al archivo)
echo "alpha"  > proyecto/src/a.txt

# Crea/sobrescribe b.txt con "beta"
echo "beta"   > proyecto/src/b.txt

# Crea/sobrescribe c.txt con "gamma"
echo "gamma"  > proyecto/src/c.txt

# Genera números del 1 al 80 y los guarda en numeros.txt
# seq 1 80 produce líneas: 1,2,3,...,80
seq 1 80      > proyecto/docs/numeros.txt
```

**Evidencia 02**
```bash
# Bloque: genera evidencia de estructura y contenido, guardándolo a un archivo
{
  # Título
  echo "=== Árbol (sin tree) ==="

  # Entra al sandbox para que ls -R sea consistente
  cd ~/linux-lab/lab0_5/sandbox

  # Lista recursivamente (subcarpetas) el contenido de proyecto
  ls -R proyecto

  # Línea en blanco
  echo

  # Muestra las primeras 5 líneas del README
  echo "=== head README ==="
  head -n 5 proyecto/docs/README.txt

  # Línea en blanco
  echo

  # Muestra las últimas 5 líneas del archivo de números
  echo "=== tail numeros ==="
  tail -n 5 proyecto/docs/numeros.txt
# Redirige stdout del bloque a un archivo en evidencia
} > ~/linux-lab/lab0_5/evidencia/02_estructura_y_archivos.txt
```

---

## 2) Globbing/patrones: trabajar “por lotes”

### 2.1 Listar por patrones
```bash
# Entra a la carpeta src
cd ~/linux-lab/lab0_5/sandbox/proyecto/src

# Globbing: *.txt = "cualquier nombre" que termine en .txt
ls *.txt

# Globbing: ?.txt = "exactamente 1 carácter" + .txt (ej: a.txt, b.txt, c.txt)
ls ?.txt
```

### 2.2 Copiar por patrones
Copia todos los `.txt` a `tmp/`:

```bash
# Copia todos los .txt del directorio actual hacia ../tmp/
cp *.txt ../tmp/

# Lista el contenido de tmp para verificar copias
ls -la ../tmp
```

### 2.3 Renombrar con `mv` (uno por uno, pero pensando en patrones)
Renombra `a.txt` a `a_v2.txt`:

```bash
# Renombra (mueve) a.txt a a_v2.txt
mv a.txt a_v2.txt

# Lista para verificar el cambio
ls -la
```

**Evidencia 03**
```bash
# Bloque: evidencia del estado de src y tmp tras copiar/renombrar
{
  # Entra a src (por si estabas en otra carpeta)
  cd ~/linux-lab/lab0_5/sandbox/proyecto/src

  # Muestra listado detallado de src
  echo "=== src (después de copiar/renombrar) ==="
  ls -la

  # Línea en blanco
  echo

  # Muestra listado detallado de tmp (carpeta hermana)
  echo "=== tmp ==="
  ls -la ../tmp
# Redirige toda la evidencia a un archivo
} > ~/linux-lab/lab0_5/evidencia/03_globbing.txt
```

---

## 3) Inspección: ¿qué tipo de archivo es? ¿cuánto pesa?

```bash
# Entra a la raíz del proyecto
cd ~/linux-lab/lab0_5/sandbox/proyecto

# 'file' intenta detectar el tipo de archivo (texto, binario, encoding, etc.)
file docs/README.txt

# 'stat' muestra metadatos: tamaño, permisos, dueño, fechas, etc.
stat docs/README.txt
```

**Evidencia 04**
```bash
# Bloque: evidencia de file y stat, guardada en un archivo
{
  # Entra a proyecto (por consistencia)
  cd ~/linux-lab/lab0_5/sandbox/proyecto

  # Tipos de varios archivos
  echo "=== file ==="
  file docs/README.txt docs/numeros.txt src/b.txt

  # Línea en blanco
  echo

  # Metadatos de README
  echo "=== stat README ==="
  stat docs/README.txt
# Redirige stdout del bloque a evidencia
} > ~/linux-lab/lab0_5/evidencia/04_file_stat.txt
```

---

## 4) Redirecciones: guardar salida (y errores) de forma controlada

### 4.1 `>` vs `>>`
```bash
# Entra a proyecto
cd ~/linux-lab/lab0_5/sandbox/proyecto

# Redirección stdout con > : crea/sobrescribe output/listado.txt con el listado
ls -la > output/listado.txt

# Redirección stdout con >> : agrega al final sin borrar lo anterior
echo "----" >> output/listado.txt

# Agrega la fecha al final del mismo archivo
date >> output/listado.txt
```

### 4.2 stderr: mandar errores a otro archivo
Provoca un error controlado:

```bash
# 'ls no_existe' genera error y escribe el mensaje en stderr
# 1> redirige stdout (fd 1) a out.txt
# 2> redirige stderr (fd 2) a err.txt
ls no_existe 1> output/out.txt 2> output/err.txt
```

### 4.3 Mezclar stdout+stderr en un solo archivo (útil para logs)
```bash
# ( ... ) ejecuta en un subshell para agrupar comandos
# && ejecuta el segundo comando solo si el primero tuvo éxito
# > redirige stdout a output/todo.txt
# 2>&1 hace que stderr (2) vaya al mismo destino que stdout (1)
(ls -la && ls no_existe) > output/todo.txt 2>&1
```

**Evidencia 05**
```bash
# Bloque: muestra contenidos de archivos generados por redirecciones
{
  # Entra a proyecto
  cd ~/linux-lab/lab0_5/sandbox/proyecto

  # Muestra el contenido completo de listado.txt
  echo "=== listado.txt ==="
  cat output/listado.txt

  echo
  # Muestra out.txt (si existe); si falla, suprime errores y continúa
  # 2>/dev/null manda stderr de cat a "basura" (silencio)
  # || true evita que el bloque falle si cat no puede leer el archivo
  echo "=== out.txt ==="
  cat output/out.txt 2>/dev/null || true

  echo
  # Muestra err.txt (si existe)
  echo "=== err.txt ==="
  cat output/err.txt 2>/dev/null || true

  echo
  # Muestra solo las primeras 20 líneas de todo.txt (por si es largo)
  echo "=== todo.txt (head) ==="
  head -n 20 output/todo.txt
# Redirige toda la salida del bloque a evidencia
} > ~/linux-lab/lab0_5/evidencia/05_redirecciones.txt
```

---

## 5) Pipes: combinar comandos (sin entrar aún a grep/find avanzados)

### 5.1 Contar líneas/palabras/caracteres
```bash
# Entra a proyecto
cd ~/linux-lab/lab0_5/sandbox/proyecto

# wc -l cuenta líneas del archivo (docs/numeros.txt debe dar 80)
wc -l docs/numeros.txt

# wc -w cuenta palabras del README
wc -w docs/README.txt
```

### 5.2 Pipeline: cuenta cuántos archivos hay en `src` (sin contar `.` y `..`)
```bash
# ls -1 lista "uno por línea"
# | (pipe) manda esa salida como entrada a wc -l para contar líneas
ls -1 src | wc -l
```

### 5.3 Resumen con `sort` y `uniq` (preparación para LAB 01)
Crea un “listado con repetidos”:

```bash
# Crea el archivo logs/usuarios.txt con varias líneas (usuarios repetidos)
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
# sort ordena alfabéticamente
# uniq -c colapsa repetidos contiguos y antepone conteo
# sort -nr ordena numéricamente (-n) de mayor a menor (-r)
sort logs/usuarios.txt | uniq -c | sort -nr
```

**Evidencia 06**
```bash
# Bloque: evidencia de pipes y conteos
{
  # Entra a proyecto
  cd ~/linux-lab/lab0_5/sandbox/proyecto

  # Conteos con wc
  echo "=== wc ==="
  wc -l docs/numeros.txt
  wc -w docs/README.txt

  echo
  # Cuenta archivos en src
  echo "=== archivos en src ==="
  ls -1 src | wc -l

  echo
  # Ranking de usuarios
  echo "=== ranking usuarios ==="
  sort logs/usuarios.txt | uniq -c | sort -nr
# Guarda todo en evidencia
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
# Entra a sandbox (donde vive la carpeta proyecto/)
cd ~/linux-lab/lab0_5/sandbox

# Bloque que genera el reporte completo y lo redirige a un archivo
{
  # Encabezado
  echo "=== REPORTE LAB0.5 ==="

  # Inserta fecha/hora actual usando sustitución $(...)
  echo "Fecha: $(date)"

  # Inserta usuario actual usando sustitución $(...)
  echo "Usuario: $(whoami)"
  echo

  # Sección 1: contar .txt usando find (por nombre) y wc -l (conteo)
  echo "1) Total .txt en proyecto:"
  find proyecto -name "*.txt" | wc -l
  echo

  # Sección 2: últimas 10 líneas del archivo de números
  echo "2) Últimas 10 líneas de numeros.txt:"
  tail -n 10 proyecto/docs/numeros.txt
  echo

  # Sección 3: ranking de usuarios (ordenar, contar repetidos, ordenar por conteo desc)
  echo "3) Ranking usuarios:"
  sort proyecto/logs/usuarios.txt | uniq -c | sort -nr
# Redirección: crea/sobrescribe el reporte final
} > proyecto/output/reporte_lab0_5.txt
```

**Evidencia 07**
```bash
# Bloque: evidencia del reporte final
{
  # Entra a proyecto
  cd ~/linux-lab/lab0_5/sandbox/proyecto

  # Muestra el reporte para verificarlo
  echo "=== reporte ==="
  cat output/reporte_lab0_5.txt
# Guarda en evidencia
} > ~/linux-lab/lab0_5/evidencia/07_reto_final.txt
```

---

## 7) Cierre y entrega

**Resumen**
```bash
# Crea el archivo de resumen con un template (el alumno debe completarlo)
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
# Entra a la carpeta del laboratorio
cd ~/linux-lab/lab0_5

# Empaqueta la carpeta evidencia/ en un .tgz
# -c crear, -z gzip, -f nombre de archivo
tar -czf lab0_5_refuerzo_02XXXXX.tgz evidencia

# Verifica que el archivo .tgz exista y su tamaño
ls -la lab0_5_refuerzo_02XXXXX.tgz
```

**Entrega:** `lab0_5_refuerzo_02XXXXX.tgz`

---
```bash
find . -maxdepth 1 -mindepth 1 -type d | wc -l
stat .
```


---

## Cheatsheet mini
- Patrones: `*`, `?`
- Redirecciones: `>`, `>>`, `2>`, `2>&1`
- Pipes: `|`
- Resumen: `wc`, `sort`, `uniq`, `head`, `tail`
- Inspección: `file`, `stat`
- Entrega: `tar -czf`
