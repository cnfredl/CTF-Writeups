## Machine: Filethingies

**Platform:** echoCTF 
**Difficulty:** Advanced 
**OS:** Linux 
**Points:** 5200
**Date:** 17/04/2026

---

## Summary

La resolución de esta máquina se divide en dos fases críticas. El acceso inicial se logró abusando de un gestor de archivos web (File Thingie) que presentaba una validación insuficiente en la subida de archivos; logrando evadir los filtros mediante el uso de una extensión `.phar` para obtener Ejecución Remota de Comandos (RCE). La escalada de privilegios consistió en descubrir un servicio interno (Rejetto HFS) ejecutándose como `root` en el puerto 1337. Mediante la creación de un túnel inverso por SSH (Port Forwarding), se accedió al panel de administración, el cual sufría de una grave mala configuración, exponiendo de forma pública archivos críticos del sistema como `/etc/shadow` y la bandera final.

---

## Enumeration

### 🔸 Nmap

```bash
nmap -sC -sV -p- 10.0.160.223
```

**Open Ports:**

* 22 → SSH
* 80 → HTTP

**Services:**

* Versiones relevantes

**Key Findings:**

* 
* Subida de archivos sin sanitizar

---

### 🔸 Web / Service Enumeration

**Technologies:**

* PHP 

**Directories / Endpoints:**

* `/ft2.php

**Observations:**

- El panel web descubierto funciona como un administrador de archivos básico. Se observó que el sistema carece de un filtro robusto para la sanitización de extensiones y contenido, lo que permite la carga de archivos potencialmente peligrosos directamente al servidor web.


---

## Initial Foothold

### 🔸 Vulnerability

* **Type:** File Upload, RCE
* **Location:** `/ft2.php
* **Impact:** Permite al atacante subir _payloads_ maliciosos que el servidor backend interpreta y ejecuta, otorgando acceso a la consola del sistema.

---

### 🔸 Exploitation

**Explanation:**

* Esta vulnerabilidad aunque básica sucede porque el programador no tiene formas de asegurar que el archivo no es malicioso como una lista negra para bloquear archivos PHP ni evita que el servidor lo ejecute.


---

## Shell Access

* **Method:** Reverse shell
* **Stabilization:**

```bash
script /dev/null -qc bash
ctr + z
stty raw -echo;fg
reset xterm
export TERM=xterm
```

---

## Post-Exploitation

### 🔸 Internal Enumeration

* **Users:** root, ETSCTF
* **Interesting Files:** /opt/hfs/hfs
* **Permissions:** root
* **Running Services:** Un servicio escuchando de manera local en `127.0.0.1:1337`.

---

## Privilege Escalation

### 🔸 Vector

- _Security Misconfiguration_ (Exposición de datos sensibles) combinada con _Pivoting / Local Port Forwarding_. 



---

### 🔸 Exploitation

```bash
# HOST
sudo systemctl start ssh
# www-data
ssh -R 8080:127.0.0.1:1337 cnfred@10.10.5.90
#Go to
http://127.0.0.1:8080
```

---

## Lessons Learned

* **Qué aprendí**: La importancia del _Pivoting_ básico con herramientas nativas (SSH) cuando los binarios externos (como Chisel) fallan por incompatibilidad de librerías.
* **Qué debería haber detectado antes**: Que los falsos positivos abundan en herramientas automatizadas como LinPEAS.
*  **Errores que cometí**: Casarme con la idea de que se necesitaba un _exploit_ complejo (CVE) para el binario HFS, cuando en realidad el acceso a los archivos era directo por una falla de configuración humana.

---

## Attack Pattern 

* **Type:** Insecure File Upload (Bypass de Blacklist) → RCE → Internal Port Forwarding (Pivoting) → Sensitive Data Exposure (Misconfiguration).
* **Where it appears:** - Entornos corporativos o CTFs donde se despliegan aplicaciones internas de administración o utilidades para uso exclusivo del _localhost_.
* **Key indicators:** Binarios inusuales en `/opt/` ejecutándose como `root` y escuchando en el anillo local (`127.0.0.1`).

---

## References

- Enumeración manual exhaustiva (procesos y red).
    
- _(No se requirió uso de exploits públicos ni documentación de vulnerabilidades de terceros para comprometer el sistema)._

