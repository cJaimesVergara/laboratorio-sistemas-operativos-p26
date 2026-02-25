# LAB IR1 — Detección y Análisis de Incidentes en Nginx (solo CLI) 

**Entrega:** `lab_ir1_deteccion_02XXXXX.tgz` con carpeta `evidencia/`   
**Enfoque:** *Solo detección y análisis* (sin contención).  
**Regla:** No uses interfaz gráfica para crear/editar archivos (solo para abrir la terminal).

---

## Archivos del laboratorio (logs “large” v3)
Este paquete incluye:

- `LABIR1_nginx_access_large_v3.log`
- `LABIR1_nginx_error_large_v3.log`

Los usarás como:

- `~/linux-lab/labIR1/logs/access.log`
- `~/linux-lab/labIR1/logs/error.log`

---

## Estructura de `access.log` (Nginx, estilo *combined*)

Ejemplo:
```text
203.0.113.67 - - [21/Feb/2026:08:40:00 -0600] "POST /api/status HTTP/1.1" 200 68 "-" "Mozilla/5.0_(iPhone)"
```

| Parte | Ejemplo | Significado |
|---|---|---|
| IP del cliente | `203.0.113.67` | IP origen que hizo la petición. |
| Ident / usuario autenticado | `- -` | Campos históricos: `identd` y “remote user”. Normalmente `-` si no aplica. |
| Timestamp | `[21/Feb/2026:08:40:00 -0600]` | Fecha/hora de la request + zona horaria. |
| Request line | `"POST /api/status HTTP/1.1"` | Método + ruta + versión HTTP. |
| Status code | `200` | Resultado HTTP (éxito/error). |
| Bytes | `68` | Tamaño de respuesta en bytes (aprox, según config). |
| Referer | `"-"` | Página de origen (si existe). `-` si no se envió. |
| User-Agent | `"Mozilla/5.0_(iPhone)"` | Identifica el cliente (navegador/herramienta). |

---

## Estructura de `error.log` (Nginx)

Ejemplo:
```text
2026/02/21 10:39:07 [warn]  1876#1876: *3330 upstream response is buffered to a temporary file, client: 192.0.2.55, server: site.local, request: "POST /login HTTP/1.1", upstream: "http://127.0.0.1:3000/login", host: "site.local"
```

| Parte | Ejemplo | Significado |
|---|---|---|
| Timestamp | `2026/02/21 10:39:07` | Fecha/hora del evento. |
| Nivel | `[warn]` | Severidad (`warn`, `error`, etc.). |
| Worker PID#TID | `1876#1876` | Proceso/hilo de Nginx que registró el evento. |
| Connection ID | `*3330` | ID interno de conexión (útil para correlación interna). |
| Mensaje | `upstream response is buffered to a temporary file` | Nginx tuvo que “bufferizar” la respuesta del upstream a un archivo temporal (suele ocurrir por tamaño/velocidad de respuesta). |
| Client | `client: 192.0.2.55` | IP del cliente que detonó el evento. |
| Server | `server: site.local` | Virtual host (bloque `server{}`) que atendió. |
| Request | `request: "POST /login HTTP/1.1"` | Request line asociada al evento. |
| Upstream | `upstream: "http://127.0.0.1:3000/login"` | Backend al que Nginx proxyeó (ej. app Node). |
| Host | `host: "site.local"` | Valor del header Host recibido. |


---

## 0) Preparación

```bash
# Crea la estructura del lab
mkdir -p ~/linux-lab/labIR1/{logs,evidencia,out}  # crea carpetas (con -p incluye padres)

# Entra al lab
cd ~/linux-lab/labIR1  # cambia de directorio

# Verifica ruta y contenido
pwd  # muestra el directorio actual
ls -la  # lista archivos (con -la incluye ocultos y detalles)
```

**Evidencia 00**
```bash
{
  echo "=== Identidad ==="  # imprime texto/variables
  whoami  # muestra el usuario actual
  hostname  # muestra el nombre del host
  date  # muestra fecha y hora
  echo "PWD=$(pwd)"  # imprime texto/variables
  echo "Shell=$SHELL"  # imprime texto/variables
  echo "Kernel=$(uname -r)"  # imprime texto/variables
} > evidencia/00_identidad.txt  # redirige la salida del bloque a un archivo
```

---

## 1) Cargar los logs large (v3)

```bash
# Ejecuta desde donde están los .log (por ejemplo, donde descomprimiste este paquete)
cp ./LABIR1_nginx_access_large_v3.log ~/linux-lab/labIR1/logs/access.log  # copia archivos
cp ./LABIR1_nginx_error_large_v3.log  ~/linux-lab/labIR1/logs/error.log  # copia archivos
```

**Evidencia 01**
```bash
{
  echo "=== logs cargados ==="  # imprime texto/variables
  ls -la logs  # lista archivos (con -la incluye ocultos y detalles)
  echo  # imprime texto/variables
  echo "=== access.log head ==="  # imprime texto/variables
  head -n 5 logs/access.log  # muestra las primeras N líneas
  echo  # imprime texto/variables
  echo "=== access.log tail ==="  # imprime texto/variables
  tail -n 5 logs/access.log  # muestra las últimas N líneas
  echo  # imprime texto/variables
  echo "=== error.log head ==="  # imprime texto/variables
  head -n 5 logs/error.log  # muestra las primeras N líneas
} > evidencia/01_logs_cargados.txt  # redirige la salida del bloque a un archivo
```


---


## 2) Triage rápido

| Código | Nombre         | Significado en Seguridad |
|-------:|----------------|--------------------------|
| 400    | Bad Request    | La solicitud es corrupta. A menudo causada por herramientas de escaneo mal configuradas o ataques de malformación de paquetes. |
| 401    | Unauthorized   | El usuario intentó acceder a una zona protegida sin credenciales o con credenciales incorrectas. Es el indicador clave de ataques de fuerza bruta. |
| 403    | Forbidden      | El servidor entiende la petición, pero se niega a autorizarla. Indica que el atacante está intentando acceder a rutas restringidas por el administrador. |
| 404    | Not Found      | El recurso no existe. En auditorías, un volumen alto indica un escaneo de directorios buscando archivos sensibles como `.env`, `.git` o `config.php`. |

---


```bash
wc -l logs/access.log > evidencia/02_total_requests.txt  # cuenta (con -l líneas)

{
  echo "=== Conteo por status ==="  # imprime texto/variables
  echo -n "400: "; grep -c " 400 " logs/access.log  # imprime texto/variables
  echo -n "401: "; grep -c " 401 " logs/access.log  # imprime texto/variables
  echo -n "403: "; grep -c " 403 " logs/access.log  # imprime texto/variables
  echo -n "404: "; grep -c " 404 " logs/access.log  # imprime texto/variables
} > evidencia/03_conteo_status.txt  # redirige la salida del bloque a un archivo
```

---

## 3) Top IPs / Top paths

```bash
cut -d' ' -f1 logs/access.log | sort | uniq -c | sort -nr | head -n 10 > evidencia/04_top10_ips.txt  # extrae campos usando delimitador
cut -d' ' -f7 logs/access.log | sort | uniq -c | sort -nr | head -n 15 > evidencia/05_top15_paths.txt  # extrae campos usando delimitador
```

---

## 4) Rutas sospechosas + ranking

```bash
grep -E "/\.env|phpmyadmin|server-status|/admin|/login|wp-login\.php|wp-admin|xmlrpc\.php" logs/access.log > evidencia/06_rutas_sospechosas_lineas.txt  # filtra líneas que coinciden con patrón
grep -E "/\.env|phpmyadmin|server-status|/admin|/login|wp-login\.php|wp-admin|xmlrpc\.php" logs/access.log | cut -d' ' -f7 | sort | uniq -c | sort -nr | head -n 10 > evidencia/07_top10_rutas_sospechosas.txt  # filtra líneas que coinciden con patrón
```

---

## 5) SQLi / Traversal + herramientas por UA

---
### Ataques detectados por patrones (en logs)

### `UNION` (SQL Injection)
- **Qué intenta:** “pegar” otra consulta SQL a la original para **extraer datos**.
- **Ejemplo típico:** `...?id=1 UNION SELECT user,pass FROM users`
- **Cómo se ve en logs:** aparece como `UNION` (a veces URL-encoded con `%20` por espacios) dentro de la query string.

### `OR 1=1` / `OR%201=1` (SQL Injection)
- **Qué intenta:** forzar una condición **siempre verdadera** para **saltarse autenticación** o filtros.
- `OR 1=1` = texto normal  
- `OR%201=1` = lo mismo pero URL-encoded (`%20` = espacio)
- **Ejemplo:** `...?user=admin' OR 1=1--`
- **Cómo se ve en logs:** parámetros con comillas (`'`), `OR`, y a veces `--` (comentario SQL) con múltiples variaciones.

### `\.\./` (Path Traversal)
- En la regex, `\.\./` representa `../` (se “escapa” el punto para que sea literal).
- **Qué intenta:** “subir” directorios para leer archivos fuera del sitio web.
- **Ejemplo:** `/../../etc/passwd`
- **Cómo se ve en logs:** rutas con `../` repetidas o mezcladas con rutas inusuales.

### `%2e%2e` (Path Traversal URL-encoded)
- `%2e` es `.` en URL encoding, por tanto `%2e%2e` = `..`
- **Qué intenta:** lo mismo que `../`, pero **codificado** para evadir filtros simples.
- **Ejemplo:** `/%2e%2e/%2e%2e/etc/shadow`
- **Cómo se ve en logs:** secuencias `%2e%2e` en la URL, a veces junto a `%2f` (`/` codificado).

---

## Agentes (User-Agents) y qué implican

### `sqlmap`
- **Qué es:** herramienta automática para **detectar y explotar SQLi**.
- **Qué implica en logs:** alta probabilidad de intentos SQLi repetidos con variaciones (`UNION`, `OR 1=1`, etc.).

### `Nikto`
- **Qué es:** scanner de vulnerabilidades web.
- **Qué implica:** prueba rutas/archivos comunes, configuraciones débiles y patrones como **traversal** y endpoints conocidos.

### `masscan`
- **Qué es:** escáner **masivo** (principalmente de puertos), muy usado en reconocimiento.
- **En access.log:** suele indicar **recon/escaneo automatizado** (a veces el UA puede estar falsificado, pero sigue siendo una señal útil).

### `curl`
- **Qué es:** cliente HTTP de línea de comandos.
- **Qué implica:** puede ser benigno (pruebas internas) o malicioso (scripts/bots).  
  La clave es el **contexto**: si aparece junto a `/.env`, `/phpmyadmin`, `/server-status`, etc., normalmente es scanning.

---

## Idea práctica
- Los patrones `UNION | OR 1=1 | ../ | %2e%2e` indican **qué intentaron hacer**.
- Los User-Agents `sqlmap | Nikto | masscan | curl` indican **con qué herramienta / nivel de automatización**, ayudando a confirmar la hipótesis.
---


```bash
grep -Ei "UNION|OR%201=1|OR 1=1|\.\./|%2e%2e" logs/access.log > evidencia/08_sqli_traversal.txt  # filtra líneas que coinciden con patrón
grep -Ei "sqlmap|Nikto|masscan|curl" logs/access.log > evidencia/09_user_agents_herramientas.txt  # filtra líneas que coinciden con patrón
```

---

## 6) Top por status

```bash
grep " 401 " logs/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -n 10 > evidencia/10_top_ips_401.txt  # filtra líneas que coinciden con patrón
grep " 403 " logs/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -n 10 > evidencia/11_top_ips_403.txt  # filtra líneas que coinciden con patrón
grep " 404 " logs/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -n 10 > evidencia/12_top_ips_404.txt  # filtra líneas que coinciden con patrón
```

---

## 7) Correlación con error.log (forbidden)

```bash
# En este error.log, la IP está en el campo 11 (token después de client:)
grep "access forbidden by rule" logs/error.log | cut -d' ' -f11 | sort | uniq > out/ips_forbidden.txt  # filtra líneas que coinciden con patrón

{
  echo "=== IPs forbidden (error.log) ==="  # imprime texto/variables
  cat out/ips_forbidden.txt  # muestra contenido de archivo
  echo  # imprime texto/variables
  echo "=== Requests en access.log por esas IPs (muestra 20 por IP) ==="  # imprime texto/variables
  while read -r ip; do  # loop: lee una línea por iteración (sin escapes con -r)
    ip_clean=$(echo "$ip" | cut -d',' -f1)  # limpia la IP quitando la coma final
    echo  # imprime texto/variables
    echo "--- $ip_clean ---"  # imprime texto/variables
    grep "^$ip_clean " logs/access.log | head -n 20  # filtra líneas que coinciden con patrón
  done < out/ips_forbidden.txt  # fin del loop; lee desde el archivo indicado
} > evidencia/13_correlacion_forbidden.txt  # redirige la salida del bloque a un archivo
```

---

## 8) Reporte final

```bash
{
  echo "=== REPORTE INCIDENTE (LAB IR1  v3) ==="  # imprime texto/variables
  echo "Fecha: $(date)"  # imprime texto/variables
  echo  # imprime texto/variables
  echo "1) Total requests:"; cat evidencia/02_total_requests.txt; echo  # imprime texto/variables
  echo "2) Conteo por status:"; cat evidencia/03_conteo_status.txt; echo  # imprime texto/variables
  echo "3) Top 10 IPs:"; cat evidencia/04_top10_ips.txt; echo  # imprime texto/variables
  echo "4) Top 15 paths:"; cat evidencia/05_top15_paths.txt; echo  # imprime texto/variables
  echo "5) Top 10 rutas sospechosas:"; cat evidencia/07_top10_rutas_sospechosas.txt; echo  # imprime texto/variables
  echo "6) SQLi / Traversal (25 líneas):"; head -n 25 evidencia/08_sqli_traversal.txt; echo  # imprime texto/variables
  echo "7) Herramientas por UA (25 líneas):"; head -n 25 evidencia/09_user_agents_herramientas.txt; echo  # imprime texto/variables
  echo "8) Top 401/403/404:"; echo "---401---"; cat evidencia/10_top_ips_401.txt; echo "---403---"; cat evidencia/11_top_ips_403.txt; echo "---404---"; cat evidencia/12_top_ips_404.txt; echo  # imprime texto/variables
  echo "9) Correlación forbidden (80 líneas):"; head -n 80 evidencia/13_correlacion_forbidden.txt  # imprime texto/variables
} | tee out/reporte_incidente.txt > evidencia/14_reporte_final.txt  # muestra en pantalla y guarda en archivo con tee
```

---

## 9) Entrega

```bash
cd ~/linux-lab/labIR1  # cambia de directorio
tar -czf lab_ir1_deteccion_02XXXXX.tgz evidencia  # empaqueta archivos en .tgz
ls -la lab_ir1_deteccion_02XXXXX.tgz  # lista archivos (con -la incluye ocultos y detalles)
```
