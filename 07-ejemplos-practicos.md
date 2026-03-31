# Ejemplos Prácticos y Ejercicios

## 1. Scripts Básicos en Bash

### Script "Hola Mundo"

```bash
#!/bin/bash
# Mi primer script

echo "Hola Mundo desde Bash!"
echo "Hoy es: $(date)"
echo "Usuario actual: $(whoami)"
```

**Cómo crear y ejecutar:**
```bash
$ nano hola.sh
# (escribir el contenido del script)
$ chmod +x hola.sh
$ ./hola.sh
```

### Script con Variables

```bash
#!/bin/bash

# Definir variables
nombre="Juan"
edad=25
ciudad="Bogotá"

# Usar variables
echo "Nombre: $nombre"
echo "Edad: $edad años"
echo "Ciudad: $ciudad"

# Variable del sistema
echo "Directorio actual: $PWD"
echo "Usuario: $USER"
```

### Script con Argumentos

```bash
#!/bin/bash

# $1, $2, etc. son argumentos
# $0 es el nombre del script
# $# es el número de argumentos

echo "Nombre del script: $0"
echo "Primer argumento: $1"
echo "Segundo argumento: $2"
echo "Número de argumentos: $#"
echo "Todos los argumentos: $@"
```

**Uso:**
```bash
$ ./script.sh arg1 arg2 arg3
```

### Script con Condicionales

```bash
#!/bin/bash

echo "Ingresa un número:"
read numero

if [ $numero -gt 10 ]; then
    echo "El número es mayor que 10"
elif [ $numero -eq 10 ]; then
    echo "El número es exactamente 10"
else
    echo "El número es menor que 10"
fi
```

### Script con Bucles

```bash
#!/bin/bash

# Bucle for
echo "Bucle FOR:"
for i in 1 2 3 4 5; do
    echo "Número: $i"
done

# Bucle for con rango
echo "Bucle con rango:"
for i in {1..5}; do
    echo "Iteración: $i"
done

# Bucle while
echo "Bucle WHILE:"
contador=1
while [ $contador -le 5 ]; do
    echo "Contador: $contador"
    contador=$((contador + 1))
done
```

## 2. Ejemplos Prácticos de Tareas Comunes

### Backup Automático

```bash
#!/bin/bash
# Script de backup simple

# Configuración
ORIGEN="/home/usuario/documentos"
DESTINO="/backup"
FECHA=$(date +%Y%m%d_%H%M%S)
ARCHIVO="backup_$FECHA.tar.gz"

# Crear backup
echo "Iniciando backup..."
tar czf "$DESTINO/$ARCHIVO" "$ORIGEN"

if [ $? -eq 0 ]; then
    echo "Backup completado: $ARCHIVO"
else
    echo "Error en el backup"
    exit 1
fi

# Eliminar backups antiguos (más de 7 días)
find "$DESTINO" -name "backup_*.tar.gz" -mtime +7 -delete
echo "Backups antiguos eliminados"
```

### Limpieza de Archivos Temporales

```bash
#!/bin/bash
# Limpia archivos temporales

echo "Limpiando archivos temporales..."

# Eliminar archivos .tmp
find /tmp -name "*.tmp" -type f -delete

# Eliminar archivos .log antiguos
find /var/log -name "*.log" -mtime +30 -type f

# Limpiar caché de apt (requiere sudo)
sudo apt clean

echo "Limpieza completada"
```

### Monitor de Espacio en Disco

```bash
#!/bin/bash
# Monitorea el espacio en disco

UMBRAL=80
EMAIL="admin@ejemplo.com"

# Verificar cada partición
df -H | grep -vE '^Filesystem|tmpfs|cdrom' | while read salida; do
    uso=$(echo $salida | awk '{print $5}' | cut -d'%' -f1)
    particion=$(echo $salida | awk '{print $1}')
    
    if [ $uso -ge $UMBRAL ]; then
        echo "ALERTA: La partición $particion está al $uso%"
        # Aquí podrías enviar un email o notificación
    fi
done
```

### Información del Sistema

```bash
#!/bin/bash
# Recopila información del sistema

echo "====== INFORMACIÓN DEL SISTEMA ======"
echo ""
echo "Fecha y Hora: $(date)"
echo "Hostname: $(hostname)"
echo "Kernel: $(uname -r)"
echo ""
echo "====== CPU ======"
lscpu | grep "Model name"
echo ""
echo "====== MEMORIA ======"
free -h
echo ""
echo "====== DISCO ======"
df -h
echo ""
echo "====== USUARIOS CONECTADOS ======"
who
echo ""
echo "====== TIEMPO DE ACTIVIDAD ======"
uptime
```

## 3. Uso de Tuberías (Pipes) y Redirecciones

### Redirecciones Básicas

```bash
# Redirigir salida a archivo (sobrescribe)
$ ls -l > listado.txt

# Redirigir salida a archivo (añade al final)
$ echo "Nueva línea" >> archivo.txt

# Redirigir errores
$ comando 2> errores.txt

# Redirigir salida y errores
$ comando > salida.txt 2>&1

# Descartar salida
$ comando > /dev/null 2>&1
```

### Tuberías (Pipes)

```bash
# Combinar comandos
$ ls -l | grep ".txt"
$ ps aux | grep firefox | grep -v grep

# Contar líneas
$ cat archivo.txt | wc -l

# Ordenar y eliminar duplicados
$ cat archivo.txt | sort | uniq

# Buscar y contar
$ grep "error" log.txt | wc -l

# Top 10 archivos más grandes
$ du -h /home | sort -rh | head -10

# Ver procesos con más memoria
$ ps aux --sort=-%mem | head -10
```

### Ejemplos Avanzados

```bash
# Buscar archivos y mostrar su tamaño
$ find . -name "*.log" -exec du -h {} \; | sort -rh

# Buscar y reemplazar en múltiples archivos
$ find . -name "*.txt" -exec sed -i 's/viejo/nuevo/g' {} \;

# Monitorear log en tiempo real con filtro
$ tail -f /var/log/syslog | grep --line-buffered "error"

# Listar archivos ordenados por fecha
$ ls -lt | head -20
```

## 4. Ejercicios Propuestos

### Nivel Básico

1. **Navegación**
   - Navega a tu directorio home
   - Lista todos los archivos incluyendo ocultos
   - Crea una carpeta llamada "practica_bash"
   - Entra a esa carpeta

2. **Creación de Archivos**
   - Crea 5 archivos de texto vacíos
   - Escribe contenido en uno usando echo
   - Copia un archivo con otro nombre
   - Mueve un archivo a otra ubicación

3. **Búsqueda**
   - Encuentra todos los archivos .txt en tu home
   - Busca la palabra "bash" en archivos de texto
   - Cuenta cuántos archivos .txt tienes

### Nivel Intermedio

4. **Script de Respaldo**
   - Crea un script que haga backup de una carpeta
   - El backup debe incluir la fecha en el nombre
   - Debe comprimir los archivos
   - Debe mostrar mensaje de éxito o error

5. **Gestión de Procesos**
   - Lista todos los procesos de tu usuario
   - Encuentra el PID de un proceso específico
   - Inicia un proceso en background
   - Monitorea el uso de recursos con top

6. **Información del Sistema**
   - Crea un script que recopile:
     - Uso de disco
     - Uso de memoria
     - Procesos más pesados
     - Usuarios conectados

### Nivel Avanzado

7. **Automatización**
   - Crea un script que limpie archivos temporales
   - Debe buscar archivos .tmp, .log viejos
   - Debe mostrar qué archivos eliminará
   - Debe pedir confirmación antes de eliminar

8. **Monitor de Sistema**
   - Crea un script que alerte si:
     - El disco está más del 80% lleno
     - La memoria está más del 90% usada
     - Hay más de 200 procesos corriendo

9. **Procesamiento de Logs**
   - Lee un archivo de log
   - Cuenta errores y warnings
   - Extrae las líneas de error a un archivo
   - Genera un resumen

## 5. Trucos y Consejos

### Atajos de Teclado en Bash

```
Ctrl+C: Cancelar comando actual
Ctrl+Z: Suspender proceso
Ctrl+D: Cerrar terminal (EOF)
Ctrl+L: Limpiar pantalla (igual que 'clear')
Ctrl+A: Ir al inicio de la línea
Ctrl+E: Ir al final de la línea
Ctrl+U: Borrar desde cursor hasta inicio
Ctrl+K: Borrar desde cursor hasta final
Ctrl+R: Buscar en historial
Tab: Autocompletar
↑/↓: Navegar por historial
```

### Comandos Útiles

```bash
# Ver historial de comandos
$ history
$ history | grep "comando"

# Ejecutar comando del historial
$ !123  # Ejecuta comando número 123
$ !!    # Ejecuta último comando
$ !$    # Último argumento del comando anterior

# Crear alias
$ alias ll='ls -la'
$ alias update='sudo apt update && sudo apt upgrade'

# Variables de entorno
$ export MI_VAR="valor"
$ echo $MI_VAR

# Buscar comando en el sistema
$ which python3
$ whereis bash
```

### Buenas Prácticas

1. **Siempre comenta tus scripts** para facilitar el mantenimiento
2. **Usa nombres descriptivos** para variables y archivos
3. **Verifica errores** con `$?` o condicionales
4. **Haz backups** antes de operaciones destructivas
5. **Prueba scripts** en entorno seguro primero
6. **Usa -i (interactivo)** con rm, mv, cp para confirmación
7. **No uses sudo** innecesariamente
8. **Documenta** tus scripts con comentarios

## 6. Recursos Adicionales

### Archivos de Configuración Importantes

```bash
~/.bashrc          # Configuración de bash del usuario
~/.bash_profile    # Se ejecuta al login
~/.bash_history    # Historial de comandos
/etc/bash.bashrc   # Configuración global de bash
/etc/profile       # Configuración de sistema
```

### Comando help

```bash
# Ayuda de comandos internos
$ help cd
$ help alias

# Manual de comandos
$ man ls
$ man grep

# Información corta
$ ls --help
$ grep --help
```

### Páginas Man Útiles

```bash
$ man bash       # Manual completo de Bash
$ man chmod      # Permisos de archivos
$ man find       # Búsqueda de archivos
$ man grep       # Búsqueda de patrones
```
