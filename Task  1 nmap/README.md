# Task 1: Basic Network Scanning with Nmap

## Internship: Oasis Infobyte — Security Analyst Internship
**Level:** Beginner  
**Tool Used:** Nmap 7.98    

---

## Objective

Perform a network scan to identify open ports and services on a target machine using Nmap, document all findings, and explain the significance of each open port.

---

## Lab Environment

| Component        | Details                              |
|-----------------|--------------------------------------|
| Attacker Machine | Kali Linux                          |
| Target Machine   | Metasploitable 2                    |
| Target IP        | 192.168.56.101                      |
| Network Type     | Host-Only Network (VirtualBox)      |
| Target OS        | Linux (Ubuntu) — metasploitable.localdomain |

---

## Command Used

```bash
nmap -sV 192.168.56.101 -oN nmap_scan_results.txt
```

**Flags explained:**
- `-sV` — Probe open ports to determine service and version information
- `-oN` — Save output to a normal text file

---

## Scan Summary

| Metric              | Value              |
|--------------------|-------------------|
| Scan Start Time     | 15:07:45 (GMT-4)  |
| Scan End Time       | 15:08:02 (GMT-4)  |
| Duration            | 17.12 seconds     |
| Total Ports Scanned | 1000              |
| Filtered Ports      | 977               |
| Open Ports Found    | 23                |
| Host Status         | Up (0.0077s latency) |

---

## Open Ports & Services — Detailed Findings

### Port 21 — FTP (vsftpd 2.3.4)
**Risk: CRITICAL**

vsftpd 2.3.4 contains a known backdoor (CVE-2011-2523) introduced into the source code in 2011. Triggering the backdoor opens a root shell on port 6200. This is one of the most well-known Metasploitable vulnerabilities and can be exploited in seconds using Metasploit.

**Significance:** Allows complete unauthenticated root access to the system.

---

### Port 22 — SSH (OpenSSH 4.7p1 Debian 8ubuntu1)
**Risk: MEDIUM**

SSH is the standard secure remote administration protocol. However, OpenSSH 4.7p1 is severely outdated (released ~2007). Older versions contain known vulnerabilities. Weak credentials could also allow brute-force attacks.

**Significance:** Remote administration channel — should be secured with key-based authentication and updated.

---

### Port 23 — Telnet (Linux telnetd)
**Risk: CRITICAL**

Telnet transmits all data — including usernames, passwords, and commands — in complete plaintext. Any attacker on the network can capture the entire session using Wireshark. Telnet has been obsolete since SSH became widely available in the late 1990s.

**Significance:** Any credentials entered over Telnet are fully visible on the network.

---

### Port 25 — SMTP (Postfix smtpd)
**Risk: MEDIUM**

The Simple Mail Transfer Protocol (SMTP) is used to send emails. An open SMTP server can be abused as an open relay — allowing attackers to send spam or phishing emails through the server. Also susceptible to user enumeration via VRFY/EXPN commands.

**Significance:** May be exploited for email relay abuse or information disclosure.

---

### Port 53 — DNS (ISC BIND 9.4.2)
**Risk: MEDIUM**

ISC BIND 9.4.2 is an outdated DNS server version. Older BIND versions are vulnerable to DNS cache poisoning, zone transfer attacks, and remote exploits. An open DNS port also enables DNS amplification attacks.

**Significance:** Outdated DNS server vulnerable to cache poisoning and zone transfers.

---

### Port 80 — HTTP (Apache httpd 2.2.8 Ubuntu DAV/2)
**Risk: HIGH**

Apache 2.2.8 is end-of-life and contains multiple known CVEs. The server hosts DVWA and Mutillidae — intentionally vulnerable web applications. WebDAV (DAV/2) is also enabled, which can allow unauthorized file uploads if misconfigured.

**Significance:** Web attack surface including SQLi, XSS, CSRF, command injection, and file upload vulnerabilities.

---

### Port 111 — RPCbind (RPC #100000)
**Risk: MEDIUM**

RPCbind maps RPC services to network ports. It acts as a directory for NFS and other RPC services. An exposed RPCbind port allows attackers to enumerate available RPC services and can facilitate NFS exploitation.

**Significance:** Enables enumeration of NFS and other RPC services on the host.

---

### Ports 139 & 445 — NetBIOS / SMB (Samba smbd 3.X–4.X)
**Risk: HIGH**

Samba provides Windows file and print sharing on Linux. Older Samba versions (including 3.x) are vulnerable to well-known exploits such as MS17-010 EternalBlue-style attacks and the Samba "username map script" RCE vulnerability (CVE-2007-2447) which is a classic Metasploitable exploit providing remote root access.

**Significance:** Remote code execution via Samba username map script vulnerability.

---

### Port 512 — rexec (netkit-rsh rexecd)
**Risk: CRITICAL**

The rexec service allows remote execution of commands using a username and password. It transmits credentials in plaintext and has no modern security features. This service should not exist on any modern system.

**Significance:** Unauthenticated remote command execution with credential exposure.

---

### Port 513 — rlogin (OpenBSD/Solaris rlogind)
**Risk: CRITICAL**

rlogin allows remote login based on IP address trust relationships, with no strong authentication. An attacker can exploit trust relationships or use IP spoofing to gain access without a password.

**Significance:** Authentication bypass via IP trust — no password required from trusted hosts.

---

### Port 514 — rsh (Netkit rshd)
**Risk: CRITICAL**

Remote Shell (rsh) executes commands on a remote machine. Like rlogin, it relies on IP-based trust with no encryption. All three r-services (512, 513, 514) together represent a severe legacy attack surface.

**Significance:** Unauthenticated remote command execution — no encryption, no modern auth.

---

### Port 1099 — Java RMI (GNU Classpath grmiregistry)
**Risk: CRITICAL**

The Java RMI registry with default configuration allows loading of classes from remote URLs, enabling Remote Code Execution. This is directly exploitable via Metasploit's `java_rmi_server` module.

**Significance:** Remote code execution by supplying malicious Java class to the RMI registry.

---

### Port 1524 — Bindshell (Metasploitable root shell)
**Risk: CRITICAL**

This is an intentionally planted backdoor — a shell bound directly to port 1524. Connecting with netcat (`nc 192.168.56.101 1524`) immediately returns a root shell with no authentication whatsoever.

**Significance:** Instant unauthenticated root access — most dangerous single port on the system.

---

### Port 2049 — NFS (RPC #100003)
**Risk: HIGH**

Network File System (NFS) allows remote filesystem mounting. If exports are misconfigured (e.g., `*(rw,no_root_squash)`), an attacker can mount the target's filesystem and read or modify any file, including `/etc/passwd` and SSH keys.

**Significance:** Unauthorized filesystem access if NFS exports are misconfigured.

---

### Port 2121 — FTP (ProFTPD 1.3.1)
**Risk: HIGH**

A second FTP service running ProFTPD 1.3.1. ProFTPD 1.3.1 has known vulnerabilities including SQL injection in the mod_sql module. FTP also transmits credentials in plaintext.

**Significance:** Secondary FTP attack surface with known CVEs and plaintext credential exposure.

---

### Port 3306 — MySQL (MySQL 5.0.51a-3ubuntu5)
**Risk: HIGH**

MySQL is bound to all network interfaces and accessible remotely. MySQL 5.0.51a is end-of-life. Combined with the SQL injection vulnerability in DVWA (demonstrated in Task 9), full database credential dumps are achievable. Remote MySQL access with weak credentials is a critical risk.

**Significance:** Database credential exposure and potential OS-level access via `INTO OUTFILE`.

---

### Port 5432 — PostgreSQL (8.3.0–8.3.7)
**Risk: HIGH**

PostgreSQL 8.3.x is severely outdated and bound to the network interface. Known vulnerabilities include CVE-2014-3566 (SSL POODLE) and CVE-2014-0224 (OpenSSL CCS Injection), both confirmed by Nmap vuln scripts in Task 10.

**Significance:** Remote database access with multiple SSL/TLS vulnerabilities.

---

### Port 5900 — VNC (protocol 3.3)
**Risk: CRITICAL**

VNC protocol 3.3 supports no-authentication mode. An attacker can connect using any VNC client and obtain full graphical desktop control of the target with zero credentials required.

**Significance:** Full graphical remote desktop access without authentication.

---

### Port 6000 — X11 (access denied)
**Risk: HIGH**

X11 is the Linux graphical display server. Even with "access denied" returned, an open X11 port indicates the display server is network-accessible. If access controls are misconfigured, attackers can capture keystrokes, screenshots, and inject input into graphical sessions.

**Significance:** Potential graphical session hijacking if X11 access controls fail.

---

### Port 6667 — IRC (UnrealIRCd — backdoored)
**Risk: CRITICAL**

The IRC service is running a trojaned version of UnrealIRCd that was distributed with a backdoor in 2010. Sending a specially crafted string triggers a remote command execution as the IRC user. Exploitable via Metasploit.

**Significance:** Unauthenticated remote code execution via IRC backdoor.

---

### Port 8009 — AJP13 (Apache Jserv Protocol v1.3)
**Risk: HIGH**

The AJP connector for Apache Tomcat is exposed. AJP is vulnerable to the "Ghostcat" vulnerability (CVE-2020-1938) which allows file read and potential remote code execution via crafted AJP requests.

**Significance:** File read and potential RCE via AJP Ghostcat attack.

---

### Port 8180 — HTTP (Apache Tomcat/Coyote JSP 1.1)
**Risk: HIGH**

Apache Tomcat is running on port 8180 with a management interface at `/manager/html`. Default Tomcat credentials (`tomcat:tomcat`) often grant access to the manager, allowing WAR file deployment which leads to remote code execution.

**Significance:** RCE via Tomcat manager WAR deployment with default credentials.

---

## Port Significance Summary Table

| Port | Service | Version | Risk | Key Threat |
|------|---------|---------|------|-----------|
| 21 | FTP | vsftpd 2.3.4 | CRITICAL | Backdoor RCE (CVE-2011-2523) |
| 22 | SSH | OpenSSH 4.7p1 | MEDIUM | Outdated, brute-force risk |
| 23 | Telnet | Linux telnetd | CRITICAL | Plaintext credential exposure |
| 25 | SMTP | Postfix | MEDIUM | Open relay, user enumeration |
| 53 | DNS | BIND 9.4.2 | MEDIUM | Zone transfer, cache poisoning |
| 80 | HTTP | Apache 2.2.8 | HIGH | SQLi, XSS, CSRF, command exec |
| 111 | RPCbind | RPC #100000 | MEDIUM | NFS/RPC service enumeration |
| 139 | NetBIOS | Samba 3.X-4.X | HIGH | Samba RCE (CVE-2007-2447) |
| 445 | SMB | Samba 3.X-4.X | HIGH | Samba RCE (CVE-2007-2447) |
| 512 | rexec | netkit-rsh | CRITICAL | Unauthenticated remote exec |
| 513 | rlogin | rlogind | CRITICAL | IP trust bypass |
| 514 | rsh | Netkit rshd | CRITICAL | Unauthenticated remote shell |
| 1099 | Java RMI | grmiregistry | CRITICAL | Remote class loading RCE |
| 1524 | Bindshell | Root shell | CRITICAL | Instant root — no auth |
| 2049 | NFS | RPC #100003 | HIGH | Filesystem mount abuse |
| 2121 | FTP | ProFTPD 1.3.1 | HIGH | Known CVEs, plaintext creds |
| 3306 | MySQL | 5.0.51a | HIGH | Remote DB access, data dump |
| 5432 | PostgreSQL | 8.3.0-8.3.7 | HIGH | SSL POODLE, CCS Injection |
| 5900 | VNC | protocol 3.3 | CRITICAL | No-auth graphical access |
| 6000 | X11 | — | HIGH | Display session hijacking |
| 6667 | IRC | UnrealIRCd | CRITICAL | Backdoor RCE |
| 8009 | AJP13 | Jserv v1.3 | HIGH | Ghostcat file read/RCE |
| 8180 | HTTP | Tomcat 1.1 | HIGH | WAR deployment RCE |

---

## Key Observations

The target (Metasploitable 2) exposes **23 open ports** presenting a massively over-exposed attack surface. Critical findings include:

- **Multiple backdoors** — vsftpd 2.3.4 (port 21), UnrealIRCd (port 6667), and a direct root bindshell (port 1524)
- **Legacy plaintext protocols** — Telnet (23), rexec/rlogin/rsh (512–514) transmit everything unencrypted
- **Outdated software stacks** — Every service version is end-of-life with known public CVEs
- **No firewall** — All 23 ports are directly reachable with no filtering

---

## Recommendations

1. **Disable all unnecessary services** — Only SSH (22) and HTTP/HTTPS (80/443) should be exposed
2. **Replace Telnet with SSH** — Never use Telnet on any production system
3. **Remove r-services** — rexec, rlogin, rsh are obsolete with no legitimate use case
4. **Update all software** — Every service version on this system is critically outdated
5. **Implement a firewall** — Use UFW to default-deny and whitelist only required ports
6. **Bind database services to localhost** — MySQL and PostgreSQL must not be network-accessible

---

## Repository Structure

```
Task1_Nmap_Scan/
├── nmap_scan_results.txt   # Raw Nmap -sV output
├── screenshots/
│   └── nmap_scan_results.jpeg  # Terminal screenshot of scan
└── README.md               # This report
```

---

## Disclaimer

This scan was performed in a controlled lab environment on an intentionally vulnerable machine (Metasploitable 2) connected via VirtualBox Host-Only Network, solely for educational purposes as part of the Oasis Infobyte Security Analyst Internship.
