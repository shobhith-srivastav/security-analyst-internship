# Network Security Assessment Report
## Oasis Infobyte — Security Analyst Internship | Task 10

---

**Assessment Type:** Black-box Network Security Assessment  
**Assessor:** Oasis Infobyte Security Analyst Intern  
**Target:** Metasploitable 2 (10.0.2.3)  
**Attacker Machine:** Kali Linux (10.0.2.5)  
**Network:** VirtualBox NAT Network  
**Tools Used:** Nmap 7.98, Wireshark  
**Date:** March 10–11, 2026  
**Classification:** Confidential — Lab Environment Only  

---

## Table of Contents

1. Executive Summary
2. Scope & Methodology
3. Lab Environment
4. Phase 1 — Network Reconnaissance (Nmap)
5. Phase 2 — Traffic Analysis (Wireshark)
6. Vulnerability Findings
7. Risk Summary Matrix
8. Recommendations
9. Conclusion

---

## 1. Executive Summary

A comprehensive security assessment was conducted against a Metasploitable 2 target machine running on a VirtualBox NAT network. The assessment combined active network scanning using Nmap (port, service version, OS detection, and vulnerability scripts) with passive traffic analysis using Wireshark.

The target was found to be **critically vulnerable** across multiple attack surfaces. A total of **23 open ports** were identified, with **10 confirmed CVE-rated vulnerabilities** and several additional high-risk misconfigurations discovered. The most severe finding was a remotely exploitable backdoor in vsftpd 2.3.4 (CVE-2011-2523) that grants immediate root access without authentication. Combined with an exposed Metasploitable root shell on port 1524, the target system can be fully compromised within minutes of initial contact.

**Overall Risk Rating: CRITICAL**

---

## 2. Scope & Methodology

### Scope
- Single host: 10.0.2.3 (Metasploitable 2)
- All TCP ports
- Web applications running on port 80 and 8180
- Network traffic analysis on the NAT segment

### Methodology
The assessment followed a structured approach across two phases:

**Phase 1 — Active Scanning (Nmap)**
- Port scan to identify all open services
- Service version detection (`-sV`)
- Operating system fingerprinting (`-O`)
- Automated vulnerability script scan (`--script vuln`)

**Phase 2 — Passive Traffic Analysis (Wireshark)**
- Live packet capture on the network interface
- HTTP traffic filtering and analysis
- Protocol behavior examination
- Credential and session data analysis

### Commands Used
```bash
nmap 10.0.2.3 -oN nmap_results.txt
nmap -sV 10.0.2.3 >> nmap_results.txt
nmap -O 10.0.2.3 >> nmap_results.txt
nmap --script vuln 10.0.2.3 >> nmap_results.txt
```

---

## 3. Lab Environment

| Component         | Details                              |
|------------------|--------------------------------------|
| Target IP         | 10.0.2.3                            |
| Target OS         | Linux 2.6.9 – 2.6.33 (Ubuntu)      |
| Target Hostname   | metasploitable.localdomain           |
| Target MAC        | 08:00:27:12:1C:0B (VirtualBox NIC)  |
| Attacker IP       | 10.0.2.5 (Kali Linux)               |
| Gateway           | 10.0.2.1 / 10.0.2.2                 |
| Network Type      | VirtualBox NAT Network               |
| Scan Duration     | ~346 seconds total (all scans)      |

---

## 4. Phase 1 — Network Reconnaissance (Nmap)

### 4.1 Open Ports & Services

A total of **23 TCP ports** were found open out of 1000 scanned.

| Port     | State | Service          | Version                          | Risk     |
|---------|-------|-----------------|----------------------------------|----------|
| 21/tcp  | Open  | FTP             | vsftpd 2.3.4                    | CRITICAL |
| 22/tcp  | Open  | SSH             | OpenSSH 4.7p1 Debian 8ubuntu1   | HIGH     |
| 23/tcp  | Open  | Telnet          | Linux telnetd                   | CRITICAL |
| 25/tcp  | Open  | SMTP            | Postfix smtpd                   | MEDIUM   |
| 53/tcp  | Open  | DNS             | ISC BIND 9.4.2                  | MEDIUM   |
| 80/tcp  | Open  | HTTP            | Apache 2.2.8 (Ubuntu) DAV/2     | HIGH     |
| 111/tcp | Open  | RPCbind         | RPC #100000                     | MEDIUM   |
| 139/tcp | Open  | NetBIOS-SSN     | Samba smbd 3.X–4.X              | HIGH     |
| 445/tcp | Open  | SMB             | Samba smbd 3.X–4.X              | HIGH     |
| 512/tcp | Open  | rexec           | netkit-rsh rexecd               | CRITICAL |
| 513/tcp | Open  | rlogin          | OpenBSD/Solaris rlogind         | CRITICAL |
| 514/tcp | Open  | rsh             | tcpwrapped                      | CRITICAL |
| 1099/tcp| Open  | Java RMI        | GNU Classpath grmiregistry      | CRITICAL |
| 1524/tcp| Open  | Bindshell       | Metasploitable root shell       | CRITICAL |
| 2049/tcp| Open  | NFS             | RPC #100003                     | HIGH     |
| 2121/tcp| Open  | FTP             | ProFTPD 1.3.1                   | HIGH     |
| 3306/tcp| Open  | MySQL           | MySQL 5.0.51a-3ubuntu5          | HIGH     |
| 5432/tcp| Open  | PostgreSQL      | PostgreSQL 8.3.0–8.3.7          | HIGH     |
| 5900/tcp| Open  | VNC             | VNC protocol 3.3                | CRITICAL |
| 6000/tcp| Open  | X11             | (access denied)                 | HIGH     |
| 6667/tcp| Open  | IRC             | UnrealIRCd (backdoored)         | CRITICAL |
| 8009/tcp| Open  | AJP13           | Apache Jserv v1.3               | HIGH     |
| 8180/tcp| Open  | HTTP (Tomcat)   | Apache Tomcat/Coyote JSP 1.1    | HIGH     |

### 4.2 Operating System Detection

```
OS: Linux 2.6.9 – 2.6.33
CPE: cpe:/o:linux:linux_kernel:2.6
Network Distance: 1 hop
```

The kernel version 2.6.x is severely outdated and end-of-life. Numerous local privilege escalation exploits exist for this kernel version range.

---

## 5. Phase 2 — Traffic Analysis (Wireshark)

### 5.1 Capture Summary

| Metric              | Value                    |
|--------------------|--------------------------|
| Total Packets       | 213                      |
| Capture Duration    | ~213 seconds             |
| Protocols Observed  | HTTP, TCP, ARP, TLS, DHCP|
| Target Connections  | HTTP port 80             |
| External Traffic    | TLS to 151.101.65.91     |

### 5.2 Critical Finding — Plaintext Credentials

**Packet 22** captured a POST request to `/dvwa/login.php` with credentials transmitted in **cleartext over HTTP**. Any attacker on the same network segment running Wireshark can capture login credentials instantly.

```
POST /dvwa/login.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded
→ Username and password visible in plaintext
```

### 5.3 Network Map Discovered via ARP

| IP Address | MAC Address           | Device             |
|-----------|----------------------|--------------------|
| 10.0.2.1  | 52:55:0a:00:02:01    | NAT Router         |
| 10.0.2.2  | 08:00:27:75:68:af    | DHCP Server        |
| 10.0.2.3  | 08:00:27:12:1c:0b    | Metasploitable 2   |
| 10.0.2.5  | 08:00:27:63:b0:05    | Kali Linux         |

### 5.4 HTTP Session Behavior

The Wireshark capture revealed systematic browsing of DVWA vulnerability modules including brute force, command execution, CSRF, file inclusion, SQL injection, file upload, XSS, and security settings — consistent with active vulnerability reconnaissance. A POST to `/dvwa/vulnerabilities/exec/` showed a ~2-second server response delay, indicating successful command injection execution.

---

## 6. Vulnerability Findings

### FINDING-01 — vsftpd 2.3.4 Backdoor
**Severity: CRITICAL | CVE: CVE-2011-2523 | Port: 21**

The FTP service is running vsftpd 2.3.4, a version that was found to contain a malicious backdoor introduced into the source code in 2011. When a username containing `:)` is sent, the backdoor opens a root shell on port 6200.

Nmap vuln script confirmed exploitation:
```
Shell command: id
Results: uid=0(root) gid=0(root)
```

**Impact:** Complete remote root access with no authentication required.  
**Fix:** Immediately remove vsftpd 2.3.4 and replace with a patched version (vsftpd 3.0.5+). Verify package integrity via checksums.

---

### FINDING-02 — Exposed Root Shell (Bindshell)
**Severity: CRITICAL | Port: 1524**

Port 1524 is running a service identified as `bindshell — Metasploitable root shell`. Connecting to this port via netcat provides an immediate root shell with no credentials required.

```bash
nc 10.0.2.3 1524
# Returns root shell directly
```

**Impact:** Any host on the network can obtain root access in one command.  
**Fix:** Disable and block this port at firewall level immediately. Investigate how this backdoor was installed.

---

### FINDING-03 — UnrealIRCd Backdoor
**Severity: CRITICAL | Port: 6667**

Nmap identified the IRC service as a trojaned version of UnrealIRCd. This backdoor allows unauthenticated remote code execution by sending a specially crafted string to the IRC port.

Reference: http://seclists.org/fulldisclosure/2010/Jun/277

**Impact:** Remote code execution as the IRC daemon user, potentially leading to full system compromise.  
**Fix:** Remove UnrealIRCd immediately and replace with a verified clean version.

---

### FINDING-04 — Telnet Service Open
**Severity: CRITICAL | Port: 23**

Telnet transmits all data — including usernames, passwords, and commands — in plaintext. Any attacker with network access can capture full session content using Wireshark.

**Impact:** Full credential disclosure and session hijacking.  
**Fix:** Disable Telnet completely. Use SSH exclusively for remote administration.

---

### FINDING-05 — R-Services Exposed (rexec/rlogin/rsh)
**Severity: CRITICAL | Ports: 512, 513, 514**

The legacy Berkeley r-services (rexec, rlogin, rsh) are active. These services were designed without authentication for trusted networks and are trivially exploitable. They rely on IP-based trust which can be bypassed via IP spoofing.

**Impact:** Unauthenticated remote command execution.  
**Fix:** Disable all r-services immediately. These are obsolete and should never be present on any system.

---

### FINDING-06 — SSL POODLE Vulnerability
**Severity: HIGH | CVE: CVE-2014-3566 | Ports: 25, 5432**

Both SMTP and PostgreSQL services support SSL 3.0, which is vulnerable to the POODLE (Padding Oracle On Downgraded Legacy Encryption) attack. An attacker performing a MITM attack can decrypt SSL 3.0 traffic by exploiting nondeterministic CBC padding.

**Impact:** Decryption of encrypted communications via man-in-the-middle attack.  
**Fix:** Disable SSL 3.0 on all services. Enforce TLS 1.2 minimum.

---

### FINDING-07 — OpenSSL CCS Injection (MITM)
**Severity: HIGH | CVE: CVE-2014-0224 | Port: 5432**

PostgreSQL's OpenSSL implementation is vulnerable to the CCS Injection attack. This allows a MITM attacker to trigger use of a zero-length master key, effectively breaking TLS session security.

**Impact:** Session hijacking and sensitive data exposure.  
**Fix:** Upgrade OpenSSL to 0.9.8za / 1.0.0m / 1.0.1h or later.

---

### FINDING-08 — Logjam TLS Downgrade (DHE_EXPORT)
**Severity: HIGH | CVE: CVE-2015-4000 | Port: 25**

The SMTP service supports DHE_EXPORT cipher suites, allowing a MITM attacker to downgrade TLS session strength to 512-bit export-grade cryptography, making it trivially breakable.

**Impact:** MITM attacker can decrypt and modify TLS sessions.  
**Fix:** Disable DHE_EXPORT cipher suites. Use DH groups of at least 2048 bits.

---

### FINDING-09 — Java RMI Default Configuration RCE
**Severity: CRITICAL | Port: 1099**

The Java RMI registry is running with its default configuration, which allows loading of classes from remote URLs. This leads to remote code execution by supplying a malicious class URL to the registry.

**Impact:** Remote code execution as the Java process user.  
**Fix:** Disable remote class loading in RMI. Restrict RMI port via firewall.

---

### FINDING-10 — MySQL Exposed on Network Interface
**Severity: HIGH | Port: 3306**

MySQL 5.0.51a is bound to all network interfaces and accessible from remote hosts. This version is end-of-life and contains multiple known vulnerabilities. The database was successfully queried via SQL injection in Task 9, yielding full credential dumps.

**Impact:** Database credential exposure, data theft, potential OS command execution via `INTO OUTFILE`.  
**Fix:** Bind MySQL to 127.0.0.1 only. Upgrade to MySQL 8.x. Implement strong authentication.

---

### FINDING-11 — SQL Injection in Web Applications
**Severity: CRITICAL | Port: 80**

Nmap's HTTP SQL injection script identified extensive SQLi vulnerabilities across the Mutillidae application. Manual exploitation on DVWA (Task 9) confirmed full credential extraction including MD5 password hashes for all 5 users.

**Impact:** Full database compromise, authentication bypass, potential remote code execution.  
**Fix:** Use prepared statements. Sanitize all user input. Deploy WAF.

---

### FINDING-12 — CSRF Vulnerabilities
**Severity: HIGH | Port: 80**

Nmap identified Cross-Site Request Forgery vulnerabilities on DVWA login and multiple Mutillidae forms. Forms lack anti-CSRF tokens.

**Impact:** Attackers can trick authenticated users into performing unintended actions.  
**Fix:** Implement CSRF tokens (synchronizer token pattern) on all state-changing forms.

---

### FINDING-13 — Slowloris DoS Vulnerability
**Severity: HIGH | CVE: CVE-2007-6750 | Port: 8180**

Apache Tomcat on port 8180 is likely vulnerable to Slowloris — a denial of service attack that keeps many connections open using partial HTTP requests, exhausting server connection limits.

**Impact:** Complete web service unavailability.  
**Fix:** Implement connection timeout limits. Use a reverse proxy (nginx) in front of Tomcat.

---

### FINDING-14 — VNC Exposed (No Authentication)
**Severity: CRITICAL | Port: 5900**

VNC is running on protocol version 3.3, which supports no-authentication mode. An attacker can connect and gain full graphical desktop access without credentials.

**Impact:** Full graphical remote access to the system.  
**Fix:** Disable VNC or require strong password authentication. Bind to localhost only and use SSH tunneling.

---

### FINDING-15 — Plaintext Credentials in HTTP Traffic
**Severity: CRITICAL | Port: 80 (Wireshark finding)**

Wireshark packet capture confirmed that DVWA login credentials are transmitted in cleartext over HTTP (Packet 22). No HTTPS is enforced on the web application.

**Impact:** Any network-level observer can capture plaintext credentials.  
**Fix:** Enforce HTTPS with a valid TLS certificate. Implement HTTP Strict Transport Security (HSTS).

---

### FINDING-16 — ARP Spoofing Exposure
**Severity: MEDIUM | Network-level**

ARP has no authentication mechanism. The full MAC-to-IP table was extracted from Wireshark capture. An attacker on the same segment can poison ARP caches to perform MITM attacks against any host pair.

**Impact:** MITM interception of all network traffic on the segment.  
**Fix:** Implement Dynamic ARP Inspection (DAI) on network switches. Use static ARP entries where feasible.

---

### FINDING-17 — NFS Exposed
**Severity: HIGH | Port: 2049**

Network File System (NFS) is running and accessible remotely. Depending on configuration, NFS shares may be mountable without authentication.

**Impact:** Unauthorized access to file system content.  
**Fix:** Restrict NFS exports using `/etc/exports` with specific IP allowlists. Disable if not needed.

---

### FINDING-18 — phpMyAdmin & phpinfo.php Exposed
**Severity: HIGH | Port: 80**

Nmap enumerated `/phpMyAdmin/` and `/phpinfo.php` as publicly accessible. phpinfo.php reveals full server configuration, PHP version, loaded modules, file paths, and environment variables.

**Impact:** Significant information disclosure enabling targeted attacks.  
**Fix:** Remove phpinfo.php from production. Restrict phpMyAdmin access by IP or remove entirely.

---

## 7. Risk Summary Matrix

| # | Finding | Port | CVE | Severity | CVSS |
|---|---------|------|-----|----------|------|
| 1 | vsftpd 2.3.4 backdoor | 21 | CVE-2011-2523 | CRITICAL | 10.0 |
| 2 | Root bindshell exposed | 1524 | — | CRITICAL | 10.0 |
| 3 | UnrealIRCd backdoor | 6667 | — | CRITICAL | 10.0 |
| 4 | Telnet open | 23 | — | CRITICAL | 9.8 |
| 5 | R-services exposed | 512–514 | — | CRITICAL | 9.8 |
| 6 | Java RMI default config RCE | 1099 | — | CRITICAL | 9.3 |
| 7 | VNC no auth | 5900 | — | CRITICAL | 9.0 |
| 8 | SQL Injection (DVWA/Mutillidae) | 80 | — | CRITICAL | 9.0 |
| 9 | Plaintext HTTP credentials | 80 | — | CRITICAL | 8.8 |
| 10 | OpenSSL CCS Injection | 5432 | CVE-2014-0224 | HIGH | 7.4 |
| 11 | SSL POODLE | 25, 5432 | CVE-2014-3566 | HIGH | 7.4 |
| 12 | Logjam / DHE_EXPORT | 25 | CVE-2015-4000 | HIGH | 7.4 |
| 13 | MySQL exposed remotely | 3306 | — | HIGH | 7.3 |
| 14 | CSRF vulnerabilities | 80 | — | HIGH | 7.1 |
| 15 | Slowloris DoS | 8180 | CVE-2007-6750 | HIGH | 7.0 |
| 16 | NFS exposed | 2049 | — | HIGH | 7.0 |
| 17 | phpMyAdmin / phpinfo exposed | 80 | — | HIGH | 6.5 |
| 18 | ARP spoofing exposure | — | — | MEDIUM | 5.9 |

**Total CRITICAL: 9 | HIGH: 8 | MEDIUM: 1**

---

## 8. Recommendations

### Immediate Actions (Within 24 Hours)

1. **Take the system offline** — The severity and number of vulnerabilities makes this machine dangerous on any network.
2. **Remove backdoored services** — vsftpd 2.3.4, UnrealIRCd, and the bindshell on port 1524 must be removed immediately.
3. **Disable Telnet and R-services** — Replace all with SSH-only access.
4. **Block MySQL, PostgreSQL, RMI from external interfaces** — Bind to localhost or firewall these ports.

### Short-Term Fixes (Within 1 Week)

5. **Upgrade all outdated software** — Apache, PHP, MySQL, OpenSSL, SSH, BIND are all severely out of date.
6. **Implement HTTPS** — All web applications must enforce TLS 1.2+ with HSTS.
7. **Deploy a firewall** — Use UFW or iptables to implement a default-deny policy and whitelist only required ports.
8. **Fix web application vulnerabilities** — Implement prepared statements, CSRF tokens, input validation, and security headers.

### Long-Term Security Improvements

9. **Kernel upgrade** — Linux 2.6.x is end-of-life. Upgrade to a supported kernel.
10. **Implement IDS/IPS** — Deploy Snort or Suricata to detect exploitation attempts.
11. **Network segmentation** — Place web servers, databases, and admin systems on separate VLANs.
12. **Implement centralized logging** — Collect and analyze logs from all services.
13. **Regular vulnerability scanning** — Schedule monthly Nmap and Nikto scans.
14. **Replace MD5 password hashing** — Use bcrypt or Argon2 for all password storage.
15. **Implement ARP inspection** — Enable Dynamic ARP Inspection on managed switches.

---

## 9. Conclusion

This network security assessment of Metasploitable 2 using Nmap and Wireshark revealed an extremely vulnerable target with **18 documented security issues**, including 9 at critical severity. The system contains multiple publicly known backdoors (vsftpd, UnrealIRCd, bindshell), critically outdated software stacks, no encryption on administrative protocols, and web applications vulnerable to SQL injection, CSRF, and command execution.

The combination of Nmap scanning and Wireshark traffic analysis provided a comprehensive view of both the attack surface (open ports, services, CVEs) and active exploitation potential (cleartext credentials, visible injection payloads, ARP table disclosure). In a real-world engagement, an attacker would achieve full system root access within minutes using any one of at least 6 confirmed critical vulnerabilities.

This assessment demonstrates the critical importance of patch management, minimal attack surface, defense-in-depth, and traffic encryption as foundational security practices.

---

## Repository Structure

```
Task10_Security_Assessment/
├── nmap_results.txt              # Raw Nmap output (all 4 scans)
├── wireshark_capture.pcap        # Wireshark packet capture (from Task 8)
├── network_security_assessment.md  # This report
├── screenshots/                  # Supporting screenshots
└── README.md                     # Task summary
```

---

## Disclaimer

This security assessment was performed in a controlled lab environment on an intentionally vulnerable machine (Metasploitable 2) connected via VirtualBox NAT Network, solely for educational purposes as part of the Oasis Infobyte Security Analyst Internship. All findings are documented for learning purposes only. Performing security assessments on systems without explicit written authorization is illegal.
