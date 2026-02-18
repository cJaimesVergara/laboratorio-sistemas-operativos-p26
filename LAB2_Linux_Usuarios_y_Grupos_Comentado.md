# LAB 02 — Linux: Creación y manipulación de usuarios y grupos (con usuario personal + sudo) (COMANDOS COMENTADOS)

**Entrega:** `lab2_usuarios_grupos_02XXXXX.tgz` con carpeta `evidencia/`

---

## Objetivo
Al final podrás:

- Partir de un **usuario genérico con sudo** (el que trae tu VM) y crear un **usuario personal** con tu **nombre** y **contraseña**
- Verificar a qué **grupos** pertenece un usuario (`id`, `groups`, `getent`)
- Crear y administrar **grupos** (`groupadd`, `groupmod`, `groupdel`)
- Agregar un usuario a otros grupos (`usermod -aG`, `gpasswd -a`)
- Agregar tu usuario personal a **sudoers** (forma recomendada: grupo `sudo`)
- Cambiar de sesión/cambiar de usuario y **probar sudo** con tu usuario personal
- Crear 2 usuarios adicionales y un espacio compartido por grupo (setgid)

> **Regla de seguridad:** Haz este lab en **Ubuntu dentro de VirtualBox/VM**.  
> No borres usuarios del sistema. Solo trabaja con usuarios creados en este laboratorio.

---

## Convención del laboratorio
Vas a usar estas variables (sustituye por tu información):

- `TU_USUARIO`  → tu usuario personal (ej. `carlos`)
- `TU_GRUPO`    → grupo adicional que crearás (ej. `equipo1`)

> **Importante:** Evita espacios y caracteres especiales en el nombre de usuario.  
> Recomendación: minúsculas + números, ej. `carlosjaimes`.

---

## 0) Preparación (usuario genérico con sudo)
Abre una terminal con el usuario inicial de la VM (el “genérico”) y valida que tengas sudo:

```bash
# whoami imprime el usuario con el que estás logueado actualmente
whoami

# sudo -v valida/actualiza credenciales de sudo (puede pedir password)
sudo -v
```

Crea carpeta de trabajo:

```bash
# Crea estructura del lab:
# -p crea padres si no existen
# {sandbox,evidencia} crea ambas carpetas con expansión de llaves
mkdir -p ~/linux-lab/lab2/{sandbox,evidencia}

# Entra a la carpeta del laboratorio
cd ~/linux-lab/lab2
```

**Evidencia 01**
```bash
{
  echo "=== Usuario inicial (genérico) ==="

  # Usuario actual
  whoami

  # id muestra UID, GID primario y grupos suplementarios
  id

  # groups muestra nombres de grupos del usuario actual
  groups

  echo "=== sudo ok ==="

  # sudo -n true: intenta ejecutar "true" sin pedir password (-n = non-interactive)
  # 2>/dev/null descarta errores si sudo pide password
  # && / || imprime mensaje según éxito o fallo
  sudo -n true 2>/dev/null && echo "sudo: OK (sin pedir password)" || echo "sudo: OK (pidió password o requiere auth)"

  # Fecha y hora
  date

  # Nombre del host
  hostname

  # uname -a muestra info del kernel y sistema
  uname -a

# > redirige stdout del bloque a archivo (sobrescribe)
} > evidencia/01_preparacion.txt
```

---

## 1) Crear tu usuario personal, revisar grupos, agregar a grupo y a sudoers

### 1.1 Crear usuario personal (con home y shell)
Crea tu usuario (sustituye `TU_USUARIO`):

```bash
# adduser crea un usuario de manera "amigable":
# - crea home
# - pide contraseña
# - configura datos básicos
# (requiere sudo)
sudo adduser TU_USUARIO
```

> Durante el asistente te pedirá contraseña (pon tu contraseña elegida) y datos opcionales.

### 1.2 Verificar que existe y a qué grupos pertenece
```bash
# id usuario: muestra UID/GID/grupos del usuario indicado
id TU_USUARIO

# groups usuario: lista grupos (por nombre) del usuario indicado
groups TU_USUARIO

# getent passwd usuario: consulta la base NSS (passwd) para ver su entrada (home, shell, uid, etc.)
getent passwd TU_USUARIO
```

### 1.3 Crear un grupo adicional y agregar tu usuario a ese grupo
Crea un grupo (ej. `equipo1`) y agrega tu usuario personal como miembro:

```bash
# groupadd crea un grupo nuevo (requiere sudo)
sudo groupadd TU_GRUPO

# usermod -aG agrega al usuario a un grupo suplementario:
# -a = append (no reemplaza grupos actuales)
# -G = lista de grupos suplementarios a asignar
sudo usermod -aG TU_GRUPO TU_USUARIO
```

Verifica:

```bash
# getent group grupo: muestra la entrada del grupo y sus miembros
getent group TU_GRUPO

# groups usuario: confirma membresía por nombre
groups TU_USUARIO

# id usuario: confirma por UID/GID y lista de grupos
id TU_USUARIO
```

### 1.4 Agregar tu usuario personal a sudoers (recomendado: grupo `sudo`)
En Ubuntu, la forma estándar es agregar al grupo `sudo`:

```bash
# Agrega al usuario al grupo sudo (le da permisos de sudoers)
sudo usermod -aG sudo TU_USUARIO
```

Verifica:

```bash
# groups usuario: debe incluir "sudo"
groups TU_USUARIO

# getent group sudo: muestra miembros del grupo sudo
# head limita salida para no saturar
getent group sudo | head
```

> Nota: el usuario debe **cerrar sesión e iniciar sesión de nuevo** para que el cambio de grupos aplique en su sesión.

### 1.5 Cambiar de sesión/cambiar de usuario y probar sudo
Opción A (rápida): cambiarte a tu usuario personal en la misma terminal:

```bash
# su - usuario: cambia a ese usuario cargando su entorno (login shell)
su - TU_USUARIO

# Verifica usuario actual
whoami

# Verifica UID/GID/grupos
id

# Lista grupos del usuario actual
groups

# Valida sudo en la sesión del usuario (puede pedir password)
sudo -v

# Ejecuta whoami con privilegios (debe decir root)
sudo whoami

# exit regresa a la sesión anterior (usuario genérico)
exit
```

Opción B: cerrar sesión y entrar como `TU_USUARIO` desde la pantalla de login.

**Evidencia 02**
```bash
{
  echo "=== Usuario personal creado ==="

  # Evidencia de que existe y sus grupos actuales
  id TU_USUARIO
  groups TU_USUARIO

  echo
  echo "=== Grupo adicional ==="
  getent group TU_GRUPO

  echo
  echo "=== Grupo sudo (debe incluir TU_USUARIO) ==="
  # head -n 3 limita la salida (si hay muchos miembros)
  getent group sudo | head -n 3

} > evidencia/02_usuario_personal_y_grupos.txt
```

**Evidencia 03 (prueba de sudo con tu usuario personal)**
```bash
{
  echo "=== Prueba sudo como TU_USUARIO ==="

  # sudo -u usuario ejecuta un comando como ese usuario
  # bash -lc ejecuta un "login shell" con un comando (-c)
  # Dentro se prueban: identidad, grupos, sudo, y sudo whoami
  sudo -u TU_USUARIO bash -lc 'whoami; id; groups; sudo -v; sudo whoami'

} > evidencia/03_prueba_sudo_tu_usuario.txt
```

---

## 2) Crear 2 usuarios “de laboratorio” (mínimo 2) y verificar

Vamos a crear:
- usuario 1: `alumno1`
- usuario 2: `alumno2`

```bash
# adduser crea usuario con home y contraseña (asistente)
sudo adduser alumno1
sudo adduser alumno2
```

Verifica:
```bash
# id muestra UID/GID/grupos de cada usuario
id alumno1
id alumno2

# getent passwd muestra entrada del usuario en passwd/NSS
getent passwd alumno1
getent passwd alumno2
```

**Evidencia 04**
```bash
{
  echo "=== usuarios de laboratorio ==="

  id alumno1
  id alumno2

  echo
  getent passwd alumno1
  getent passwd alumno2

} > evidencia/04_usuarios_laboratorio.txt
```

---

## 3) Contraseñas y políticas (práctico)

Forzar cambio de contraseña al primer login:

```bash
# passwd -e expira la contraseña para forzar cambio en el próximo login
sudo passwd -e alumno1
sudo passwd -e alumno2
```

Ver info de expiración (policy):

```bash
# chage -l lista la política/estado de expiración de la contraseña
sudo chage -l alumno1
sudo chage -l alumno2
```

**Evidencia 05**
```bash
{
  echo "=== chage alumno1 ==="
  sudo chage -l alumno1

  echo
  echo "=== chage alumno2 ==="
  sudo chage -l alumno2

} > evidencia/05_politicas_password.txt
```

---

## 4) Crear y manipular grupos (equipo y proyecto)

Crea:
- grupo: `equipo1`
- grupo: `proyecto`

```bash
# groupadd crea grupos nuevos
sudo groupadd equipo1
sudo groupadd proyecto
```

Verifica:

```bash
# getent group muestra la entrada del grupo y miembros
getent group equipo1
getent group proyecto
```

**Evidencia 06**
```bash
{
  echo "=== grupos creados ==="
  getent group equipo1
  getent group proyecto

} > evidencia/06_grupos_creados.txt
```

---

## 5) Asignar grupos a usuarios (primario y secundario)

### 5.1 Agregar ambos usuarios al grupo `equipo1` (secundario)
```bash
# usermod -aG agrega a grupos suplementarios (sin quitar otros)
sudo usermod -aG equipo1 alumno1
sudo usermod -aG equipo1 alumno2
```

Verifica:
```bash
# groups usuario: lista grupos por nombre
groups alumno1
groups alumno2

# id usuario: confirma grupos por IDs y nombres
id alumno1
id alumno2
```

### 5.2 Hacer que `alumno1` tenga grupo primario `proyecto`
```bash
# usermod -g establece el grupo PRIMARIO (GID) del usuario
sudo usermod -g proyecto alumno1

# id muestra el GID primario actualizado
id alumno1
```

> Nota: el grupo primario afecta el grupo por defecto de archivos creados por el usuario.

**Evidencia 07**
```bash
{
  echo "=== groups alumno1 ==="
  groups alumno1

  echo "=== id alumno1 ==="
  id alumno1

  echo
  echo "=== groups alumno2 ==="
  groups alumno2

  echo "=== id alumno2 ==="
  id alumno2

} > evidencia/07_asignacion_grupos.txt
```

---

## 6) Espacio compartido por grupo (la parte “real”)

Crea `/srv/compartido_equipo1` para el grupo `equipo1`:

```bash
# mkdir -p crea el directorio /srv/compartido_equipo1 si no existe
sudo mkdir -p /srv/compartido_equipo1

# chown cambia dueño y grupo:
# root:equipo1 => dueño root, grupo equipo1
sudo chown root:equipo1 /srv/compartido_equipo1

# chmod 2770:
# 2 (setgid) + 770 (rwxrwx---) => heredar grupo en archivos nuevos
sudo chmod 2770 /srv/compartido_equipo1
```

**Explicación rápida**
- `2770` = `rwxrws---`
- El **2** al inicio activa **setgid** en directorio:  
  todo archivo nuevo dentro hereda el **grupo** `equipo1` (clave para trabajo en equipo).

Verifica:
```bash
# ls -ld muestra permisos del directorio (no su contenido)
ls -ld /srv/compartido_equipo1
```

**Evidencia 08**
```bash
{
  echo "=== permisos carpeta compartida ==="
  ls -ld /srv/compartido_equipo1

} > evidencia/08_carpeta_compartida.txt
```

---

## 7) Probar acceso real con los 2 usuarios

> **Verificación requerida:** Debes demostrar el cambio de usuario usando `su - alumno1` y `su - alumno2` (o `sudo -u` en el reto).

### 7.1 Cambiar a alumno1 y crear archivo
```bash
# su - alumno1: cambia a alumno1 con entorno de login
su - alumno1

# cd cambia a la carpeta compartida
cd /srv/compartido_equipo1

# > crea/sobrescribe el archivo con el texto
echo "Hola desde alumno1" > nota_alumno1.txt

# ls -la lista contenido con detalles y ocultos (si hubiera)
ls -la

# exit regresa a la sesión anterior
exit
```

### 7.2 Cambiar a alumno2 y crear archivo
```bash
# Cambia a alumno2
su - alumno2

# Entra a la carpeta compartida
cd /srv/compartido_equipo1

# Crea archivo del alumno2
echo "Hola desde alumno2" > nota_alumno2.txt

# Lista contenido
ls -la

# cat muestra contenido del archivo del alumno1 (prueba lectura)
cat nota_alumno1.txt

# Regresa a usuario admin
exit
```

### 7.3 Ver dueño/grupo desde tu usuario admin
```bash
# Lista contenido con dueños y grupos
ls -la /srv/compartido_equipo1
```

**Evidencia 09**
```bash
{
  echo "=== listado final compartido ==="
  ls -la /srv/compartido_equipo1

  echo
  echo "=== contenido ==="
  cat /srv/compartido_equipo1/nota_alumno1.txt
  cat /srv/compartido_equipo1/nota_alumno2.txt

} > evidencia/09_prueba_acceso.txt
```

---

## 8) Reto integrador (más reto, pero realista)

### Reto A — “Solo lectura” para alumno2
Queremos que:
- `alumno1` pueda **escribir**
- `alumno2` solo pueda **leer**

Crea un nuevo directorio:
```bash
# Crea directorio
sudo mkdir -p /srv/solo_alumno1

# Dueño alumno1 y grupo equipo1
sudo chown alumno1:equipo1 /srv/solo_alumno1

# chmod 2750:
# 2 setgid + 750 (rwxr-x---) => grupo puede leer/entrar, pero no escribir
sudo chmod 2750 /srv/solo_alumno1
```

> `2750` = `rwxr-s---`  
> Permite al grupo entrar/leer, pero no escribir.

Prueba (sin cambiar de terminal, usando `sudo -u`):
```bash
# Ejecuta como alumno1: crea archivo ok.txt y lista
sudo -u alumno1 bash -lc 'echo "solo alumno1 escribe" > /srv/solo_alumno1/ok.txt && ls -la /srv/solo_alumno1'

# Ejecuta como alumno2: intenta escribir (debe fallar); 2>&1 captura error; || true evita cortar el script
sudo -u alumno2 bash -lc 'echo "intento alumno2" > /srv/solo_alumno1/fail.txt' 2>&1 || true

# Ejecuta como alumno2: lista y lee ok.txt (debe poder)
sudo -u alumno2 bash -lc 'ls -la /srv/solo_alumno1 && cat /srv/solo_alumno1/ok.txt'
```

**Evidencia 10**
```bash
{
  echo "=== permisos /srv/solo_alumno1 ==="
  ls -ld /srv/solo_alumno1

  echo
  echo "=== prueba alumno1 crea archivo ==="
  sudo -u alumno1 bash -lc 'echo "solo alumno1 escribe" > /srv/solo_alumno1/ok.txt && ls -la /srv/solo_alumno1'

  echo
  echo "=== prueba alumno2 intenta escribir (espera error) ==="
  sudo -u alumno2 bash -lc 'echo "intento alumno2" > /srv/solo_alumno1/fail.txt' 2>&1 || true

  echo
  echo "=== alumno2 puede leer/listar ==="
  sudo -u alumno2 bash -lc 'ls -la /srv/solo_alumno1 && cat /srv/solo_alumno1/ok.txt'

} > evidencia/10_reto_A.txt
```

---

## 9) Bonus: renombrar grupo y observar efectos (opcional)

Renombra `equipo1` a `equipo_alpha`:

```bash
# groupmod -n cambia el nombre del grupo
sudo groupmod -n equipo_alpha equipo1

# Verifica nueva entrada del grupo
getent group equipo_alpha
```

> Ojo: los archivos quedan con el **GID**; el nombre mostrado cambia al consultar.

**Evidencia 11 (opcional)**
```bash
{
  echo "=== groupmod ==="
  getent group equipo_alpha

  echo
  echo "=== carpetas con grupo actualizado ==="
  ls -ld /srv/compartido_equipo1 /srv/solo_alumno1

} > evidencia/11_bonus_groupmod.txt
```

---

## 10) Cierre y entrega

**Resumen**
```bash
# Here-doc para crear el archivo de resumen del alumno
cat > evidencia/12_resumen.txt << 'EOF'
Resumen (2–6 líneas)
- ¿Qué diferencia hay entre grupo primario y secundario?
- ¿Cómo comprobaste que TU_USUARIO ya era sudoer?
- ¿Para qué sirve el bit setgid (2xxx) en un directorio compartido?
- 1 problema que tuviste y cómo lo resolviste:
EOF
```

**Comprimir entrega - Colocar tu número de id**
```bash
# Entra a la carpeta del lab
cd ~/linux-lab/lab2

# tar crea un .tgz:
# -c create, -z gzip, -f nombre del archivo
tar -czf lab2_usuarios_grupos_02XXXXX.tgz evidencia

# Verifica que el archivo existe y su tamaño
ls -la lab2_usuarios_grupos_02XXXXX.tgz
```

**Entrega:** `lab2_usuarios_grupos_02XXXXX.tgz`

---

# `su` vs `sudo` en Linux (explicación)

En Linux, **`su`** y **`sudo`** sirven para ejecutar acciones como *otro usuario* (muchas veces como **root**), pero lo hacen de forma distinta.

---

## 1) `su` (switch user)

`su` cambia tu sesión al **usuario indicado** (por default, `root`).

### Uso típico
```bash
su - usuario
```

- El `-` (o `-l`) carga el **entorno completo** del usuario destino (HOME, PATH, etc.).
- Si no pones usuario (`su -`), normalmente intenta cambiar a **root**.

### ¿Qué contraseña pide?
- Normalmente pide la **contraseña del usuario destino** (por ejemplo, la de `root` si haces `su -`).

### Ventaja
- Obtienes una “sesión completa” como ese usuario.

### Riesgo
- Si entras como `root`, todo lo que ejecutes corre como root (más fácil equivocarte).

### Ejemplos
```bash
su -            # cambia a root (si root tiene contraseña habilitada)
su - alumno1    # cambia a alumno1 (carga su entorno)
```

---

## 2) `sudo` (superuser do)

`sudo` ejecuta **un comando** con privilegios de otro usuario (por default, **root**) sin cambiar toda tu sesión.

### Uso típico
```bash
sudo comando
```

### ¿Qué contraseña pide?
- Generalmente pide la **contraseña de tu usuario actual** (no la del usuario destino), si estás autorizado.

### ¿Cómo se autoriza?
- Se controla con `sudoers` (en Ubuntu típicamente por pertenecer al grupo `sudo`).

### Ventaja
- Es más seguro para trabajo diario: elevas permisos **solo para un comando**.
- Es auditable (queda registro de uso en logs del sistema).

### Ejemplos
```bash
sudo apt update
sudo mkdir /srv/compartido

sudo -u alumno1 whoami    # ejecuta un comando como alumno1
```

---

## 3) Diferencia rápida (en una frase)

- **`su`** = “me convierto en otro usuario (sesión)”.
- **`sudo`** = “ejecuto este comando como otro usuario (normalmente root)”.

---

## 4) Comandos útiles para verificar en prácticas

### ¿Quién soy?
```bash
whoami
```

### ¿Qué grupos tengo? (¿soy sudoer?)
```bash
groups
```

### Validar sudo (puede pedir password)
```bash
sudo -v
```

### Ver mi UID/GID/grupos en detalle
```bash
id
```

---


## Cheatsheet mini
- Usuarios: `adduser`, `useradd`, `usermod`, `userdel`, `passwd`, `chage`
- Grupos: `groupadd`, `groupmod`, `groupdel`, `gpasswd`, `getent group`
- Verificación: `id`, `groups`, `getent passwd`
- Sudoers (recomendado): `usermod -aG sudo TU_USUARIO`
- Cambiar de usuario: `su - usuario`, `sudo -u usuario comando`
- Permisos: `chown user:group`, `chmod 2770` (setgid en dir)

