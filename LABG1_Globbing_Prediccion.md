# LAB G1 — Globbing y patrones (`*`, `?`, `[]`) + Ejercicio de predicción

**Entrega:** `labG1_globbing_02XXXXX.tgz` con carpeta `evidencia/`

---

## Objetivo
Al final podrás:

- Entender y usar **globbing** en Bash: `*`, `?`, `[]`, rangos y negación (`[!...]` / `[^...]`)
- Predecir la expansión de patrones **antes** de ejecutar comandos
- Verificar resultados y documentar evidencia reproducible
- Generar tu entrega con `tar`

> **Regla:** No uses interfaz gráfica para crear/editar archivos (solo para abrir la terminal).

---

## Concepto clave (1 minuto)
El *globbing* lo expande el **shell** (Bash), **antes** de que el comando se ejecute.

- `*`  → **cero o más** caracteres  
- `?`  → **exactamente 1** carácter  
- `[]` → **exactamente 1** carácter dentro de un conjunto/rango  
  - `[abc]` = a o b o c  
  - `[1-5]` = 1 a 5  
  - `[!A]` o `[^A]` = cualquier cosa excepto A

---

## 0) Preparación

```bash
# Crea estructura del lab
mkdir -p ~/linux-lab/labG1/{sandbox,evidencia}

# Entra al lab
cd ~/linux-lab/labG1

# Verifica ubicación
pwd

# Lista contenido
ls -la
```

**Evidencia 01**
```bash
{
  echo "=== Identidad ==="
  whoami
  hostname
  date
  echo "PWD=$(pwd)"
  echo "Shell=$SHELL"
  echo "Kernel=$(uname -r)"
} > evidencia/01_identidad.txt
```

---

## 1) Dataset de práctica (archivos para probar patrones)

> Aquí creas un conjunto de archivos con nombres “tramposos” para ver diferencias entre `*`, `?` y `[]`.

```bash
cd ~/linux-lab/labG1/sandbox

# Limpia si lo ejecutas más de una vez (sin romper)
rm -rf demo 2>/dev/null || true

# Crea carpeta demo y entra
mkdir -p demo
cd demo

# Crea archivos (vacíos) de ejemplo
touch a.txt b.txt c.txt aa.txt ab.txt ac.txt       img1.png img2.png img10.png       reporte-2025-01.log reporte-2025-02.log reporte-2026-01.log       nota_A.md nota_B.md nota_Z.md

# Verifica
ls -1
```

**Evidencia 02**
```bash
{
  echo "=== Archivos creados (demo) ==="
  cd ~/linux-lab/labG1/sandbox/demo
  ls -1
} > ~/linux-lab/labG1/evidencia/02_dataset.txt
```

---

## 2) Warm-up: `echo` para ver la expansión del shell

> `echo` te deja ver **qué expandió** el shell (sin ejecutar nada “peligroso”).

```bash
cd ~/linux-lab/labG1/sandbox/demo
echo *.txt
echo ?.txt
echo a?.txt
echo img?.png
echo img*.png
echo nota_[A-Z].md
```

**Evidencia 03**
```bash
{
  echo "=== echo (expansión de patrones) ==="
  cd ~/linux-lab/labG1/sandbox/demo
  echo "echo *.txt =>"; echo *.txt
  echo
  echo "echo ?.txt =>"; echo ?.txt
  echo
  echo "echo a?.txt =>"; echo a?.txt
  echo
  echo "echo img?.png =>"; echo img?.png
  echo
  echo "echo img*.png =>"; echo img*.png
  echo
  echo "echo nota_[A-Z].md =>"; echo nota_[A-Z].md
} > ~/linux-lab/labG1/evidencia/03_echo_expansion.txt
```

---

## 3) Ejercicio de predicción (lo importante)

### Instrucciones
1) **NO ejecutes** los comandos aún.  
2) En el archivo `evidencia/04_prediccion.txt`, escribe para cada inciso:
   - ¿Qué archivos esperas que salgan?
   - ¿Cuántos archivos?
3) Luego ejecutas y comparas.

Crea tu hoja de predicción (sin editor gráfico):

```bash
cat > ~/linux-lab/labG1/evidencia/04_prediccion.txt << 'EOF'
PREDICCIÓN — antes de ejecutar

A1) ls *.txt
- espero:
- total:

A2) ls ?.txt
- espero:
- total:

A3) ls a?.txt
- espero:
- total:

A4) ls a*.txt
- espero:
- total:

A5) ls img?.png
- espero:
- total:

A6) ls img*.png
- espero:
- total:

A7) ls img[1-2].png
- espero:
- total:

A8) ls nota_[A-Z].md
- espero:
- total:

A9) ls nota_[!A].md
- espero:
- total:

A10) ls reporte-2025-*.log
- espero:
- total:

A11) ls reporte-2025-0[1-2].log
- espero:
- total:

B1) ls [ab]*.txt
- espero:
- total:

B2) echo img?.png
- espero:
- total:

EOF
```

---

## 4) Ejecución real (verifica tus predicciones)

Ejecuta en este orden (desde `demo/`):

```bash
cd ~/linux-lab/labG1/sandbox/demo

ls *.txt
ls ?.txt
ls a?.txt
ls a*.txt
ls img?.png
ls img*.png
ls img[1-2].png
ls nota_[A-Z].md
ls nota_[!A].md
ls reporte-2025-*.log
ls reporte-2025-0[1-2].log

ls [ab]*.txt
echo img?.png
```

**Evidencia 05 (resultados reales)**
```bash
{
  cd ~/linux-lab/labG1/sandbox/demo

  echo "=== RESULTADOS REALES ==="

  echo
  echo "A1) ls *.txt"
  ls *.txt

  echo
  echo "A2) ls ?.txt"
  ls ?.txt

  echo
  echo "A3) ls a?.txt"
  ls a?.txt

  echo
  echo "A4) ls a*.txt"
  ls a*.txt

  echo
  echo "A5) ls img?.png"
  ls img?.png

  echo
  echo "A6) ls img*.png"
  ls img*.png

  echo
  echo "A7) ls img[1-2].png"
  ls img[1-2].png

  echo
  echo "A8) ls nota_[A-Z].md"
  ls nota_[A-Z].md

  echo
  echo "A9) ls nota_[!A].md"
  ls nota_[!A].md

  echo
  echo "A10) ls reporte-2025-*.log"
  ls reporte-2025-*.log

  echo
  echo "A11) ls reporte-2025-0[1-2].log"
  ls reporte-2025-0[1-2].log

  echo
  echo "B1) ls [ab]*.txt"
  ls [ab]*.txt

  echo
  echo "B2) echo img?.png"
  echo img?.png

} > ~/linux-lab/labG1/evidencia/05_resultados_reales.txt
```

---

## 5) Comparación y reflexión (mini)

Completa:

```bash
cat > ~/linux-lab/labG1/evidencia/06_comparacion.txt << 'EOF'
COMPARACIÓN — predicción vs realidad (4–10 líneas)

1) ¿En qué inciso te equivocaste más y por qué?
2) Explica con tus palabras por qué `img?.png` NO incluye `img10.png`.
3) Explica qué hace `[!A]` en `nota_[!A].md`.
4) ¿Qué aprendiste sobre usar `echo` para “ver” la expansión del shell?
EOF
```

---

## 6) Reto integrador (rápido pero realista)

Crea un reporte `evidencia/07_reto.txt` que contenga:

1) Lista de todos los `.txt`  
2) Lista de `img` con **un solo dígito**  
3) Lista de `nota_` excepto `A`  
4) Lista de logs de 2025  
5) En una línea final, imprime:  
   `Total TXT = N` (usa globbing + `wc -l`)

> Hint: usa `printf "%s\n" patron` para listar uno por línea cuando lo necesites.

```bash
{
  cd ~/linux-lab/labG1/sandbox/demo

  echo "=== RETO G1 ==="
  echo

  echo "1) .txt:"
  ls *.txt
  echo

  echo "2) img con un dígito (img?.png):"
  ls img?.png
  echo

  echo "3) notas excepto A:"
  ls nota_[!A].md
  echo

  echo "4) logs 2025:"
  ls reporte-2025-*.log
  echo

  echo -n "Total TXT = "
  # ls -1 lista uno por línea; wc -l cuenta líneas => cantidad de archivos
  ls -1 *.txt | wc -l

} > ~/linux-lab/labG1/evidencia/07_reto.txt
```

---

## 7) Cierre y entrega

**Checklist**
- [ ] `evidencia/01_identidad.txt`
- [ ] `evidencia/02_dataset.txt`
- [ ] `evidencia/03_echo_expansion.txt`
- [ ] `evidencia/04_prediccion.txt` (llenado por ti)
- [ ] `evidencia/05_resultados_reales.txt`
- [ ] `evidencia/06_comparacion.txt`
- [ ] `evidencia/07_reto.txt`

**Comprimir entrega - Colocar tu número de id**
```bash
cd ~/linux-lab/labG1
tar -czf labG1_globbing_02XXXXX.tgz evidencia
ls -la labG1_globbing_02XXXXX.tgz
```

**Entrega:** `labG1_globbing_02XXXXX.tgz`

---

## Cheatsheet mini
- `*` → cero o más caracteres
- `?` → exactamente 1 carácter
- `[]` → exactamente 1 carácter (conjunto/rango)
- Negación: `[!x]` o `[^x]`
- Ver expansión: `echo patron`
- Entrega: `tar -czf`
