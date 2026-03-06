# LAB U1 — Gestión de usuarios y privilegios (Ubuntu CLI) 

**Nivel:** básico (administración esencial)  
**Tema:** creación, revisión, modificación y cambio entre usuarios; grupos; **sudo** y buenas prácticas.

> ⚠️ Recomendación: realiza este lab en **máquina virtual** o entorno de laboratorio.  
> Algunos pasos requieren privilegios de administrador (`sudo`).

---

## Objetivo didáctico

Aprender a:

- Crear usuarios y asignar contraseñas
- Revisar información de usuarios (UID/GID, home, shell, grupos)
- Modificar usuarios (shell, comentario, grupos)
- Cambiar de usuario (`su`, `sudo -iu`)
- Entender qué hace **sudo**, por qué es más seguro que trabajar como root, y cómo se configura
- Empaquetar evidencias en un `.tar.gz`

---

## Entregables (evidencia)

Carpeta: `~/lab_u1_usuarios/evidencia/` con archivos `.txt`  
Archivo final: `lab_u1_usuarios_evidencia.tar.gz`

---

# 0) Preparación (5 min)

**Propósito:** crear el espacio de trabajo y evidencias.  
**Se espera obtener:** directorio `~/lab_u1_usuarios/` con subcarpeta `evidencia/`.

```bash
mkdir -p ~/lab_u1_usuarios/evidencia                # Crea carpeta del lab y evidencias (si ya existe, no falla)
cd ~/lab_u1_usuarios                                # Entra al directorio del lab
pwd                                                 # Confirma ruta actual (útil para evidencias)
ls -la                                              # Verifica que exista la carpeta evidencia/
```

---

# 1) Recap rápido: ¿qué es un usuario en Linux? (5 min)

**Propósito:** reconocer que un usuario es una identidad con UID/GID, home, shell y grupos.  
**Se espera obtener:** evidencia de tu usuario actual.

```bash
{
  echo "=== Usuario actual ==="                      # Encabezado para ordenar evidencia
  whoami                                            # Muestra el usuario actual (nombre)
  id                                                # Muestra UID, GID y grupos del usuario actual
  echo                                              # Línea en blanco (estética)
  echo "=== Sesión ==="                             # Encabezado sección sesión
  tty                                               # Muestra la terminal actual (útil para su/sudo)
} | tee evidencia/01_usuario_actual.txt              # Muestra en pantalla y guarda en archivo
```

---

# 2) Crear usuarios (15 min)

> Vamos a crear **dos usuarios** para probar cambios y switching:
- `alumno1`
- `alumno2`

## 2.1 Crear `alumno1` con `adduser`

**Propósito:** crear un usuario “completo” (home + contraseña + info básica).  
**Se espera obtener:** usuario `alumno1` con home `/home/alumno1`.

```bash
sudo adduser alumno1                                           # Crea usuario (te pedirá contraseña e info opcional)
getent passwd alumno1 | tee evidencia/02_passwd_alumno1.txt     # Muestra registro del usuario (como en /etc/passwd)
```

## 2.2 Crear `alumno2`

**Propósito:** tener un segundo usuario para comparar permisos/grupos.  
**Se espera obtener:** usuario `alumno2` con home `/home/alumno2`.

```bash
sudo adduser alumno2                                           # Crea segundo usuario con home y contraseña
getent passwd alumno2 | tee evidencia/03_passwd_alumno2.txt     # Evidencia del registro del usuario
```

### Nota didáctica: `adduser` vs `useradd`
- `adduser` (Ubuntu/Debian): interactivo y “amigable”.
- `useradd`: más “bajo nivel”; requiere flags para crear home, etc.

---

# 3) Revisar usuarios y propiedades (10–15 min)

## 3.1 Revisar UID/GID, home y shell

**Propósito:** ver los campos clave del registro del usuario.  
**Se espera obtener:** evidencia con UID, GID, home y shell de ambos.

```bash
{
  echo "=== id alumno1 ==="                           # Encabezado
  id alumno1                                         # UID/GID/grupos de alumno1
  echo                                               # Línea en blanco
  echo "=== id alumno2 ==="                           # Encabezado
  id alumno2                                         # UID/GID/grupos de alumno2
  echo                                               # Línea en blanco
  echo "=== getent passwd (campos) ==="              # Encabezado
  getent passwd alumno1                              # Línea estilo /etc/passwd de alumno1
  getent passwd alumno2                              # Línea estilo /etc/passwd de alumno2
} | tee evidencia/04_revision_ids_y_passwd.txt        # Guarda evidencia completa
```

### ¿Qué significan los campos de `/etc/passwd`?
Formato:  
`usuario:x:UID:GID:comentario:/home/usuario:/bin/bash`

- `usuario`: nombre
- `x`: la contraseña real **NO** está aquí (está en `/etc/shadow`)
- `UID`: identificador numérico del usuario
- `GID`: grupo principal
- `comentario`: GECOS (nombre completo, etc.)
- `home`: directorio home
- `shell`: intérprete (bash, sh, etc.)

> ⚠️ No pegues `/etc/shadow` en evidencias: contiene hashes.

## 3.2 Ver grupos del sistema y del usuario

**Propósito:** entender que los permisos se manejan también por grupos.  
**Se espera obtener:** evidencia de pertenencia a grupos y una muestra de grupos existentes.

```bash
{
  echo "=== Grupos de alumno1 ==="                    # Encabezado
  groups alumno1                                     # Lista grupos de alumno1 (por nombre)
  echo                                               # Línea en blanco
  echo "=== Grupos de alumno2 ==="                    # Encabezado
  groups alumno2                                     # Lista grupos de alumno2
  echo                                               # Línea en blanco
  echo "=== Ejemplo de grupos del sistema (30) ==="  # Encabezado
  getent group | head -n 30                          # Muestra primeros 30 grupos del sistema
} | tee evidencia/05_grupos.txt                       # Guarda evidencia
```

---

# 4) Modificar usuarios (15–20 min)

Shells disponibles
```bash
cat /etc/shells

#Shel del usuario
echo "$SHELL"

#para ver el shell de cada usuario
getent passwd usuario | cut -d: -f7

```



## 4.1 Cambiar el shell de `alumno2`

**Propósito:** ver que el shell es una propiedad del usuario.  
**Se espera obtener:** `alumno2` con shell `/bin/sh` (o el que elijas).

```bash
sudo usermod -s /bin/sh alumno2                      # Cambia el shell de login de alumno2 a /bin/sh
getent passwd alumno2 | tee evidencia/06_shell_alumno2.txt  # Evidencia del cambio en /etc/passwd
```

> Para volver a bash: `sudo usermod -s /bin/bash alumno2`

## 4.2 Agregar `alumno1` a un grupo adicional

**Propósito:** practicar administración de grupos con `usermod -aG`.  
**Se espera obtener:** `alumno1` pertenezca a un grupo extra (ej. `adm` si existe).

```bash
sudo usermod -aG adm alumno1                         # Agrega alumno1 al grupo adm (si existe)
id alumno1 | tee evidencia/07_alumno1_grupo_extra.txt # Evidencia: id muestra grupos actualizados (ojo: puede requerir re-login en sesiones abiertas)
```

> `-aG` es importante:  
> - `-G` reemplaza lista de grupos  
> - `-aG` agrega sin borrar los grupos previos

## 4.3 (Opcional) Cambiar “comentario” (GECOS) de `alumno1`

**Propósito:** ver que hay metadatos del usuario (no permisos).  
**Se espera obtener:** el campo GECOS actualizado.

```bash
sudo usermod -c "Alumno de laboratorio U1" alumno1   # Cambia comentario (GECOS) de alumno1
getent passwd alumno1 | tee evidencia/08_gecos_alumno1.txt  # Evidencia del campo actualizado
```

---

# 5) Cambiar entre usuarios (switching) (10–15 min)

## 5.1 Cambiar a `alumno1` con `su -`

**Propósito:** entrar como otro usuario y ver el cambio de entorno (home, variables, prompt).  
**Se espera obtener:** `whoami` = alumno1 y `pwd` en `/home/alumno1`.

```bash
su - alumno1                                         # Cambia a alumno1 como sesión tipo login (pedirá contraseña de alumno1)
```

Ya dentro, ejecuta:

```bash
whoami                                               # Debe decir alumno1
pwd                                                  # Debe ser /home/alumno1
id                                                   # UID/GID/grupos de alumno1 (en esa sesión)
exit                                                 # Regresa a tu usuario original
```

Guarda evidencia (desde tu usuario original) indicando que lo hiciste:

```bash
{
  echo "=== Antes (usuario actual) ==="              # Encabezado
  whoami                                            # Usuario original
  echo "=== Nota: se ejecutó su - alumno1 y exit ===" # Nota para el profesor
} | tee evidencia/09_switch_su.txt                   # Evidencia
```

> **¿Por qué `su -` con guion?**  
> El guion (`-`) inicia sesión “login”: cambia HOME y carga entorno del usuario.

## 5.2 Alternativa: `sudo -iu alumno1` (con login -i)

**Propósito:** cambiar a otro usuario usando sudo (si tienes permisos).  
**Se espera obtener:** sesión como alumno1 usando privilegios de sudo.
Si no pones -u, sería root.

```bash
sudo -iu alumno1                                     # Abre login shell como alumno1 usando sudo (no usa la contraseña de alumno1, usa la tuya)
whoami                                               # Debe decir alumno1
pwd                                                  # Debe ser /home/alumno1
exit                                                 # Regresa al usuario original
```

Guarda evidencia:

```bash
echo "Se probó sudo -iu alumno1 (verificación manual)" | tee evidencia/10_switch_sudo_iu.txt  # Nota de evidencia
```

---

# 6) ¿Qué es sudo y por qué se usa? (concepto + práctica) (15–20 min)

## 6.1 Explicación breve (para el reporte)

**Propósito:** fijar la idea clave de seguridad de sudo.  
**Se espera obtener:** un archivo con explicación corta.

```bash
cat > evidencia/11_explicacion_sudo.txt <<'EOF'      # Crea un archivo de texto con explicación
sudo:
- Permite elevar privilegios para un comando específico.
- Es más seguro que usar root todo el tiempo.
- Se controla por /etc/sudoers y /etc/sudoers.d/.
- Se recomienda editar con visudo para evitar romper la configuración.
EOF
```

## 6.2 Dar permisos sudo a `alumno1` (método común en Ubuntu)

**Propósito:** permitir que `alumno1` use `sudo` mediante el grupo `sudo`.  
**Se espera obtener:** `alumno1` listado como miembro del grupo `sudo`.

```bash
sudo usermod -aG sudo alumno1                         # Agrega alumno1 al grupo sudo (método típico en Ubuntu)
getent group sudo | tee evidencia/12_grupo_sudo.txt    # Evidencia: muestra los miembros actuales del grupo sudo
```

> Nota: en sesiones ya abiertas de `alumno1`, hay que **salir y entrar** para ver nuevos grupos.

## 6.3 Probar sudo como `alumno1`

**Propósito:** comprobar que `alumno1` puede ejecutar comandos con privilegios.  
**Se espera obtener:** `sudo whoami` imprime `root`.

1) Cambia a `alumno1`:
```bash
su - alumno1                                          # Entra como alumno1 para probar sudo (pedirá contraseña)
```

2) Ya dentro:
```bash
sudo -v                                               # Solicita/actualiza credenciales sudo (pedirá contraseña de alumno1)
sudo whoami                                           # Debe imprimir root
exit                                                  # Regresa al usuario original
```

Guarda evidencia (nota):

```bash
echo "Se probó sudo desde alumno1: sudo whoami => root (verificación manual)" | tee evidencia/13_prueba_sudo_alumno1.txt
```

## 6.4 (Opcional, recomendado) Regla sudo específica con `/etc/sudoers.d/`

**Propósito:** otorgar privilegios acotados (solo un comando), práctica profesional.  
**Se espera obtener:** un archivo `/etc/sudoers.d/alumno2_status` y `visudo -c` OK.

```bash
echo 'alumno2 ALL=(root) NOPASSWD: /usr/bin/systemctl status *' | sudo tee /etc/sudoers.d/alumno2_status >/dev/null  # Crea regla sudo para alumno2
sudo chmod 440 /etc/sudoers.d/alumno2_status                       # Permisos correctos para archivos sudoers.d (lectura root)
sudo visudo -c | tee evidencia/14_visudo_check.txt                 # Verifica sintaxis de sudoers (debe decir OK)
```

Prueba (como alumno2):
```bash
su - alumno2                                         # Entra como alumno2
sudo systemctl status cron 2>/dev/null || echo "Servicio no encontrado"  # Ejecuta comando permitido sin password (según regla)
exit                                                 # Regresa al usuario original
```

---

# 7) Cierre y limpieza (5–10 min)

## 7.1 (Opcional) Quitar sudo a `alumno1`

**Propósito:** practicar reversión de permisos.  
**Se espera obtener:** `alumno1` eliminado del grupo sudo.

```bash
sudo gpasswd -d alumno1 sudo | tee evidencia/15_quitar_sudo.txt     # Elimina alumno1 del grupo sudo
getent group sudo | tee -a evidencia/15_quitar_sudo.txt             # Confirma miembros actuales del grupo sudo
```

## 7.2 (Opcional) Eliminar usuarios creados

**Propósito:** dejar el sistema limpio si es entorno de lab.  
**Se espera obtener:** usuarios eliminados y homes borrados.

```bash
sudo deluser --remove-home alumno1 | tee evidencia/16_borrado_alumno1.txt  # Borra alumno1 y su home
sudo deluser --remove-home alumno2 | tee evidencia/17_borrado_alumno2.txt  # Borra alumno2 y su home
```

> Si no quieres borrar usuarios, omite este paso.

---

# 8) Entrega final: comprimir evidencias a `.tar.gz`

**Propósito:** empaquetar todo para entrega.  
**Se espera obtener:** `lab_u1_usuarios_evidencia.tar.gz`.

```bash
cd ~/lab_u1_usuarios                                       # Entra al directorio del lab
tar -czf ../lab_u1_usuarios_evidencia.tar.gz evidencia      # Comprime evidencia/ en un tar.gz
ls -lh ../lab_u1_usuarios_evidencia.tar.gz                  # Confirma tamaño y existencia del archivo
```

---

## Preguntas de reflexión (para reporte)


```md
## 1) ¿Qué diferencia hay entre `adduser` y `useradd`?

### `adduser` (alto nivel, “amigable” en Ubuntu/Debian)
- Es un **script/wrapper** pensado para humanos.
- Suele ser **interactivo** (te pide contraseña y datos).
- Normalmente **crea el home**, copia `/etc/skel` (archivos de configuración básicos), asigna shell por defecto y configura grupo.

Ejemplo:
```bash
sudo adduser alumno1
```

### `useradd` (bajo nivel, “crudo”)
- Es el comando **base** del sistema (más cercano a “Linux puro”).
- **No interactivo**.
- Si lo usas sin flags, puede **no crear home** ni copiar `/etc/skel`.
- La contraseña suele asignarse aparte (`passwd`).

Ejemplo típico completo:
```bash
sudo useradd -m -s /bin/bash alumno1
sudo passwd alumno1
```

---

## 2) ¿Qué información clave contiene `/etc/passwd` y qué NO debe contener?

### Contiene (por cada usuario, en una línea):
Formato:
`usuario:x:UID:GID:GECOS:HOME:SHELL`

- `usuario`: nombre de cuenta
- `x`: marcador indicando que el hash de contraseña está en `/etc/shadow`
- `UID`: id numérico del usuario
- `GID`: id numérico del grupo principal
- `GECOS`: comentario (nombre completo, etc.)
- `HOME`: directorio home (ej. `/home/usuario`)
- `SHELL`: shell de login (ej. `/bin/bash`)

### NO debe contener:
- **Contraseñas** ni hashes de contraseñas (eso va en `/etc/shadow`, con permisos restringidos).

---

## 3) ¿Por qué `sudo` es preferible a trabajar como root todo el tiempo?

- **Menos riesgo:** trabajas como usuario normal y elevas privilegios solo cuando lo necesitas.
- **Principio de mínimo privilegio:** reduces el daño de errores (ej. borrar archivos críticos).
- **Auditoría:** queda registro de acciones (logs), útil para seguridad y administración.
- **Control granular:** puedes permitir solo ciertos comandos o reglas por usuario/grupo.
- **Sesiones más seguras:** evitas tener una sesión root abierta permanentemente.

---

## 4) ¿Qué cambia al usar `su - usuario` vs `su usuario`?

### `su usuario`
- Cambia el **UID** al usuario destino, pero **mantiene gran parte del entorno actual**.
- Puede conservar variables, PATH y directorio actual, lo que a veces confunde.

### `su - usuario` (login shell)
- Inicia una sesión tipo “login” del usuario destino.
- Cambia a su **HOME** y carga su entorno (perfil), como si acabara de iniciar sesión.
- Es la opción recomendada para “ser ese usuario” de forma limpia.

---

## 5) ¿Qué ventaja tiene usar `/etc/sudoers.d/` en lugar de editar `/etc/sudoers` directamente?

- **Más seguro y mantenible:** evitas tocar el archivo principal.
- **Modularidad:** reglas separadas por usuario/rol/proyecto (más fácil de administrar).
- **Menos riesgo de romper sudo:** si te equivocas en `/etc/sudoers`, puedes dejar el sistema sin sudo.
- **Buenas prácticas:** es el enfoque recomendado para políticas adicionales; se valida con `visudo -c`.
- **Facilita automatización:** despliegas/quitas reglas como archivos independientes.

> Recomendación: siempre validar con `sudo visudo -c`.
```

