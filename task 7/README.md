# Task 7: Vulnerability Scanning with Nikto

## Internship: Oasis Infobyte — Security Analyst Internship
**Level:** Intermediate  
**Tool Used:** Nikto v2.6.0  

---

## Objective

Use Nikto to perform a vulnerability scan on a web server and analyze the results to identify potential security issues.

---

## Lab Environment

| Component        | Details                        |
|-----------------|-------------------------------|
| Attacker Machine | Kali Linux                    |
| Target Machine   | Metasploitable 2              |
| Target IP        | 10.0.2.3                      |
| Target URL       | http://10.0.2.3/dvwa/         |
| Network Type     | NAT Network (VirtualBox)      |
| Web Server       | Apache/2.2.8 (Ubuntu) DAV/2   |
| PHP Version      | PHP/5.2.4-2ubuntu5.10         |

---

## Command Used

```bash
nikto -h http://10.0.2.3/dvwa/ -o scan_result.txt
```

**Flags explained:**
- `-h` — Target host/URL
- `-o` — Output file to save results

---

## Scan Summary

| Metric              | Value         |
|--------------------|--------------|
| Scan Start Time     | 13:08:59 GMT-4 |
| Scan End Time       | 13:14:42 GMT-4 |
| Total Duration      | 343 seconds   |
| Total Requests      | 8238          |
| Errors              | 16            |
| Vulnerabilities Found | 26          |

---

## Findings & Analysis

### 1. Outdated Software Versions (HIGH RISK)

| Software | Detected Version        | Current Version |
|---------|------------------------|----------------|
| Apache  | 2.2.8                  | 2.4.66+        |
| PHP     | 5.2.4-2ubuntu5.10      | 8.5.1+         |

**Risk:** Outdated versions contain known, publicly documented CVEs. Apache 2.2.8 and PHP 5.2.4 are both end-of-life and have numerous unpatched vulnerabilities that can be easily exploited.

**Mitigation:** Upgrade Apache to 2.4.x and PHP to 8.x immediately.

---

### 2. Missing Security Headers (MEDIUM RISK)

The following HTTP security headers are absent from server responses:

| Missing Header               | Purpose                                              |
|-----------------------------|------------------------------------------------------|
| `Strict-Transport-Security` | Forces HTTPS connections, prevents downgrade attacks |
| `X-Content-Type-Options`    | Prevents MIME-type sniffing                          |
| `Content-Security-Policy`   | Prevents XSS and data injection attacks              |
| `Permissions-Policy`        | Controls browser feature access                      |
| `Referrer-Policy`           | Controls referrer information leakage                |

**Mitigation:** Add all of the above headers in Apache's configuration file (`httpd.conf` or `.htaccess`).

---

### 3. Insecure Cookie Configuration (MEDIUM RISK)

Two cookies were identified without the `HttpOnly` flag:

- `PHPSESSID`
- `security`

**Risk:** Without `HttpOnly`, cookies are accessible via JavaScript. This makes the application vulnerable to session hijacking through Cross-Site Scripting (XSS) attacks.

**Mitigation:** Set `HttpOnly` and `Secure` flags on all session cookies.

---

### 4. Directory Indexing Enabled (MEDIUM RISK)

The following directories have open directory indexing:

- `/dvwa/docs/`
- `/dvwa/config/`

**Risk:** An attacker can browse the directory structure and access configuration files, potentially revealing database credentials, API keys, or application logic.

**Mitigation:** Disable directory indexing in Apache config with `Options -Indexes`.

---

### 5. HTTP TRACE Method Enabled — XST Vulnerability (MEDIUM RISK)

**Finding:** The HTTP TRACE method is active on the server.

**Risk:** This enables Cross-Site Tracing (XST) attacks. Combined with XSS, an attacker can steal `HttpOnly` cookies and session tokens by tracing HTTP requests.

**CVE Reference:** Well-documented attack vector — OWASP XST.

**Mitigation:** Disable TRACE method in Apache config:
```apache
TraceEnable Off
```

---

### 6. PHP Easter Eggs Exposed (LOW–MEDIUM RISK)

Four PHP Easter Egg URLs were discovered:

```
/dvwa/?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000
/dvwa/?=PHPE9568F36-D428-11d2-A769-00AA001ACF42
/dvwa/?=PHPE9568F34-D428-11d2-A769-00AA001ACF42
/dvwa/?=PHPE9568F35-D428-11d2-A769-00AA001ACF42
```

**Risk:** These expose server-side PHP version and configuration details, aiding fingerprinting and targeted attacks.

**Mitigation:** Disable PHP Easter Eggs by setting `expose_php = Off` in `php.ini`.

---

### 7. ETag Inode Leakage (LOW RISK)

**Finding:** ETags in `robots.txt` expose inode numbers, file size, and modification timestamps.

**CVE:** CVE-2003-1418

**Risk:** Inode information can assist in brute-force or enumeration attacks.

**Mitigation:** Configure Apache to exclude inode data from ETags:
```apache
FileETag MTime Size
```

---

### 8. Apache MultiViews Enabled (LOW RISK)

**Finding:** `mod_negotiation` with `MultiViews` is enabled, allowing file name brute-forcing.

**Risk:** Attackers can enumerate existing files on the server even without knowing exact file names.

**Mitigation:** Disable `MultiViews` in Apache config:
```apache
Options -MultiViews
```

---

### 9. Configuration File Accessible (HIGH RISK)

**Finding:** `/dvwa/config/` directory is publicly accessible and may expose database configuration.

**Risk:** If `config.inc.php` or similar files are readable, database credentials could be fully exposed.

**Mitigation:** Restrict access to config directories via `.htaccess` or Apache config.

---

### 10. Admin Login Page Exposed (INFORMATIONAL)

**Finding:** `/dvwa/login.php` is publicly accessible.

**Risk:** Exposes the admin login to brute-force and credential stuffing attacks.

**Mitigation:** Restrict access to admin pages by IP whitelist or implement rate limiting.

---

## Vulnerability Summary Table

| # | Finding                        | Severity | CVE/Reference     |
|---|-------------------------------|----------|-------------------|
| 1 | Outdated Apache (2.2.8)        | HIGH     | Multiple CVEs     |
| 2 | Outdated PHP (5.2.4)           | HIGH     | Multiple CVEs     |
| 3 | Config directory accessible    | HIGH     | —                 |
| 4 | Missing security headers (×5)  | MEDIUM   | OWASP Headers     |
| 5 | Cookies without HttpOnly       | MEDIUM   | OWASP A05         |
| 6 | Directory indexing enabled     | MEDIUM   | —                 |
| 7 | HTTP TRACE / XST               | MEDIUM   | OWASP XST         |
| 8 | PHP Easter Eggs exposed        | LOW-MED  | —                 |
| 9 | ETag inode leakage             | LOW      | CVE-2003-1418     |
| 10| Apache MultiViews enabled      | LOW      | —                 |

---

## Conclusion

The Nikto scan against Metasploitable 2's DVWA application revealed **26 vulnerabilities** across critical categories including severely outdated software, missing security headers, insecure cookie handling, and exposed configuration directories. The target system is intentionally vulnerable and serves as an ideal learning environment. In a real production environment, any one of these findings — especially the outdated Apache/PHP versions and exposed config directory — would constitute a critical security incident requiring immediate remediation.

---

## Repository Structure

```
Task7_Nikto_Scan/
├── nikto_scan_results.txt   # Raw Nikto output
├── screenshots/             # Screenshots of scan execution
└── README.md                # This report
```

---

## Disclaimer

This scan was performed in a controlled lab environment on an intentionally vulnerable machine (Metasploitable 2) for educational purposes only. Unauthorized scanning of systems you do not own is illegal.
