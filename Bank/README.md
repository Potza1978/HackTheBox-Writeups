#HTB: Bank - Write-up (Road to eJPT v2) 🏦

**Bank** es una máquina Linux de nivel "Easy".
Este laboratorio es fundamental para entender la importancia de la enumeración web profunda y la identificación de configuraciones SUID inseguras.

---

## 🕵️ 1. Enumeración (Reconnaissance)

### Escaneo de Puertos
El análisis inicial con `nmap` reveló los siguientes servicios:
* **TCP/22 (SSH):** OpenSSH 6.6.1p1
* **TCP/53 (DNS):** ISC BIND 9.9.5
* **TCP/80 (HTTP):** Apache httpd 2.4.7

### Descubrimiento Web (Fuzzing)
Utilicé herramientas de enumeración de directorios para localizar rutas ocultas.
Se identificó el directorio `/balance-transfer/` con el listado de directorios habilitado.

Para gestionar el gran volumen de archivos `.acc` encontrados, ejecuté un script de **Python** para filtrar por tamaño de archivo.
Esto permitió localizar credenciales de acceso válidas para el panel de usuario.

---

## 🕸️ 2. Explotación (Initial Access)

### Vulnerabilidad de File Upload
Dentro del panel de soporte, identifiqué un formulario de subida de archivos.

El servidor implementaba una "lista negra" que bloqueaba la extensión `.php`.
Sin embargo, la configuración del servidor permitía procesar archivos `.htb` como scripts ejecutables.

**Pasos de explotación:**
1. Preparación de una **Reverse Shell** en PHP.
2. Cambio de extensión de `shell.php` a `shell.htb`.
3. Ejecución del archivo mediante una petición web tras levantar un listener con Netcat:
   ```bash
   nc -nlvp 4444
   ```

## 3. Escalada de Privilegios (Root)
### Análisis de Permisos SUID
Tras estabilizar la shell, realicé una búsqueda de binarios con el bit SUID configurado para identificar posibles vectores de elevación de privilegios:

Bash

find / -perm -4000 -type f 2>/dev/null

Hallazgo: El binario /var/htb/bin/emergency permite obtener una shell de administrador sin requerir autenticación adicional, 
debido a una configuración insegura que hereda los permisos del propietario (root).


Lecciones Críticas para eJPT v2
Configuración Web: Deshabilitar Options Indexes en Apache para evitar fugas de información (Information Disclosure).

Validación de Archivos: No confiar en filtros de extensión basados en nombres; usar validación de "Magic Bytes" o Listas Blancas estrictas.

Seguridad del Sistema: Auditar regularmente los archivos con permisos SUID para prevenir Privilege Escalation.

Flags:

User: 384d44e886affeffcffd3558437df7d4

Root: e71f6a0d0dacb6dda397b9ad96029794

