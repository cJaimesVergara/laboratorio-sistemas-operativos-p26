# Manipulación de Archivos y Directorios

## 1. touch - Crear Archivos Vacíos

Crea un archivo vacío o actualiza la fecha de modificación.

```bash
touch nombre_archivo
```

**Ejemplos:**
```bash
$ touch archivo.txt
$ touch archivo1.txt archivo2.txt archivo3.txt
```

## 2. mkdir - Crear Directorios

Crea uno o más directorios.

```bash
mkdir [opciones] nombre_directorio
```

**Opciones comunes:**
- `mkdir -p` : Crea directorios padres si no existen
- `mkdir -v` : Modo verbose (muestra lo que se está creando)

**Ejemplos:**
```bash
$ mkdir mi_carpeta
$ mkdir carpeta1 carpeta2 carpeta3
$ mkdir -p proyectos/2024/enero
```

## 3. cp - Copiar Archivos y Directorios

Copia archivos o directorios.

```bash
cp [opciones] origen destino
```

**Opciones comunes:**
- `cp -r` : Copia recursivamente (para directorios)
- `cp -i` : Modo interactivo (pide confirmación antes de sobrescribir)
- `cp -v` : Modo verbose
- `cp -u` : Copia solo si el origen es más nuevo que el destino

**Ejemplos:**
```bash
$ cp archivo.txt archivo_copia.txt
$ cp archivo.txt /home/usuario/documentos/
$ cp -r carpeta1 carpeta1_copia
$ cp -v *.txt /backup/
```

## 4. mv - Mover o Renombrar Archivos

Mueve archivos o los renombra.

```bash
mv [opciones] origen destino
```

**Opciones comunes:**
- `mv -i` : Modo interactivo
- `mv -v` : Modo verbose
- `mv -u` : Mueve solo si el origen es más nuevo

**Ejemplos:**
```bash
# Renombrar archivo
$ mv archivo_viejo.txt archivo_nuevo.txt

# Mover archivo a otro directorio
$ mv archivo.txt /home/usuario/documentos/

# Mover múltiples archivos
$ mv archivo1.txt archivo2.txt archivo3.txt /destino/

# Renombrar directorio
$ mv carpeta_vieja carpeta_nueva
```

## 5. rm - Eliminar Archivos y Directorios

Elimina archivos o directorios.

```bash
rm [opciones] archivo
```

**Opciones comunes:**
- `rm -r` : Elimina recursivamente (para directorios)
- `rm -i` : Modo interactivo (pide confirmación)
- `rm -f` : Forzar eliminación sin confirmación
- `rm -v` : Modo verbose

**⚠️ ADVERTENCIA:** El comando rm es permanente. No existe "papelera de reciclaje".

**Ejemplos:**
```bash
$ rm archivo.txt
$ rm -i archivo_importante.txt
$ rm -r carpeta/
$ rm -rf carpeta_y_contenido/
$ rm *.tmp
```

## 6. cat - Visualizar Contenido de Archivos

Muestra el contenido completo de un archivo.

```bash
cat [opciones] archivo
```

**Ejemplos:**
```bash
$ cat archivo.txt
$ cat archivo1.txt archivo2.txt
$ cat -n archivo.txt  # Muestra números de línea
```

## 7. more y less - Visualización Paginada

Permiten ver archivos largos página por página.

```bash
more archivo
less archivo
```

**Controles en less:**
- Espacio: Siguiente página
- b: Página anterior
- /texto: Buscar texto
- q: Salir

**Ejemplo:**
```bash
$ less archivo_largo.txt
$ more /var/log/syslog
```

## 8. head y tail - Ver Inicio o Final de Archivos

**head** muestra las primeras líneas:
```bash
head [opciones] archivo
head -n 20 archivo.txt  # Primeras 20 líneas
```

**tail** muestra las últimas líneas:
```bash
tail [opciones] archivo
tail -n 20 archivo.txt  # Últimas 20 líneas
tail -f /var/log/syslog  # Sigue el archivo en tiempo real
```

## 9. find - Buscar Archivos

Busca archivos y directorios basándose en criterios.

```bash
find [directorio] [opciones] [criterios]
```

**Ejemplos:**
```bash
# Buscar por nombre
$ find /home -name "archivo.txt"

# Buscar archivos .txt en el directorio actual
$ find . -name "*.txt"

# Buscar por tipo (f=archivo, d=directorio)
$ find . -type f -name "*.log"

# Buscar archivos modificados en las últimas 24 horas
$ find . -type f -mtime -1

# Buscar y ejecutar comando
$ find . -name "*.tmp" -delete
```

## 10. grep - Buscar Texto en Archivos

Busca patrones de texto en archivos.

```bash
grep [opciones] patrón archivo
```

**Opciones comunes:**
- `grep -i` : Ignora mayúsculas/minúsculas
- `grep -r` : Búsqueda recursiva en directorios
- `grep -n` : Muestra números de línea
- `grep -v` : Muestra líneas que NO coinciden
- `grep -c` : Cuenta las líneas que coinciden

**Ejemplos:**
```bash
$ grep "error" log.txt
$ grep -i "warning" log.txt
$ grep -r "función" ./proyecto/
$ grep -n "TODO" *.py
$ grep -v "comentario" archivo.txt
```

## 11. wc - Contar Líneas, Palabras y Caracteres

```bash
wc [opciones] archivo
```

**Opciones:**
- `wc -l` : Cuenta líneas
- `wc -w` : Cuenta palabras
- `wc -c` : Cuenta caracteres

**Ejemplos:**
```bash
$ wc archivo.txt
  150  1200  8500 archivo.txt  # líneas palabras caracteres

$ wc -l *.txt
```

## Ejercicios Prácticos

1. Crea una estructura de directorios para un proyecto
2. Crea varios archivos de texto con contenido
3. Practica copiar, mover y renombrar archivos
4. Usa find para buscar archivos por diferentes criterios
5. Usa grep para buscar texto en múltiples archivos
6. Combina comandos con pipes (|): `cat archivo.txt | grep "error" | wc -l`
