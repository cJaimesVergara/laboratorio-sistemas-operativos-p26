# LAB 00 — Linux básico + Archivos/Usuarios/Grupos/Permisos (COMENTADO)

**Entrega:** `lab0_basico_02XXXXX.tgz` con carpeta `evidencia/`

---

## Objetivo
Al final podrás:

- Navegar: `pwd`, `ls`, `cd`
- Archivos: `mkdir`, `touch`, `echo`, `cat`, `less`, `head`, `tail`
- Copiar/mover/renombrar: `cp`, `mv`
- Borrar seguro: `rm`, `rmdir`
- Usuarios y grupos: `whoami`, `id`, `groups`
- Permisos: `ls -l`, `chmod` (básico), entender `rwx` y `ugo`
- Comprimir entrega: `tar`

> **Regla:** No uses interfaz gráfica para crear/editar archivos (solo para abrir la terminal).

---

## 0) Preparación

```bash
# Crea (si no existen) las carpetas sandbox y evidencia dentro de lab0
# -p crea carpetas padre y no falla si ya existían
# {sandbox,evidencia} es expansión de llaves: crea ambas en un solo comando
mkdir -p ~/linux-lab/lab0/{sandbox,evidencia}

# Entra a la carpeta del laboratorio
cd ~/linux-lab/lab0

# Imprime la ruta absoluta del directorio actual (para confirmar ubicación)
pwd

# Lista con detalle el contenido (incluye ocultos, permisos, tamaños, dueño)
ls -la
```

**Evidencia 01**
```bash
# Agrupa comandos en un bloque { ... } y redirige toda la salida (stdout) a un archivo
{
  # Encabezado del bloque
  echo "=== Identidad ==="

  # Muestra el usuario actual
  whoami

  # Muestra el nombre del equipo (hostname)
  hostname

  # Muestra fecha y hora actuales
  date

  # Imprime la variable de entorno HOME
  echo "HOME=$HOME"

  # Imprime el directorio actual (PWD) calculado al momento
  echo "PWD=$(pwd)"
# Redirección con >: crea/sobrescribe el archivo con la salida del bloque
} > evidencia/01_identidad.txt
```

---

## 1) Navegación

```bash
# Entra a la carpeta sandbox (ruta relativa desde lab0)
cd sandbox

# Muestra ruta actual (debería terminar en .../lab0/sandbox)
pwd

# Sube un nivel (directorio padre: lab0)
cd ..

# Confirma ruta actual (debería ser .../lab0)
pwd

# Entra a tu HOME (atajo ~)
cd ~

# Confirma ruta actual (tu carpeta personal)
pwd

# Entra directamente a lab0 usando ruta absoluta con ~
cd ~/linux-lab/lab0

# Confirma ruta final
pwd
```

**Evidencia 02**
```bash
# Bloque: captura evidencia de navegación y listados
{
  # Encabezado
  echo "=== Navegación ==="

  # Imprime el PWD en una línea con etiqueta
  echo "PWD:"; pwd
  echo

  # Lista contenido de lab0 (donde estás al momento de ejecutar)
  echo "Contenido lab0:"; ls -la
  echo

  # Lista contenido de sandbox (carpeta hija)
  echo "Contenido sandbox:"; ls -la sandbox
# Redirige todo a un archivo de evidencia
} > evidencia/02_navegacion.txt
```

---

## 2) Archivos y carpetas

```bash
# Entra a sandbox usando ruta absoluta
cd ~/linux-lab/lab0/sandbox

# Crea carpetas docs y out
# -p evita error si ya existen y permite crear padres si faltan
mkdir -p docs out

# Crea un archivo vacío llamado notas.txt (si existe, actualiza timestamp)
touch notas.txt

# Escribe una línea (sobrescribe) en notas.txt con >
echo "Mi primer laboratorio de Linux" > notas.txt

# Agrega otra línea al final (append) con >>
# $(date) se sustituye por la fecha/hora actuales antes de ejecutar echo
echo "Fecha: $(date)" >> notas.txt

# Muestra el contenido completo del archivo
cat notas.txt
```

**Evidencia 03**
```bash
# Bloque: evidencia de estructura y contenido de notas.txt
{
  echo "=== Estructura sandbox ==="

  # Lista contenido de sandbox con detalle
  ls -la
  echo

  # Muestra el archivo notas.txt
  echo "=== notas.txt ==="
  cat notas.txt
# Guarda la salida en evidencia (ruta absoluta para no depender del cwd)
} > ~/linux-lab/lab0/evidencia/03_archivos.txt
```

---

## 3) Ver archivos

```bash
# Agrega una nueva línea al final del archivo
echo "Linea 1" >> notas.txt

# Agrega otra línea
echo "Linea 2" >> notas.txt

# Agrega otra línea
echo "Linea 3" >> notas.txt

# head muestra las primeras N líneas (-n 3)
head -n 3 notas.txt

# tail muestra las últimas N líneas (-n 3)
tail -n 3 notas.txt

# less abre un visor paginado (sal con q)
less notas.txt
```

**Evidencia 04**
```bash
# Bloque: evidencia de head/tail (solo captura salida, no interactivo como less)
{
  echo "=== head/tail ==="

  # Primeras 3 líneas
  head -n 3 notas.txt
  echo

  # Últimas 3 líneas
  tail -n 3 notas.txt
# Guarda la salida en evidencia
} > ~/linux-lab/lab0/evidencia/04_ver_archivos.txt
```

---

## 4) Copiar, mover y renombrar

```bash
# Copia notas.txt a docs/ con nuevo nombre
cp notas.txt docs/notas_copia.txt

# Renombra (mueve) notas.txt a notas_v2.txt en el mismo directorio
mv notas.txt notas_v2.txt

# Mueve el archivo copiado desde docs/ hacia out/
mv docs/notas_copia.txt out/

# Lista el contenido de sandbox para verificar ubicaciones
ls -la

# Lista el contenido de out para confirmar que llegó el archivo
ls -la out
```

**Evidencia 05**
```bash
# Bloque: evidencia de resultados de copiar/mover y contenido final
{
  echo "=== sandbox ==="
  ls -la
  echo

  echo "=== out ==="
  ls -la out
  echo

  # Muestra el contenido de notas_v2.txt
  echo "=== notas_v2.txt ==="
  cat notas_v2.txt
# Redirige evidencia a archivo
} > ~/linux-lab/lab0/evidencia/05_copiar_mover.txt
```

---

## 5) Usuarios y grupos

### 5.1 ¿Quién soy y a qué grupos pertenezco?
```bash
# Muestra tu usuario actual (login)
whoami

# Muestra uid, gid y grupos (más detallado)
id

# Muestra solo la lista de grupos a los que perteneces
groups
```

**Qué observar:**
- `uid`: tu identificador de usuario
- `gid`: tu grupo principal
- `groups`: lista de grupos que “te dan permisos”

**Evidencia 06**
```bash
# Bloque: guarda evidencia de usuario y grupos
{
  echo "=== Usuario y grupos ==="

  # Usuario actual
  whoami
  echo

  # Detalle de id (uid, gid, grupos)
  id
  echo

  # Lista breve de grupos
  groups
# Guarda la salida del bloque
} > ~/linux-lab/lab0/evidencia/06_usuarios_grupos.txt
```

### 5.2 Ejercicio corto (razonamiento)
Guarda tus respuestas aquí:
```bash
# Crea/sobrescribe el archivo de respuestas usando un heredoc
# << 'EOF' (con comillas) evita expansión de variables/comandos dentro del bloque
cat > ~/linux-lab/lab0/evidencia/06b_respuestas_ug.txt << 'EOF'
Usuario:
Grupo principal:
Otros grupos:
EOF
```

Usa << 'EOF' cuando estás escribiendo plantillas, scripts, comandos, ejemplos, .env, snippets… y no quieres que se ejecuten/expandan cosas accidentalmente.

Usa << EOF cuando sí quieres que el archivo final tenga valores reales (por ejemplo, meter la fecha actual, el usuario actual, rutas, etc.).

---

## 6) Permisos (rwx) paso a paso

### 6.1 Ver permisos con `ls -l`
```bash
# Entra a sandbox (por si estabas en otro lugar)
cd ~/linux-lab/lab0/sandbox

# Lista en formato largo: permisos, dueño, grupo, tamaño, fecha
ls -l

# Lista en formato largo un archivo específico
ls -l notas_v2.txt
```

**Cómo leer una línea:**
Ejemplo: `-rw-r--r-- 1 alumno alumno 123 Feb  5 10:00 notas_v2.txt`

- Primer caracter: `-` archivo, `d` directorio
- Bloques: `u` (usuario), `g` (grupo), `o` (otros)
- Letras: `r` leer, `w` escribir, `x` ejecutar (o “entrar” si es directorio)

### 6.2 Quitar lectura a “otros”
```bash
# chmod o-r: quita permiso de lectura (r) al grupo "others" (o)
chmod o-r notas_v2.txt

# Verifica permisos cambiados
ls -l notas_v2.txt
```

### 6.3 Dejar permisos “privados” (solo tú)
```bash
# chmod 600: u=rw-, g=---, o=--- (archivo privado)
chmod 600 notas_v2.txt

# Verifica resultado
ls -l notas_v2.txt
```

### 6.4 Volver a algo común (legible por todos, editable por ti)
```bash
# chmod 644: u=rw-, g=r--, o=r-- (común en archivos de texto)
chmod 644 notas_v2.txt

# Verifica resultado
ls -l notas_v2.txt
```

### 6.5 Permisos en directorios
```bash
# ls -ld: muestra permisos del directorio (no su contenido)
ls -ld docs out
```

**Evidencia 07**
```bash
# Bloque: evidencia de permisos del archivo y de los directorios
{
  echo "=== Permisos archivo + directorios ==="

  # Permisos del archivo
  ls -l notas_v2.txt
  echo

  # Permisos de directorios (entrada y lectura)
  ls -ld docs out
# Guarda evidencia
} > ~/linux-lab/lab0/evidencia/07_permisos.txt
```

### 6.6 Mini script ejecutable
```bash
# Crea un script hello.sh (sobrescribe si existe) usando heredoc
cat > hello.sh << 'EOF'
#!/usr/bin/env bash
echo "Hola desde bash"
EOF

# Muestra permisos actuales del script (probablemente sin x)
ls -l hello.sh

# chmod +x agrega permiso de ejecución (x) según umask/estado actual
chmod +x hello.sh

# Verifica que ahora tenga x (ej: -rwxr-xr-x)
ls -l hello.sh

# Ejecuta el script desde el directorio actual (./ = ruta relativa)
./hello.sh
```

**Evidencia 08**
```bash
# Bloque: evidencia del script ejecutable y su salida
{
  echo "=== Script ejecutable ==="

  # Permisos y metadatos del script
  ls -l hello.sh

  # Ejecuta el script (debe imprimir "Hola desde bash")
  ./hello.sh
# Guarda evidencia
} > ~/linux-lab/lab0/evidencia/08_script.txt
```

---

## 7) Borrado seguro

```bash
# Crea un archivo temporal
touch basura.txt

# Elimina el archivo (rm borra archivos; cuidado: no hay "papelera" en terminal)
rm basura.txt

# Crea un directorio vacío
mkdir temp

# Elimina directorio SOLO si está vacío
rmdir temp
```

**Evidencia 09**
```bash
# Guarda listado completo del sandbox en un archivo de evidencia
# (así se comprueba que basura.txt ya no existe y temp fue removido)
ls -la ~/linux-lab/lab0/sandbox > ~/linux-lab/lab0/evidencia/09_borrado.txt
```

---

## 8) Cierre y entrega

**Resumen**
```bash
# Crea el archivo de resumen con una plantilla para que el alumno la complete
cat > ~/linux-lab/lab0/evidencia/10_resumen.txt << 'EOF'
Resumen (2–5 líneas)
- 3 comandos que ahora dominas:
- ¿Qué significan rwx para u/g/o en tus palabras?
- 1 error que te pasó y cómo lo resolviste:
EOF
```

**Comprimir entrega - Colocar tu número de id**
```bash
# Entra a la carpeta lab0 (donde está la carpeta evidencia/)
cd ~/linux-lab/lab0

# Empaqueta evidencia/ en un .tgz
# -c crear, -z gzip, -f nombre del archivo
tar -czf lab0_basico_02XXXXX.tgz evidencia

# Verifica que el archivo exista y su tamaño
ls -la lab0_basico_02XXXXX.tgz
```

**Entrega:** `lab0_basico_02XXXXX.tgz`

---

## Apéndice — Permisos con número (600, 775, 777, etc.)

En Linux, los permisos numéricos (por ejemplo **600**, **775**, **777**) son una forma compacta de describir permisos para:

- **u** = usuario (dueño)
- **g** = grupo
- **o** = otros

El orden de los 3 dígitos siempre es: **u g o**.

### Tabla base: r=4, w=2, x=1
Cada dígito es una suma:

- **r (read / leer)** = 4  
- **w (write / escribir)** = 2  
- **x (execute / ejecutar / entrar)** = 1  

Combinaciones típicas:

- **0** = ---  
- **1** = --x  
- **2** = -w-  
- **3** = -wx (2+1)  
- **4** = r--  
- **5** = r-x (4+1)  
- **6** = rw- (4+2)  
- **7** = rwx (4+2+1)  

### Cómo leer un permiso numérico
Ejemplo: `chmod 754 archivo`

- **7** (u) = rwx  
- **5** (g) = r-x  
- **4** (o) = r--  

### Ejemplos comunes
- **600** = `rw-------` # tú lees/escribes, nadie más (archivos privados/credenciales)  
- **644** = `rw-r--r--` # tú editas, los demás solo leen (archivos de texto comunes)  
- **700** = `rwx------` # carpeta/script privado  
- **755** = `rwxr-xr-x` # carpeta ejecutable típica (otros pueden entrar/leer, no escribir)  
- **775** = `rwxrwxr-x` # trabajo en equipo (dueño y grupo editan, otros solo leen/entran)  
- **777** = `rwxrwxrwx` # ⚠️ todos pueden todo (evitar salvo casos muy controlados)  

### “x” significa algo distinto en archivos vs directorios
- En **archivos**: `x` = se puede **ejecutar**.  
- En **directorios**: `x` = se puede **entrar** (`cd`) y **acceder** a contenido dentro.  

> Nota: en directorios, usar `644` suele ser mala idea porque quita `x` a grupo/otros y no podrán entrar.

### Comandos útiles
```bash
# Ver permisos de un archivo
ls -l archivo

# Ver permisos de un directorio (sin listar el contenido interno)
ls -ld carpeta

# Hacer archivo privado (solo tú lees/escribes)
chmod 600 secreto.txt

# Carpeta editable por el grupo (trabajo en equipo)
chmod 775 carpeta_equipo
```

---

## Cheatsheet mini
- `pwd`, `ls -la`, `cd`
- `mkdir -p`, `touch`
- `echo >`, `echo >>`
- `cat`, `less`, `head`, `tail`
- `cp`, `mv`
- `rm`, `rmdir`
- `whoami`, `id`, `groups`
- `ls -l`, `chmod`
- `tar -czf`
