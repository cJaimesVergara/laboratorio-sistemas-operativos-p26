# Globbing y patrones en Linux (`*`, `?`, `[]`)

Estos ejemplos funcionan en **Bash** (Ubuntu). El *globbing* lo expande el **shell**, no el comando.

---

## Setup rápido (opcional)
Crea una carpeta y archivos de ejemplo para practicar:

```bash
mkdir -p ~/globbing-demo && cd ~/globbing-demo
touch a.txt b.txt c.txt aa.txt ab.txt ac.txt       img1.png img2.png img10.png       reporte-2025-01.log reporte-2025-02.log reporte-2026-01.log       nota_A.md nota_B.md nota_Z.md
ls
```

---

## 1) `*` (cero o más caracteres)

- Todos los `.txt`:
```bash
ls *.txt
```

- Todo lo que empieza con `img`:
```bash
ls img*
```

- Logs de 2025 (cualquier mes):
```bash
ls reporte-2025-*.log
```

---

## 2) `?` (exactamente 1 carácter)

- Archivos con **un solo carácter** antes de `.txt`:
```bash
ls ?.txt
# a.txt b.txt c.txt
```

- `img?.png` (solo un dígito: 1–9; **NO** incluye `img10.png`):
```bash
ls img?.png
# img1.png img2.png
```

---

## 3) `[]` (un solo carácter dentro de un conjunto/rango)

- Solo `a.txt`, `b.txt` o `c.txt`:
```bash
ls [abc].txt
```

- Rango numérico (1 a 2):
```bash
ls img[1-2].png
```

- Rango de letras (A a Z):
```bash
ls nota_[A-Z].md
```

- Negación: “cualquier cosa excepto A” (`[!...]` o `[^...]`):
```bash
ls nota_[!A].md
# o
ls nota_[^A].md
```

---

## 4) Combinaciones comunes

- Empieza con `a` y luego **un carácter** (aa, ab, ac) y termina en `.txt`:
```bash
ls a?.txt
```

- Empieza con `a` y luego **lo que sea**:
```bash
ls a*.txt
```

- Logs de 2025, meses 01 a 02 (si el nombre lo permite):
```bash
ls reporte-2025-0[1-2].log
```

---

## 5) Tip clave: el shell es quien expande

Para ver qué va a expandir el shell:

```bash
echo *.txt
```

Si no hay coincidencias, dependiendo de la configuración del shell, puede:
- dejar el patrón tal cual, o
- no imprimir nada, o
- fallar si `nullglob` está activado (casos avanzados).
