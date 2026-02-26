# Comandos de Red

## 1. ifconfig - Configuración de Interfaces de Red (deprecado)

Muestra o configura interfaces de red.

**Nota:** En sistemas modernos, se recomienda usar `ip` en su lugar.

```bash
ifconfig [interfaz]
```

**Instalación si no está disponible:**
```bash
sudo apt install net-tools
```

**Ejemplo:**
```bash
$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        ether 00:11:22:33:44:55  txqueuelen 1000  (Ethernet)
```

## 2. ip - Configuración de Red Moderna

Comando moderno para gestionar interfaces de red.

```bash
ip [opciones] objeto comando
```

**Comandos comunes:**

### Ver direcciones IP
```bash
$ ip address show
$ ip a
$ ip addr show eth0
```

### Ver rutas
```bash
$ ip route show
$ ip r
```

### Ver interfaces
```bash
$ ip link show
```

**Ejemplos:**
```bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    inet 127.0.0.1/8 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0

$ ip route
default via 192.168.1.1 dev eth0 
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100
```

## 3. ping - Probar Conectividad

Envía paquetes ICMP para probar conectividad de red.

```bash
ping [opciones] host
```

**Opciones comunes:**
- `ping -c N` : Envía N paquetes y se detiene
- `ping -i segundos` : Intervalo entre paquetes
- `ping -s tamaño` : Tamaño del paquete

**Ejemplos:**
```bash
# Ping continuo (Ctrl+C para detener)
$ ping google.com

# Enviar solo 4 paquetes
$ ping -c 4 google.com
PING google.com (172.217.168.46) 56(84) bytes of data.
64 bytes from google.com: icmp_seq=1 ttl=54 time=15.2 ms
64 bytes from google.com: icmp_seq=2 ttl=54 time=14.8 ms
64 bytes from google.com: icmp_seq=3 ttl=54 time=15.1 ms
64 bytes from google.com: icmp_seq=4 ttl=54 time=15.0 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 14.827/15.025/15.187/0.137 ms

# Ping a dirección IP
$ ping -c 3 8.8.8.8
```

## 4. traceroute - Rastrear Ruta de Paquetes

Muestra la ruta que toman los paquetes hasta el destino.

```bash
traceroute [opciones] host
```

**Instalación:**
```bash
sudo apt install traceroute
```

**Ejemplo:**
```bash
$ traceroute google.com
traceroute to google.com (172.217.168.46), 30 hops max, 60 byte packets
 1  router.local (192.168.1.1)  1.234 ms  1.123 ms  1.089 ms
 2  10.0.0.1 (10.0.0.1)  5.456 ms  5.432 ms  5.401 ms
 3  isp-gateway.net (203.0.113.1)  12.345 ms  12.234 ms  12.123 ms
...
```

## 5. netstat - Estadísticas de Red (deprecado)

Muestra conexiones de red, tablas de enrutamiento, estadísticas.

**Nota:** En sistemas modernos, usar `ss` en su lugar.

```bash
netstat [opciones]
```

**Opciones comunes:**
- `netstat -tuln` : Puertos TCP/UDP escuchando
- `netstat -tan` : Todas las conexiones TCP
- `netstat -r` : Tabla de enrutamiento

**Ejemplos:**
```bash
$ netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN

$ netstat -an | grep ESTABLISHED
```

## 6. ss - Socket Statistics (moderno)

Reemplazo moderno de netstat, más rápido y con más información.

```bash
ss [opciones]
```

**Opciones comunes:**
- `ss -tuln` : Puertos TCP/UDP escuchando
- `ss -tan` : Todas las conexiones TCP
- `ss -s` : Resumen de estadísticas

**Ejemplos:**
```bash
$ ss -tuln
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:Port
tcp   LISTEN 0      128    0.0.0.0:22           0.0.0.0:*
tcp   LISTEN 0      128    127.0.0.1:3306       0.0.0.0:*

$ ss -tan state established
$ ss -s  # Resumen de estadísticas
```

## 7. nslookup y dig - Consultas DNS

### nslookup - Consulta DNS básica
```bash
nslookup dominio [servidor_dns]
```

**Ejemplos:**
```bash
$ nslookup google.com
Server:     8.8.8.8
Address:    8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Address: 172.217.168.46

$ nslookup google.com 8.8.8.8
```

### dig - Consulta DNS avanzada
```bash
dig [opciones] dominio [tipo_registro]
```

**Instalación:**
```bash
sudo apt install dnsutils
```

**Ejemplos:**
```bash
$ dig google.com

$ dig google.com MX  # Registros de correo
$ dig google.com NS  # Servidores de nombres
$ dig +short google.com  # Respuesta corta
$ dig -x 8.8.8.8  # Búsqueda inversa
```

## 8. wget - Descargar Archivos

Descarga archivos desde la web.

```bash
wget [opciones] URL
```

**Opciones comunes:**
- `wget -O archivo` : Guarda con nombre específico
- `wget -c` : Continúa descarga interrumpida
- `wget -b` : Descarga en background
- `wget -r` : Descarga recursiva (sitios completos)

**Ejemplos:**
```bash
# Descargar archivo
$ wget https://ejemplo.com/archivo.zip

# Guardar con nombre diferente
$ wget -O mi_archivo.zip https://ejemplo.com/archivo.zip

# Continuar descarga interrumpida
$ wget -c https://ejemplo.com/archivo_grande.iso

# Descargar en background
$ wget -b https://ejemplo.com/archivo.zip
```

## 9. curl - Transferir Datos

Herramienta para transferir datos con URLs.

```bash
curl [opciones] URL
```

**Opciones comunes:**
- `curl -O` : Guarda con nombre original
- `curl -o archivo` : Guarda con nombre específico
- `curl -I` : Solo muestra headers
- `curl -L` : Sigue redirecciones

**Ejemplos:**
```bash
# Ver contenido de URL
$ curl https://ejemplo.com

# Descargar archivo
$ curl -O https://ejemplo.com/archivo.zip
$ curl -o nuevo_nombre.zip https://ejemplo.com/archivo.zip

# Ver solo headers HTTP
$ curl -I https://ejemplo.com

# Hacer petición POST
$ curl -X POST -d "param=value" https://api.ejemplo.com

# Autenticación
$ curl -u usuario:contraseña https://ejemplo.com
```

## 10. scp - Copiar Archivos por SSH

Copia archivos entre hosts de forma segura.

```bash
scp [opciones] origen destino
```

**Ejemplos:**
```bash
# Copiar archivo local a servidor remoto
$ scp archivo.txt usuario@servidor:/ruta/destino/

# Copiar desde servidor remoto a local
$ scp usuario@servidor:/ruta/archivo.txt ./

# Copiar directorio recursivamente
$ scp -r carpeta/ usuario@servidor:/ruta/destino/

# Usar puerto SSH diferente
$ scp -P 2222 archivo.txt usuario@servidor:/destino/
```

## 11. rsync - Sincronizar Archivos

Sincroniza archivos y directorios eficientemente.

```bash
rsync [opciones] origen destino
```

**Opciones comunes:**
- `rsync -a` : Modo archivo (preserva permisos, etc.)
- `rsync -v` : Verbose
- `rsync -z` : Compresión
- `rsync --delete` : Elimina archivos en destino que no están en origen

**Ejemplos:**
```bash
# Sincronizar directorios locales
$ rsync -av /origen/ /destino/

# Sincronizar a servidor remoto
$ rsync -avz /local/ usuario@servidor:/remoto/

# Sincronizar con progreso
$ rsync -avz --progress /origen/ /destino/

# Sincronizar y eliminar archivos extra
$ rsync -av --delete /origen/ /destino/
```

## 12. hostname - Nombre del Host

Muestra o establece el nombre del host.

```bash
hostname [opciones]
```

**Ejemplos:**
```bash
$ hostname
mi-servidor

$ hostname -I  # Muestra IPs
192.168.1.100

$ hostname -f  # Nombre completo (FQDN)
mi-servidor.ejemplo.com
```

## 13. whois - Información de Dominio

Obtiene información sobre dominios registrados.

```bash
whois dominio
```

**Instalación:**
```bash
sudo apt install whois
```

**Ejemplo:**
```bash
$ whois google.com
```

## 14. Configuración de Firewall - ufw

Ubuntu Firewall - interfaz simple para iptables.

```bash
sudo ufw [comando]
```

**Comandos comunes:**
```bash
# Ver estado
$ sudo ufw status

# Habilitar firewall
$ sudo ufw enable

# Deshabilitar firewall
$ sudo ufw disable

# Permitir puerto
$ sudo ufw allow 22
$ sudo ufw allow ssh
$ sudo ufw allow 80/tcp

# Denegar puerto
$ sudo ufw deny 23

# Eliminar regla
$ sudo ufw delete allow 80

# Ver reglas numeradas
$ sudo ufw status numbered
```

## Ejercicios Prácticos

1. Usa `ip a` para ver tus interfaces de red
2. Prueba la conectividad a google.com con `ping`
3. Usa `dig` para consultar registros DNS de un dominio
4. Descarga un archivo con `wget` y otro con `curl`
5. Examina las conexiones activas con `ss -tan`
6. Practica `scp` copiando archivos entre sistemas
7. Usa `traceroute` para ver la ruta a un servidor remoto

## Comandos Útiles Combinados

```bash
# Ver qué proceso está usando un puerto
$ sudo ss -tulpn | grep :80
$ sudo netstat -tulpn | grep :80

# Verificar conectividad y DNS
$ ping -c 3 google.com && dig google.com

# Descargar y descomprimir
$ wget https://ejemplo.com/archivo.tar.gz && tar xzf archivo.tar.gz
```
