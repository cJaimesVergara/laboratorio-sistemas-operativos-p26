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

## 0) Preparación

```bash
# Crea la estructura del lab
mkdir -p ~/linux-lab/labIR1/{logs,evidencia,out}

# Entra al lab
cd ~/linux-lab/labIR1

# Verifica ruta y contenido
pwd
ls -la
```

**Evidencia 00**
```bash
{
  echo "=== Identidad ==="
  whoami
  hostname
  date
  echo "PWD=$(pwd)"
  echo "Shell=$SHELL"
  echo "Kernel=$(uname -r)"
} > evidencia/00_identidad.txt
```

---

## 1) Cargar los logs large (v3)

```bash
# Ejecuta desde donde están los .log (por ejemplo, donde descomprimiste este paquete)
cp ./LABIR1_nginx_access_large_v3.log ~/linux-lab/labIR1/logs/access.log
cp ./LABIR1_nginx_error_large_v3.log  ~/linux-lab/labIR1/logs/error.log
```

**Evidencia 01**
```bash
{
  echo "=== logs cargados ==="
  ls -la logs
  echo
  echo "=== access.log head ==="
  head -n 5 logs/access.log
  echo
  echo "=== access.log tail ==="
  tail -n 5 logs/access.log
  echo
  echo "=== error.log head ==="
  head -n 5 logs/error.log
} > evidencia/01_logs_cargados.txt
```


---


## 2) Triage rápido

| Código | Nombre         | Significado en Seguridad |
|-------:|----------------|--------------------------|
| 400    | Bad Request    | La solicitud es corrupta. A menudo causada por herramientas de escaneo (fuzzers) mal configuradas o ataques de malformación de paquetes. |
| 401    | Unauthorized   | El usuario intentó acceder a una zona protegida sin credenciales o con credenciales incorrectas. Es el indicador clave de ataques de fuerza bruta. |
| 403    | Forbidden      | El servidor entiende la petición, pero se niega a autorizarla. Indica que el atacante está intentando acceder a rutas restringidas por el administrador. |
| 404    | Not Found      | El recurso no existe. En auditorías, un volumen alto indica un escaneo de directorios buscando archivos sensibles como `.env`, `.git` o `config.php`. |

---


```bash
wc -l logs/access.log > evidencia/02_total_requests.txt

{
  echo "=== Conteo por status ==="
  echo -n "400: "; grep -c " 400 " logs/access.log
  echo -n "401: "; grep -c " 401 " logs/access.log
  echo -n "403: "; grep -c " 403 " logs/access.log
  echo -n "404: "; grep -c " 404 " logs/access.log
} > evidencia/03_conteo_status.txt
```

---

## 3) Top IPs / Top paths

```bash
cut -d' ' -f1 logs/access.log | sort | uniq -c | sort -nr | head -n 10 > evidencia/04_top10_ips.txt
cut -d' ' -f7 logs/access.log | sort | uniq -c | sort -nr | head -n 15 > evidencia/05_top15_paths.txt
```

---

## 4) Rutas sospechosas + ranking

```bash
grep -E "/\.env|phpmyadmin|server-status|/admin|/login|wp-login\.php|wp-admin|xmlrpc\.php" logs/access.log > evidencia/06_rutas_sospechosas_lineas.txt
grep -E "/\.env|phpmyadmin|server-status|/admin|/login|wp-login\.php|wp-admin|xmlrpc\.php" logs/access.log | cut -d' ' -f7 | sort | uniq -c | sort -nr | head -n 10 > evidencia/07_top10_rutas_sospechosas.txt
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
grep -Ei "UNION|OR%201=1|OR 1=1|\.\./|%2e%2e" logs/access.log > evidencia/08_sqli_traversal.txt
grep -Ei "sqlmap|Nikto|masscan|curl" logs/access.log > evidencia/09_user_agents_herramientas.txt
```

---

## 6) Top por status

```bash
grep " 401 " logs/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -n 10 > evidencia/10_top_ips_401.txt
grep " 403 " logs/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -n 10 > evidencia/11_top_ips_403.txt
grep " 404 " logs/access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -n 10 > evidencia/12_top_ips_404.txt
```

---

## 7) Correlación con error.log (forbidden)

```bash
# En este error.log, la IP está en el campo 11 (token después de client:)
grep "access forbidden by rule" logs/error.log | cut -d' ' -f11 | sort | uniq > out/ips_forbidden.txt

{
  echo "=== IPs forbidden (error.log) ==="
  cat out/ips_forbidden.txt
  echo
  echo "=== Requests en access.log por esas IPs (muestra 20 por IP) ==="
  while read -r ip; do
    ip_clean=$(echo "$ip" | cut -d',' -f1)
    echo
    echo "--- $ip_clean ---"
    grep "^$ip_clean " logs/access.log | head -n 20
  done < out/ips_forbidden.txt
} > evidencia/13_correlacion_forbidden.txt
```

---

## 8) Reporte final

```bash
{
  echo "=== REPORTE INCIDENTE (LAB IR1  v3) ==="
  echo "Fecha: $(date)"
  echo
  echo "1) Total requests:"; cat evidencia/02_total_requests.txt; echo
  echo "2) Conteo por status:"; cat evidencia/03_conteo_status.txt; echo
  echo "3) Top 10 IPs:"; cat evidencia/04_top10_ips.txt; echo
  echo "4) Top 15 paths:"; cat evidencia/05_top15_paths.txt; echo
  echo "5) Top 10 rutas sospechosas:"; cat evidencia/07_top10_rutas_sospechosas.txt; echo
  echo "6) SQLi / Traversal (25 líneas):"; head -n 25 evidencia/08_sqli_traversal.txt; echo
  echo "7) Herramientas por UA (25 líneas):"; head -n 25 evidencia/09_user_agents_herramientas.txt; echo
  echo "8) Top 401/403/404:"; echo "---401---"; cat evidencia/10_top_ips_401.txt; echo "---403---"; cat evidencia/11_top_ips_403.txt; echo "---404---"; cat evidencia/12_top_ips_404.txt; echo
  echo "9) Correlación forbidden (80 líneas):"; head -n 80 evidencia/13_correlacion_forbidden.txt
} | tee out/reporte_incidente.txt > evidencia/14_reporte_final.txt
```

---

## 9) Entrega

```bash
cd ~/linux-lab/labIR1
tar -czf lab_ir1_deteccion_02XXXXX.tgz evidencia
ls -la lab_ir1_deteccion_02XXXXX.tgz
```
