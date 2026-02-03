# HackTheBox: Delivery - WriteUp Detallado 🚩

##  Información del Laboratorio
* **Target IP:** 10.129.10.53
* **Attacker IP:** 10.10.14.93
* **Dificultad:** Fácil 
* **OS:** Linux 🐧 (Identificado por TTL=63)
* **Carpeta:** `HTB/Delivery`

---

## 1. Fase de Reconocimiento
Iniciamos verificando la conexión con el objetivo mediante `ping`. El valor de **TTL=63** nos confirma que el sistema operativo base es **Linux**.

### Escaneo de Puertos (Nmap)
Realizamos un escaneo completo de puertos para identificar servicios y versiones:

```bash
nmap -sC -sV -p- 10.129.10.53 -oN nmap_scan.txt
```

## 2. Acceso Inicial (Explotación Web)



### Vulnerabilidad Lógica en el Registro
El servidor de chat **Mattermost** solo permite registros con correos del dominio `@delivery.htb`. Para saltarnos esto, usamos el servicio de soporte en `helpdesk.delivery.htb`:

1.  **Creación de Ticket:** Al abrir un ticket, el sistema nos asigna una dirección de correo temporal del tipo `ID_TICKET@delivery.htb`.
2.  **Registro:** Utilizamos ese correo para crear nuestra cuenta en Mattermost.
3.  **Activación:** Accedemos al portal de ayuda para leer el ticket, donde recibimos el email de verificación y activamos la cuenta.

### Movimiento Lateral
Dentro de Mattermost, exploramos el canal **#Internal** y encontramos credenciales compartidas:

* **Usuario:** `maildeliverer`
* **Password:** `Youve_G0t_Mail!`

Accedemos por SSH:
```bash
ssh maildeliverer@10.129.10.53
```

---

## 3. Escalada de Privilegios a Root

### Análisis de Base de Datos
Revisamos el archivo de configuración en `/opt/mattermost/config/config.json` para obtener las credenciales de la base de datos MySQL. Una vez dentro de la DB, extraemos el hash del usuario root de Mattermost:

* **Hash (BCrypt):** `$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO`



### Cracking con Hashcat (Rule-Based Attack)
Aprovechamos la pista "PleaseSubscribe!" encontrada en el chat y aplicamos reglas de mutación para crackear el hash:

```bash
echo "PleaseSubscribe!" > word.txt
hashcat -m 3200 root_hash.txt word.txt -r /usr/share/hashcat/rules/best64.rule
```

**Resultado:** Se obtuvo la contraseña `PleaseSubscribe!21`, permitiendo el acceso total como **root** mediante `su root` o SSH.

---

## Lecciones Aprendidas

1. **Seguridad en Comunicaciones Internas:** Nunca se deben compartir credenciales en texto plano a través de plataformas de chat (Mattermost), ya que cualquier compromiso de cuenta o de base de datos expone el acceso a otros sistemas.
2. **Políticas de Contraseñas:** El uso de una palabra base con variaciones predecibles (como añadir años o números al final) es vulnerable a ataques de reglas (*Rule-based attacks*) en herramientas como Hashcat.
3. **Validación de Identidad:** Los flujos de registro que dependen de correos electrónicos internos deben asegurar que dichos correos no puedan ser generados de forma trivial por usuarios externos (como ocurrió con el sistema de tickets).

---

> **Nota:** Este laboratorio fue realizado en un entorno controlado con fines educativos en la plataforma HackTheBox.











