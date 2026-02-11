# LAB 00 — Linux básico + Archivos/Usuarios/Grupos/Permisos
  
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
mkdir -p ~/linux-lab/lab0/{sandbox,evidencia}
cd ~/linux-lab/lab0
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
} > evidencia/01_identidad.txt
```

---

## 1) Navegación

```bash
cd sandbox
pwd
cd ..
pwd
cd ~
pwd
cd ~/linux-lab/lab0
pwd
```

**Evidencia 02**
```bash
{
  echo "=== Navegación ==="
  echo "PWD:"; pwd
  echo
  echo "Contenido lab0:"; ls -la
  echo
  echo "Contenido sandbox:"; ls -la sandbox
} > evidencia/02_navegacion.txt
```

---

## 2) Archivos y carpetas

```bash
cd ~/linux-lab/lab0/sandbox
mkdir -p docs out
touch notas.txt
echo "Mi primer laboratorio de Linux" > notas.txt
echo "Fecha: $(date)" >> notas.txt
cat notas.txt
```

**Evidencia 03**
```bash
{
  echo "=== Estructura sandbox ==="
  ls -la
  echo
  echo "=== notas.txt ==="
  cat notas.txt
} > ~/linux-lab/lab0/evidencia/03_archivos.txt
```

---

## 3) Ver archivos

```bash
echo "Linea 1" >> notas.txt
echo "Linea 2" >> notas.txt
echo "Linea 3" >> notas.txt
head -n 3 notas.txt
tail -n 3 notas.txt
less notas.txt
```

**Evidencia 04**
```bash
{
  echo "=== head/tail ==="
  head -n 3 notas.txt
  echo
  tail -n 3 notas.txt
} > ~/linux-lab/lab0/evidencia/04_ver_archivos.txt
```

---

## 4) Copiar, mover y renombrar

```bash
cp notas.txt docs/notas_copia.txt
mv notas.txt notas_v2.txt
mv docs/notas_copia.txt out/
ls -la
ls -la out
```

**Evidencia 05**
```bash
{
  echo "=== sandbox ==="
  ls -la
  echo
  echo "=== out ==="
  ls -la out
  echo
  echo "=== notas_v2.txt ==="
  cat notas_v2.txt
} > ~/linux-lab/lab0/evidencia/05_copiar_mover.txt
```

---

## 5) Usuarios y grupos

### 5.1 ¿Quién soy y a qué grupos pertenezco?
```bash
whoami
id
groups
```

**Qué observar:**
- `uid`: tu identificador de usuario
- `gid`: tu grupo principal
- `groups`: lista de grupos que “te dan permisos”

**Evidencia 06**
```bash
{
  echo "=== Usuario y grupos ==="
  whoami
  echo
  id
  echo
  groups
} > ~/linux-lab/lab0/evidencia/06_usuarios_grupos.txt
```

### 5.2 Ejercicio corto (razonamiento)
Guarda tus respuestas aquí:
```bash
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
cd ~/linux-lab/lab0/sandbox
ls -l
ls -l notas_v2.txt
```

**Cómo leer una línea:**
Ejemplo: `-rw-r--r-- 1 alumno alumno 123 Feb  5 10:00 notas_v2.txt`

- Primer caracter: `-` archivo, `d` directorio
- Bloques: `u` (usuario), `g` (grupo), `o` (otros)
- Letras: `r` leer, `w` escribir, `x` ejecutar (o “entrar” si es directorio)

### 6.2 Quitar lectura a “otros”
```bash
chmod o-r notas_v2.txt
ls -l notas_v2.txt
```

### 6.3 Dejar permisos “privados” (solo tú)
```bash
chmod 600 notas_v2.txt
ls -l notas_v2.txt
```

### 6.4 Volver a algo común (legible por todos, editable por ti)
```bash
chmod 644 notas_v2.txt
ls -l notas_v2.txt
```

### 6.5 Permisos en directorios
```bash
ls -ld docs out
```

**Evidencia 07**
```bash
{
  echo "=== Permisos archivo + directorios ==="
  ls -l notas_v2.txt
  echo
  ls -ld docs out
} > ~/linux-lab/lab0/evidencia/07_permisos.txt
```

### 6.6 Mini script ejecutable
```bash
cat > hello.sh << 'EOF'
#!/usr/bin/env bash
echo "Hola desde bash"
EOF

ls -l hello.sh
chmod +x hello.sh
ls -l hello.sh
./hello.sh
```

**Evidencia 08**
```bash
{
  echo "=== Script ejecutable ==="
  ls -l hello.sh
  ./hello.sh
} > ~/linux-lab/lab0/evidencia/08_script.txt
```

---

## 7) Borrado seguro

```bash
touch basura.txt
rm basura.txt
mkdir temp
rmdir temp
```

**Evidencia 09**
```bash
ls -la ~/linux-lab/lab0/sandbox > ~/linux-lab/lab0/evidencia/09_borrado.txt
```

---

## 8) Cierre y entrega

**Resumen**
```bash
cat > ~/linux-lab/lab0/evidencia/10_resumen.txt << 'EOF'
Resumen (2–5 líneas)
- 3 comandos que ahora dominas:
- ¿Qué significan rwx para u/g/o en tus palabras?
- 1 error que te pasó y cómo lo resolviste:
EOF
```

**Comprimir entrega - Colocar tu número de id**
```bash
cd ~/linux-lab/lab0
tar -czf lab0_basico_02XXXXX.tgz evidencia
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
ls -l archivo # Ver permisos de archivo
ls -ld carpeta # Ver permisos del directorio (no su contenido)
chmod 600 secreto.txt # Hacer archivo privado
chmod 775 carpeta_equipo # Carpeta editable por el grupo
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
