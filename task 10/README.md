# Task 10: Network Security Assessment Report

## Internship: Oasis Infobyte — Security Analyst Internship
**Level:** Advanced  
**Tools Used:** Nmap 7.98, Wireshark  
**Target:** Metasploitable 2 (10.0.2.3)  


---

## Objective

Perform a comprehensive network security assessment by scanning a local test network using Nmap, capturing network traffic with Wireshark, and producing a detailed security report identifying all vulnerabilities.

---

## Commands Used

```bash
# 1. Basic port scan
nmap 10.0.2.3 -oN nmap_results.txt

# 2. Service version detection
nmap -sV 10.0.2.3 >> nmap_results.txt

# 3. OS fingerprinting
nmap -O 10.0.2.3 >> nmap_results.txt

# 4. Full vulnerability script scan
nmap --script vuln 10.0.2.3 >> nmap_results.txt
```

---

## Key Findings Summary

| Severity | Count | Examples |
|---------|-------|---------|
| CRITICAL | 9 | vsftpd backdoor, root bindshell, UnrealIRCd backdoor, Telnet, R-services, RMI RCE, VNC, SQLi, plaintext creds |
| HIGH | 8 | SSL POODLE, CCS Injection, Logjam, MySQL exposed, CSRF, Slowloris, NFS, phpMyAdmin |
| MEDIUM | 1 | ARP spoofing exposure |

**Total Open Ports:** 23  
**Total Vulnerabilities Found:** 18  
**Overall Risk Rating:** CRITICAL  

---

## Files

| File | Description |
|------|-------------|
| `nmap_results.txt` | Raw Nmap output from all 4 scans |
| `wireshark_capture.pcap` | Network traffic capture |
| `network_security_assessment.md` | Full detailed assessment report |
| `screenshots/` | Supporting screenshots |

---

## Disclaimer

Performed in a controlled lab environment on Metasploitable 2 for educational purposes only.
