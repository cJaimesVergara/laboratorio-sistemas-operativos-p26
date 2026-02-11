# Comandos Básicos de Navegación

## 1. pwd - Print Working Directory

Muestra el directorio actual en el que te encuentras.

```bash
pwd
```

**Ejemplo:**
```bash
$ pwd
/home/usuario/documentos
```

## 2. ls - List

Lista los archivos y directorios en el directorio actual.

```bash
ls [opciones] [directorio]
```

**Opciones comunes:**
- `ls -l` : Formato largo con detalles (permisos, propietario, tamaño, fecha)
- `ls -a` : Muestra archivos ocultos (que empiezan con .)
- `ls -lh` : Formato largo con tamaños legibles (KB, MB, GB)
- `ls -R` : Lista recursivamente subdirectorios
- `ls -t` : Ordena por fecha de modificación

**Ejemplos:**
```bash
$ ls
archivo1.txt  carpeta1  carpeta2

$ ls -la
total 24
drwxr-xr-x 4 usuario usuario 4096 Feb 11 10:00 .
drwxr-xr-x 3 usuario usuario 4096 Feb 10 09:00 ..
-rw-r--r-- 1 usuario usuario  256 Feb 11 10:00 archivo1.txt
drwxr-xr-x 2 usuario usuario 4096 Feb 11 09:30 carpeta1

$ ls -lh
total 4.0K
-rw-r--r-- 1 usuario usuario 1.5K Feb 11 10:00 archivo1.txt
```

## 3. cd - Change Directory

Cambia el directorio actual.

```bash
cd [directorio]
```

**Atajos especiales:**
- `cd` o `cd ~` : Ir al directorio home del usuario
- `cd ..` : Subir un nivel (directorio padre)
- `cd -` : Ir al directorio anterior
- `cd /` : Ir al directorio raíz

**Ejemplos:**
```bash
$ cd /home/usuario/documentos
$ pwd
/home/usuario/documentos

$ cd ..
$ pwd
/home/usuario

$ cd -
/home/usuario/documentos
```

## 4. tree

Muestra la estructura de directorios en forma de árbol.

```bash
tree [opciones] [directorio]
```

**Opciones comunes:**
- `tree -L 2` : Limita la profundidad a 2 niveles
- `tree -d` : Solo muestra directorios
- `tree -a` : Incluye archivos ocultos

**Nota:** Si tree no está instalado, puedes instalarlo con:
```bash
sudo apt install tree
```

**Ejemplo:**
```bash
$ tree -L 2
.
├── carpeta1
│   ├── archivo1.txt
│   └── archivo2.txt
└── carpeta2
    └── subcarpeta
```

## 5. Rutas Absolutas vs Relativas

### Rutas Absolutas
Comienzan desde el directorio raíz (/) y especifican la ubicación completa.

```bash
cd /home/usuario/documentos/proyecto
```

### Rutas Relativas
Especifican la ubicación relativa al directorio actual.

```bash
cd ../proyectos
cd ./subcarpeta
```

## Ejercicios Prácticos

1. Navega a tu directorio home y lista todos los archivos incluyendo ocultos
2. Crea una estructura de directorios y usa tree para visualizarla
3. Practica navegar usando rutas absolutas y relativas
4. Usa `ls` con diferentes opciones para familiarizarte con su salida
