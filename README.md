# Kali Linux

Entorno Kali Linux completo accesible vía escritorio web (KasmVNC). Incluye todas las herramientas de pentesting y análisis de seguridad.

## Características

- 🖥️ **Escritorio completo**: XFCE accesible desde navegador
- 🛠️ **Herramientas preinstaladas**: Nmap, Metasploit, Burp Suite, Wireshark, etc.
- 🔒 **Aislado**: Contenedor Docker independiente
- 💾 **Persistente**: Datos y configuración en volumen Docker
- 🌐 **Acceso web**: KasmVNC en puerto 3000 (HTTP) o 3001 (HTTPS)

## ⚠️ ADVERTENCIAS DE SEGURIDAD

**CRÍTICO**: Kali Linux contiene herramientas de pentesting que pueden ser peligrosas si se exponen incorrectamente.

### Recomendaciones obligatorias:

1. **NUNCA expongas directamente a Internet sin protección**
3. **Cambia la contraseña por defecto** inmediatamente
4. **Considera usar VPN/Wireguard** para acceso remoto seguro
5. **Revisa logs regularmente** para detectar accesos no autorizados
6. **Usa solo en redes confiables** o totalmente aisladas

## Requisitos Previos

- Docker Engine instalado
- **Mínimo 2GB RAM** (4GB recomendado para uso intensivo)
- **Contraseña generada**: KALI_PASSWORD

## Generar Contraseña

```bash
# KALI_PASSWORD (acceso web)
openssl rand -base64 32
```

Guarda el resultado, lo necesitarás en el archivo `.env`.

---

## Despliegue con Docker Compose

### 1. Crear Directorio y Archivos

```bash
# Crear directorio
mkdir kali-linux
cd kali-linux
```

### 2. Crear compose.yaml

Crea el archivo `compose.yaml`:

```yaml
services:
  kali-linux:
    container_name: kali-linux
    image: lscr.io/linuxserver/kali-linux:latest
    restart: unless-stopped
    security_opt:
      - seccomp:unconfined
    environment:
      PUID: 1000
      PGID: 1000
      TZ: Europe/Madrid
      CUSTOM_USER: ${KALI_USER:-admin}
      PASSWORD: ${KALI_PASSWORD}
      CUSTOM_RES: ${CUSTOM_RES:-1920x1080}
    volumes:
      - kali_config:/config
    shm_size: "1gb"

volumes:
  kali_config:
    name: kali_config
    
# añadir estas líneas al final del archivo para proxy inverso 
networks:
  default:
    external: true
    name: proxy
```

### 3. Configurar Variables de Entorno

Crea el archivo `.env`:

```env
# Usuario y contraseña
KALI_USER=admin
KALI_PASSWORD=password_generado_anteriormente

DOMAIN_HOST=kali.tudominio.com
```



```yaml
services:
  kali-linux:
    labels:
```


```yaml
http:
  middlewares:
    ip-allowlist:
      ipAllowList:
        sourceRange:
          - "127.0.0.1/32"      # Localhost
          - "10.0.0.0/8"        # Red privada clase A
          - "172.16.0.0/12"     # Red privada clase B
          - "192.168.0.0/16"    # Red privada clase C
```

### 5. Desplegar

```bash
# Crear red proxy si no existe
docker network create proxy

# Iniciar servicios
docker compose up -d

# Ver logs
docker compose logs -f kali-linux
```

Deberías ver:
```
[init] done.
```

---

## Método Alternativo: Clonar desde Git

Si prefieres usar Git para mantener la configuración actualizada:

```bash
# Clonar repositorio
git clone https://git.ictiberia.com/groales/kali-linux.git
cd kali-linux

# Copiar y editar variables
cp .env.example .env
nano .env


# Desplegar
docker network create proxy
docker compose up -d
```

---

## Primer Acceso


Accede a: `https://kali.tudominio.com`

### Acceso directo (sin proxy)

- **HTTP**: `http://IP_SERVIDOR:3000`
- **HTTPS**: `https://IP_SERVIDOR:3001`

### Credenciales de acceso

- **Usuario**: El configurado en `KALI_USER` (por defecto: `admin`)
- **Contraseña**: La configurada en `KALI_PASSWORD`

### En el escritorio

Una vez dentro del escritorio web:
- Usuario del sistema: `abc` (para terminal y permisos)
- Contraseña del sistema: La misma de `KALI_PASSWORD`

---

## Uso y Configuración

### Herramientas principales disponibles

Kali Linux incluye cientos de herramientas. Las más utilizadas:

**Escaneo y reconocimiento:**
```bash
nmap -sV -A target.com
masscan -p1-65535 10.0.0.0/8 --rate=1000
```

**Análisis de vulnerabilidades:**
```bash
nikto -h http://target.com
sqlmap -u "http://target.com/page.php?id=1" --dbs
```

**Explotación:**
```bash
msfconsole
searchsploit apache 2.4
```

**Análisis de red:**
```bash
wireshark
tcpdump -i eth0 -w capture.pcap
```

**Cracking:**
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```

**Web:**
```bash
burpsuite
zaproxy
```

### Instalar herramientas adicionales

```bash
# Actualizar repositorios
sudo apt update

# Instalar herramienta específica
sudo apt install nombre-herramienta

# Ver herramientas disponibles
apt-cache search kali-tools
```

### Persistencia de datos

Todo lo guardado en `/config` persiste entre reinicios:
- `/config/.config` - Configuración de aplicaciones
- `/config/Desktop` - Escritorio del usuario
- `/config/Documents` - Documentos
- `/config/Downloads` - Descargas

### Copiar archivos al contenedor

```bash
# Desde host a contenedor
docker cp archivo.txt kali-linux:/config/Documents/

# Desde contenedor a host
docker cp kali-linux:/config/Documents/reporte.pdf ./
```

---

## Configuración Avanzada

### Cambiar resolución del escritorio

**Problema común en Mac**: Las pantallas Retina detectan resoluciones muy altas (4464x2008) que hacen que todo se vea muy pequeño.

**Solución**: Fija una resolución específica editando las variables de entorno.


```env
CUSTOM_RES=1920x1080
```

O edita `compose.yaml`:

```yaml
environment:
  CUSTOM_RES: 1920x1080  # Recomendada para Mac
```

**Resoluciones recomendadas para Mac:**
- `1920x1080` - Full HD (óptima para la mayoría)
- `1680x1050` - MacBook Pro 15" (2015 y anteriores)
- `1440x900` - MacBook Air 13"
- `2560x1440` - QHD (si tienes pantalla grande)
- `1280x720` - HD (si la conexión es lenta)

Después de cambiar, reinicia el stack:
```bash
docker restart kali-linux
```

### Habilitar audio

Añade en `compose.yaml`:

```yaml
devices:
  - /dev/snd:/dev/snd
environment:
  PULSE_SERVER: unix:/run/user/1000/pulse/native
```

### Usar teclado en otro idioma

```yaml
environment:
  KEYBOARD: es  # Teclado español
```

### Más memoria compartida (para navegadores)

```yaml
shm_size: "2gb"  # Por defecto: 1gb
```

---

## Backup y Restauración

### Backup del volumen

```bash
# Detener contenedor
docker compose down

# Backup
docker run --rm \
  -v kali_config:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/kali-backup-$(date +%Y%m%d).tar.gz -C /data .

# Reiniciar
docker compose up -d
```

### Restauración

```bash
# Detener contenedor
docker compose down

# Restaurar
docker run --rm \
  -v kali_config:/data \
  -v $(pwd):/backup \
  alpine sh -c "cd /data && tar xzf /backup/kali-backup-YYYYMMDD.tar.gz"

# Reiniciar
docker compose up -d
```

---

## Actualización

### Actualizar contenedor

```bash
# Detener
docker compose down

# Actualizar imagen
docker compose pull

# Reiniciar
docker compose up -d
```

### Actualizar herramientas dentro de Kali

Desde el terminal en el escritorio web:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
```

---

## Solución de Problemas

### El escritorio no carga

**Verificar logs:**
```bash
docker compose logs kali-linux | tail -50
```

**Reiniciar contenedor:**
```bash
docker compose restart kali-linux
```

**Verificar memoria:**
```bash
docker stats kali-linux
```

Si usa >90% de RAM, aumenta recursos en Docker Desktop o servidor.

### Contraseña incorrecta

**Cambiar contraseña:**

1. Edita `.env`:
```env
KALI_PASSWORD=nueva_password
```

2. Reinicia:
```bash
docker compose down
docker compose up -d
```

### Herramientas no funcionan

**Verificar `seccomp:unconfined`:**

Algunas herramientas necesitan esta opción en `compose.yaml`:
```yaml
security_opt:
  - seccomp:unconfined
```

**Verificar permisos:**
```bash
docker exec -it kali-linux bash
ls -la /config
```

### No puedo copiar/pegar

**Habilitar portapapeles:**

En el escritorio web, verifica:
1. Settings → Clipboard
2. Activa "Enable clipboard"

### Lento o se congela

**Aumentar recursos:**
```yaml
shm_size: "2gb"
```

**Verificar CPU:**
```bash
docker stats kali-linux
```

**Reducir herramientas ejecutándose:**
Cierra aplicaciones no usadas en el escritorio.


**Verificar middleware IP allowlist:**
```bash
```

**Verificar red proxy:**
```bash
docker network inspect proxy
```

```bash
```

---

## Notas de Seguridad

### Aislamiento de red

Para máximo aislamiento, crea una red dedicada:

```yaml
services:
  kali-linux:
    networks:
      - kali_isolated

networks:
  kali_isolated:
    internal: true  # Sin acceso a Internet
  proxy:
    external: true
```

### Logging y auditoría

Monitorea accesos:
```bash
docker compose logs -f kali-linux | grep -i login
```

### Actualizaciones de seguridad

Mantén actualizado:
- **Imagen Docker**: `docker compose pull` mensualmente
- **Sistema operativo**: `sudo apt update && sudo apt upgrade` semanalmente
- **Herramientas**: `sudo apt dist-upgrade` antes de auditorías importantes

---

## Variables de Entorno

| Variable | Requerido | Por Defecto | Descripción |
|----------|-----------|-------------|-------------|
| `PUID` | No | `1000` | User ID para permisos |
| `PGID` | No | `1000` | Group ID para permisos |
| `TZ` | No | `Europe/Madrid` | Zona horaria |
| `CUSTOM_USER` | No | `abc` | Usuario personalizado |
| `PASSWORD` | **Sí** | - | Contraseña de acceso web |
| `CUSTOM_RES` | No | `1280x720` | Resolución del escritorio. Mac Retina: usar `1920x1080` |
| `KEYBOARD` | No | `en-us-qwerty` | Layout del teclado |
| `TITLE` | No | `Kali Linux` | Título de la ventana |

---

## Recursos Adicionales

- 📖 [Documentación oficial Kali Linux](https://www.kali.org/docs/)
- 🐳 [Imagen LinuxServer.io](https://docs.linuxserver.io/images/docker-kali-linux)
- 🛡️ [Kali Tools](https://www.kali.org/tools/)
- 📚 [Kali Training](https://kali.training/)

---

## Licencia

- **Kali Linux**: GPL-3.0
- **Esta configuración**: MIT

---

## Soporte

Para issues con:
- **Imagen Docker**: [LinuxServer.io Discord](https://discord.gg/YWrKVTn)
- **Kali Linux**: [Kali Forums](https://forums.kali.org/)
- **Esta configuración**: Abre un issue en Gitea
