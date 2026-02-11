# Usuarios y Permisos

## 1. Usuarios en Linux

### id - Información del Usuario Actual

Muestra los IDs del usuario y grupos.

```bash
id [usuario]
```

**Ejemplo:**
```bash
$ id
uid=1000(usuario) gid=1000(usuario) groups=1000(usuario),4(adm),27(sudo)

$ id root
uid=0(root) gid=0(root) groups=0(root)
```

### users - Usuarios Conectados

Lista los usuarios actualmente conectados.

```bash
$ users
usuario admin
```

## 2. sudo - Ejecutar Comandos como Superusuario

Ejecuta comandos con privilegios de administrador.

```bash
sudo comando
```

**Opciones comunes:**
- `sudo -i` : Inicia una sesión como root
- `sudo -u usuario` : Ejecuta como otro usuario
- `sudo -l` : Lista permisos sudo del usuario

**Ejemplos:**
```bash
$ sudo apt update
$ sudo -i
# whoami
root
# exit

$ sudo -u postgres psql
```

### su - Cambiar de Usuario

Cambia al usuario especificado (switch user).

```bash
su [usuario]
```

**Ejemplos:**
```bash
$ su
Password: 
# whoami
root

$ su - usuario2
Password:
$ whoami
usuario2
```

## 3. Permisos de Archivos

### Entender los Permisos

En Linux, cada archivo tiene permisos para tres categorías:
- **u** (user): Propietario del archivo
- **g** (group): Grupo del archivo
- **o** (others): Otros usuarios

Cada categoría tiene tres tipos de permisos:
- **r** (read): Lectura (4)
- **w** (write): Escritura (2)
- **x** (execute): Ejecución (1)

**Ejemplo de salida de `ls -l`:**
```bash
-rw-r--r-- 1 usuario grupo 1024 Feb 11 10:00 archivo.txt
drwxr-xr-x 2 usuario grupo 4096 Feb 11 09:00 carpeta
```

**Desglose:**
- `-` o `d`: Tipo (archivo o directorio)
- `rw-`: Permisos del propietario (lectura y escritura)
- `r--`: Permisos del grupo (solo lectura)
- `r--`: Permisos de otros (solo lectura)

### chmod - Cambiar Permisos

Modifica los permisos de archivos y directorios.

```bash
chmod [opciones] permisos archivo
```

**Modo Numérico (Octal):**
```bash
$ chmod 755 script.sh
# rwxr-xr-x (7=rwx, 5=r-x, 5=r-x)

$ chmod 644 archivo.txt
# rw-r--r-- (6=rw-, 4=r--, 4=r--)

$ chmod 600 archivo_privado.txt
# rw------- (solo el propietario puede leer y escribir)
```

**Valores comunes:**
- `755`: Ejecutables y directorios (rwxr-xr-x)
- `644`: Archivos regulares (rw-r--r--)
- `600`: Archivos privados (rw-------)
- `700`: Directorios privados (rwx------)

**Modo Simbólico:**
```bash
# Agregar permiso de ejecución al propietario
$ chmod u+x script.sh

# Quitar permiso de escritura al grupo
$ chmod g-w archivo.txt

# Dar permiso de lectura a todos
$ chmod a+r archivo.txt

# Múltiples cambios
$ chmod u+x,g+w,o-r archivo.txt

# Recursivo
$ chmod -R 755 directorio/
```

**Símbolos:**
- `u` = usuario (propietario)
- `g` = grupo
- `o` = otros
- `a` = todos (all)
- `+` = agregar permiso
- `-` = quitar permiso
- `=` = establecer permiso exacto

### chown - Cambiar Propietario

Cambia el propietario y/o grupo de archivos.

```bash
chown [opciones] propietario[:grupo] archivo
```

**Ejemplos:**
```bash
# Cambiar propietario
$ sudo chown nuevo_usuario archivo.txt

# Cambiar propietario y grupo
$ sudo chown nuevo_usuario:nuevo_grupo archivo.txt

# Solo cambiar grupo
$ sudo chown :nuevo_grupo archivo.txt

# Recursivo
$ sudo chown -R usuario:grupo directorio/
```

### chgrp - Cambiar Grupo

Cambia el grupo de archivos.

```bash
chgrp [opciones] grupo archivo
```

**Ejemplo:**
```bash
$ sudo chgrp developers proyecto/
$ sudo chgrp -R developers proyecto/
```

## 4. Gestión de Usuarios (requiere sudo)

### useradd - Crear Usuario

```bash
sudo useradd [opciones] nombre_usuario
```

**Opciones comunes:**
- `-m` : Crea el directorio home
- `-s` : Especifica el shell
- `-G` : Agrega a grupos adicionales

**Ejemplo:**
```bash
$ sudo useradd -m -s /bin/bash nuevo_usuario
$ sudo passwd nuevo_usuario
```

### usermod - Modificar Usuario

```bash
sudo usermod [opciones] nombre_usuario
```

**Ejemplos:**
```bash
# Agregar usuario a un grupo
$ sudo usermod -aG sudo usuario

# Cambiar el shell
$ sudo usermod -s /bin/zsh usuario

# Cambiar el directorio home
$ sudo usermod -d /home/nuevo_home usuario
```

### userdel - Eliminar Usuario

```bash
sudo userdel [opciones] nombre_usuario
```

**Ejemplos:**
```bash
# Eliminar usuario (mantiene home)
$ sudo userdel usuario

# Eliminar usuario y su home
$ sudo userdel -r usuario
```

### passwd - Cambiar Contraseña

```bash
passwd [usuario]
```

**Ejemplos:**
```bash
# Cambiar tu propia contraseña
$ passwd

# Cambiar contraseña de otro usuario (requiere sudo)
$ sudo passwd otro_usuario
```

## 5. Gestión de Grupos

### groups - Ver Grupos de Usuario

```bash
$ groups
usuario adm cdrom sudo dip plugdev

$ groups otro_usuario
```

### groupadd - Crear Grupo

```bash
sudo groupadd nombre_grupo
```

### groupdel - Eliminar Grupo

```bash
sudo groupdel nombre_grupo
```

## 6. umask - Máscara de Permisos Predeterminados

Define los permisos predeterminados para nuevos archivos.

```bash
umask [valor]
```

**Ejemplo:**
```bash
$ umask
0022

$ umask 0077  # Archivos privados por defecto
```

## Ejercicios Prácticos

1. Crea un archivo y examina sus permisos con `ls -l`
2. Practica cambiar permisos con `chmod` en modo numérico y simbólico
3. Crea un script y hazlo ejecutable
4. Experimenta con `sudo` para ejecutar comandos como root
5. Investiga los grupos a los que perteneces con `groups` e `id`
6. Crea una estructura de directorios con diferentes permisos para simular un proyecto colaborativo

## Ejemplo Práctico: Script Ejecutable

```bash
# Crear un script
$ echo '#!/bin/bash' > script.sh
$ echo 'echo "Hola Mundo"' >> script.sh

# Ver permisos (no es ejecutable)
$ ls -l script.sh
-rw-r--r-- 1 usuario usuario 31 Feb 11 10:00 script.sh

# Intentar ejecutar (fallará)
$ ./script.sh
bash: ./script.sh: Permission denied

# Hacer ejecutable
$ chmod +x script.sh
$ ls -l script.sh
-rwxr-xr-x 1 usuario usuario 31 Feb 11 10:00 script.sh

# Ahora funciona
$ ./script.sh
Hola Mundo
```
