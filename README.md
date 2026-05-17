# MANUAL OPERATIVO Y MATRIZ DE CONTROL: KALI-TOOLS ARSENAL

Este documento detalla la arquitectura, el despliegue técnico y el catálogo de herramientas ofensivas, análisis forense y auditoría táctica de *Red Team*. El entorno está diseñado para operar de manera aislada utilizando un enfoque de **"Subsistema de Kali Linux basado en Kernel Compartido"** sobre una estación de trabajo base Ubuntu 24.04 LTS (Noble Numbat) y Debian 13 (Trixie).

---

## 1. FILOSOFÍA DE DISEÑO: EL ENFOQUE "SUBSISTEMA" VS VIRTUALIZACIÓN RÍGIDA

*   **Arquitectura de Kernel Compartido (Sin Hipervisores):** A diferencia de las máquinas virtuales tradicionales (VirtualBox/VMware) que emulan hardware virtual completo y drenan recursos críticos de CPU y RAM, este subsistema utiliza Docker para montar el espacio de usuario completo de `kali-rolling` directamente sobre el Kernel nativo de la estación de trabajo Ubuntu. El consumo de recursos en reposo es prácticamente cero.
*   **Acceso Directo al Hardware Físico (`--net=host`):** El gran talón de Aquiles de la virtualización clásica o el NAT es la incapacidad de interactuar limpiamente con el tráfico de red de bajo nivel. Al instanciar este subsistema acoplado directamente a la pila de red del host, todas las suites de hacking tienen acceso promiscuo total y nativo a las interfaces físicas (tarjetas de red ethernet, interfaces Wi-Fi, etc.), permitiendo inyección de paquetes, escaneos reales y capturas al 100% de la velocidad del hardware.
*   **Sandboxing y Resiliencia de Infraestructura:** El despliegue aísla por completo las librerías, entornos de ejecución (como Java para Ghidra) y dependencias de Python del arsenal ofensivo. Esto previene la degradación del gestor de paquetes nativo del host y mantiene intactas las políticas rígidas del cortafuegos (`ufw`) de producción de la máquina real. Si el subsistema se corrompe por herramientas inestables, puede ser destruido y regenerado en milisegundos con una sola directiva, garantizando la persistencia impecable del Host.

---

## 2. ARQUITECTURA DE FLUJO DE DATOS

```text
   +-------------------------------------------------------+

   |            UBUNTU 24.04 LTS (Host Estable)            |
   |      - Producción y Enrutamiento Doméstico            |
   |      - Políticas Rígidas de Firewall (UFW)            |
   +---------------------------+---------------------------+

                               |
            [ --net=host ]     | (Acceso directo a Interfaces Físicas
                               |  enp3s0, wlan0, mon0, etc.)
                               v
   +-------------------------------------------------------+

   |          SUBSISTEMA KALI-TOOLS (Contenedor)           |
   |      - Espacio de Usuario Aislado                     |
   |      - Arsenal Completo de Red Team                   |
   +-------------------------------------------------------+
```

---

## 3. PROCEDIMIENTO DE DESPLIEGUE QUIRÚRGICO (EN EL HOST UBUNTU)

Ejecute las siguientes directivas secuenciales en la terminal de su máquina anfitriona como usuario **root**:

### Paso 3.1: Inicializar el Motor y el Subsistema Persistente
```bash
apt update && apt install -y docker.io

docker run -d -it --net=host --name kali-tools --restart unless-stopped kalilinux/kali-rolling
```

### Paso 3.2: Crear el Comando de Acceso Rápido al Subsistema
Ejecute este bloque exacto en la terminal del host para mapear la entrada rápida:

```bash
cat << 'EOF' > /usr/local/bin/kali-tools
#!/bin/bash
# Acceder al subsistema interactivo de forma directa y limpia
docker exec -it kali-tools bash
EOF

chmod +x /usr/local/bin/kali-tools
```

---

## 4. INYECCIÓN MASIVA DEL ARSENAL TÁCTICO

Invoque su nuevo comando unificado para inicializar y entrar al subsistema:

```bash
kali-tools
```

Una vez que el prompt de su terminal cambie a la shell interna de Kali (`root@kali`), ejecute la carga masiva de los 4 metapaquetes core de manera secuencial:

```bash
apt update && apt install -y kali-tools-forensics kali-tools-information-gathering kali-tools-vulnerability kali-tools-passwords
```

> 💡 **Nota durante la instalación:** Si el gestor de paquetes solicita confirmación para `wireshark-common` (capturas sin privilegios), seleccione **YES**. Si solicita el modo de ejecución para el multiplexor `sslh`, seleccione la opción **2 (standalone)** para asegurar el máximo rendimiento de red.

Para salir del subsistema y regresar a la terminal limpia de Ubuntu, escriba:
```bash
exit
```

---

## 5. DIMENSIONAMIENTO Y REQUISITOS DE ALMACENAMIENTO

*   **Volumen de Descarga (Paquetes comprimidos):** ~3.5 GB a 4.5 GB.
*   **Espacio Consolidado en Disco (Instalado):** ~8.5 GB a 12 GB.

### METAPAQUETES TÁCTICOS DISPONIBLES:

#### 1. `kali-tools-forensics` (~2.5 GB)
*   **Especialidad:** Análisis de sistemas de archivos, forense en memoria RAM y metadatos.
*   **Herramientas críticas incluidas:**
    *   `guymager`: Adquisición e imágenes de disco bit a bit con verificación hash.
    *   `sleuthkit` (TSK): Análisis forense de sistemas de archivos a bajo nivel por comandos.
    *   `autopsy`: Interfaz gráfica de análisis forense digital y correlación de líneas de tiempo.
    *   `foremost` / `scalpel`: Extracción de archivos (*file carving*) basadas en cabeceras binarias.
    *   `volatility` / `volatility3`: Framework definitivo para análisis forense de memoria RAM.
    *   `chntpw`: Modificación directa del registro SAM de Windows para resetear credenciales.
    *   `hashdeep` / `md5deep`: Auditoría recursiva y cálculo de hashes de integridad masivos.

#### 2. `kali-tools-information-gathering` (~1.8 GB)
*   **Especialidad:** Mapeo de red avanzado, análisis de tráfico y herramientas OSINT.
*   **Herramientas críticas incluidas:**
    *   `nmap` / `ncat`: Escaneo de puertos avanzado, mapeo de topologías y persistencia.
    *   `wireshark` / `tshark`: Captura e inspección profunda de protocolos en modo promiscuo.
    *   `fping`: Barridos de ping masivos y paralelos sobre segmentos de red.
    *   `nbtscan`: Escaneo y recolección de nombres NetBIOS en redes locales.
    *   `dmitry`: Recolección de información pública de hosts (Whois, subdominios, correos).
    *   `arp-scan`: Descubrimiento y mapeo de direcciones físicas MAC en la red local.

#### 3. `kali-tools-vulnerability` (~3.2 GB)
*   **Especialidad:** Marcos de explotación de vulnerabilidades e ingeniería inversa.
*   **Herramientas críticas incluidas:**
    *   `sqlmap`: Motor automatizado para detección y explotación de inyecciones SQL.
    *   `nikto`: Escáner de servidores web para detectar scripts inseguros y malas configuraciones.
    *   `ghidra`: Suite de ingeniería inversa y descompilación de binarios (NSA).
    *   `binwalk`: Análisis y extracción de sistemas de archivos en imágenes de firmware.
    *   `lynis`: Auditoría de seguridad exhaustiva y check de *hardening* para Unix.

#### 4. `kali-tools-passwords` (~2.0 GB)
*   **Especialidad:** Crackers criptográficos por GPU/CPU y generadores de diccionarios.
*   **Herramientas críticas incluidas:**
    *   `hashcat`: Recuperador de contraseñas por fuerza bruta basado en reglas de alta velocidad.
    *   `john` (John the Ripper): Craqueador de hashes fuera de línea multihilo.
    *   `hydra` / `medusa`: Ataques de fuerza bruta paralelos en caliente (SSH, FTP, RDP, etc.).
    *   `aircrack-ng`: Suite completa de auditoría e inspección de seguridad inalámbrica Wi-Fi.
    *   `crunch`: Generador de diccionarios de contraseñas parametrizado por patrones de texto.

---

## 6. OPERATORIA EN EL DÍA A DÍA

El flujo de trabajo táctico diario queda reducido a la máxima simplicidad:
1. Abra la consola estándar en su Ubuntu.
2. Ingrese instantáneamente al entorno de auditoría escribiendo: `kali-tools`
3. Ejecute de forma directa cualquier binario nativo indexado (`nmap`, `sqlmap`, `hashcat`, etc.).
4. Finalizada la evaluación, digite `exit` para regresar de inmediato al entorno seguro de producción.

***

**Desplegado para la cuenta GitHub @matarturo**

