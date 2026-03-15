# Task 8: Capture Network Traffic with Wireshark

## Internship: Oasis Infobyte — Security Analyst Internship
**Level:** Intermediate  
**Tool Used:** Wireshark  


---

## Objective

Capture and analyze network traffic using Wireshark between an attacker machine (Kali Linux) and a vulnerable target (Metasploitable 2), focusing on HTTP traffic, TCP handshakes, ARP activity, and protocol behavior.

---

## Lab Environment

| Component        | Details                        |
|-----------------|-------------------------------|
| Attacker Machine | Kali Linux (10.0.2.5)         |
| Target Machine   | Metasploitable 2 (10.0.2.3)   |
| Gateway          | 10.0.2.1 / 10.0.2.2           |
| External IP      | 151.101.65.91 (TLS traffic)   |
| Network Type     | NAT Network (VirtualBox)      |
| Capture Filter   | HTTP (port 80), ARP, TLS      |
| Total Packets    | 213 captured                  |

---

## Command / Steps Used

1. Opened Wireshark on Kali Linux
2. Selected the active network interface
3. Started live capture
4. Browsed DVWA application on Metasploitable 2 (`http://10.0.2.3/dvwa/`)
5. Applied display filter: `http` to isolate HTTP traffic
6. Saved capture as `wireshark_capture.pcap`

---

## Traffic Overview

The capture contains **213 packets** spanning multiple protocols across approximately **213 seconds** of session time.

| Protocol   | Purpose                                      |
|-----------|----------------------------------------------|
| HTTP       | Web browsing of DVWA application             |
| TCP        | Transport-layer connections (port 80, 443)   |
| TLS v1.2   | Encrypted traffic to external IP             |
| ARP        | Address resolution on the NAT network        |
| DHCP       | IP lease renewal by Kali machine             |

---

## Section 1: TCP Three-Way Handshake Analysis

### Connection 1 — Initial HTTP Session (Packets 7–11)

```
Packet 7:  10.0.2.5 → 10.0.2.3   TCP SYN      Port 39900 → 80
Packet 10: 10.0.2.3 → 10.0.2.5   TCP SYN/ACK  Port 80 → 39900
Packet 11: 10.0.2.5 → 10.0.2.3   TCP ACK      (Handshake complete)
```

**Analysis:** Classic TCP 3-way handshake. Kali initiates a connection to Metasploitable2's Apache web server on port 80. The handshake completes successfully in ~0.3ms, indicating both machines are on the same local NAT network with minimal latency.

### Connection 2 — Second HTTP Session (Packets 51–53)

```
Packet 51: 10.0.2.5 → 10.0.2.3   TCP SYN      Port 45434 → 80
Packet 52: 10.0.2.3 → 10.0.2.5   TCP SYN/ACK
Packet 53: 10.0.2.5 → 10.0.2.3   TCP ACK      (Handshake complete)
```

**Analysis:** A fresh TCP connection is established (new source port 45434), used for continued DVWA browsing. This session carries the majority of the HTTP traffic captured.

---

## Section 2: HTTP Traffic Analysis

### 2.1 GET Requests Observed

| Packet | URL Requested                          | Response Code |
|--------|----------------------------------------|--------------|
| 12     | GET /                                  | 200 OK       |
| 16     | GET /dvwa/                             | 302 Found    |
| 19     | GET /dvwa/login.php                    | 200 OK       |
| 54     | GET /dvwa/                             | 200 OK       |
| 66     | GET /dvwa/login.php                    | 200 OK       |
| 71     | GET /dvwa/instructions.php             | 200 OK       |
| 78     | GET /dvwa/setup.php                    | 200 OK       |
| 83     | GET /dvwa/vulnerabilities/brute/       | 200 OK       |
| 90     | GET /dvwa/vulnerabilities/exec/        | 200 OK       |
| 108    | GET /dvwa/vulnerabilities/csrf/        | 200 OK       |
| 116    | GET /dvwa/vulnerabilities/fi/?page=... | 200 OK       |
| 121    | GET /dvwa/vulnerabilities/sqli/        | 200 OK       |
| 126    | GET /dvwa/vulnerabilities/sqli_blind/  | 200 OK       |
| 133    | GET /dvwa/vulnerabilities/upload/      | 200 OK       |
| 138    | GET /dvwa/vulnerabilities/xss_r/       | 200 OK       |
| 145    | GET /dvwa/vulnerabilities/xss_s/       | 200 OK       |
| 152    | GET /dvwa/security.php                 | 200 OK       |
| 157    | GET /dvwa/phpinfo.php                  | 200 OK       |
| 198    | GET /dvwa/about.php                    | 200 OK       |

**Observation:** The attacker systematically browsed all major DVWA vulnerability modules. This pattern is consistent with reconnaissance before targeted exploitation.

---

### 2.2 POST Requests (Login & Command Execution)

#### POST 1 — Login (Packet 22)
```
Packet 22: POST /dvwa/login.php HTTP/1.1
Content-Type: application/x-www-form-urlencoded
→ Response: 302 Found (redirect to index.php — login successful)
```

**Analysis:** The login credentials were submitted via HTTP POST in **plaintext** (unencrypted). This is a critical security issue — anyone on the network capturing traffic can read the username and password directly. No HTTPS is in use.

#### POST 2 — Command Injection (Packet 97)
```
Packet 97:  POST /dvwa/vulnerabilities/exec/ HTTP/1.1
Content-Type: application/x-www-form-urlencoded
→ Response: 200 OK (delayed — 2 second response time)
```

**Analysis:** A POST request was made to the Command Execution vulnerability module. The server took approximately **2 seconds** to respond (packets 97–106), suggesting a command was executed server-side. This is consistent with OS command injection being performed.

---

### 2.3 HTTP Redirect (302 Found)

```
Packet 17: GET /dvwa/  → 302 Found → Redirect to /dvwa/login.php
Packet 23: POST /dvwa/login.php → 302 Found → Redirect to /dvwa/index.php
```

**Analysis:** Standard session management redirects. After successful login, the server issues a 302 redirect to the authenticated dashboard.

---

## Section 3: ARP Traffic Analysis

### ARP Packets Observed

| Packet | Request                              | Response                            |
|--------|--------------------------------------|-------------------------------------|
| 5–6    | Who has 10.0.2.1? (from Kali)       | 10.0.2.1 is at 52:55:0a:00:02:01   |
| 8–9    | Who has 10.0.2.5? (from Metaspl.)   | 10.0.2.5 is at 08:00:27:63:b0:05   |
| 43–44  | Who has 10.0.2.1? (from Kali)       | 10.0.2.1 is at 52:55:0a:00:02:01   |
| 46–47  | Who has 10.0.2.5? (from gateway)    | 10.0.2.5 is at 08:00:27:63:b0:05   |
| 49–50  | Who has 10.0.2.2? (from Kali)       | 10.0.2.2 is at 08:00:27:75:68:af   |
| 62–63  | Who has 10.0.2.3? (from Kali)       | 10.0.2.3 is at 08:00:27:12:1c:0b   |
| 99     | Who has 10.0.2.1? (from Metaspl.)   | (No response captured)              |

**MAC Address Table Discovered:**

| IP Address | MAC Address           | Device          |
|-----------|----------------------|-----------------|
| 10.0.2.2  | 08:00:27:75:68:af    | DHCP Gateway    |
| 10.0.2.3  | 08:00:27:12:1c:0b    | Metasploitable 2|
| 10.0.2.5  | 08:00:27:63:b0:05    | Kali Linux      |
| 10.0.2.1  | 52:55:0a:00:02:01    | NAT Router      |

**Security Implication:** ARP has no authentication mechanism. In a real network, an attacker could poison the ARP cache to perform Man-in-the-Middle (MITM) attacks by associating their MAC address with a legitimate IP address.

---

## Section 4: TLS Traffic Analysis

### Encrypted External Traffic (Packets 1–4, 34–42)

```
Packets 1–4:   10.0.2.5 ↔ 151.101.65.91   TLSv1.2 (port 443)
Packets 34–42: 10.0.2.5 ↔ 151.101.65.91   TLSv1.2 (connection teardown)
```

**Analysis:** Kali was simultaneously communicating with an external IP (151.101.65.91) over TLS. The IP resolves to Fastly CDN infrastructure, likely from background browser or system updates. The session properly closes with FIN/ACK and RST packets.

---

## Section 5: DHCP Analysis

### DHCP IP Lease Renewal (Packets 45–48)

```
Packet 45: 10.0.2.5 → 10.0.2.2   DHCP Request   Transaction ID: 0x9a20c799
Packet 48: 10.0.2.2 → 10.0.2.5   DHCP ACK       IP confirmed: 10.0.2.5
```

**Analysis:** Kali's DHCP lease was renewed mid-session. The DHCP server (10.0.2.2) acknowledged the request and confirmed the existing IP assignment. DHCP traffic is unencrypted and can be exploited via DHCP starvation or rogue DHCP server attacks.

---

## Section 6: TCP Session Lifecycle

### Keep-Alive Mechanism
```
Packet 29:  TCP Keep-Alive        (Kali → Metaspl.)
Packet 30:  TCP Keep-Alive ACK    (Metaspl. → Kali)
```

Both sides maintained the connection using TCP Keep-Alive probes during idle periods, preventing the session from timing out.

### Connection Teardown (Packets 31–33)
```
Packet 31: FIN/ACK   Metasploitable → Kali   (Server initiates close)
Packet 32: FIN/ACK   Kali → Metasploitable   (Client confirms)
Packet 33: ACK       Metasploitable → Kali   (Teardown complete)
```

**Analysis:** Clean 4-way TCP teardown. The server (Metasploitable) initiated the connection close after the idle timeout, and Kali responded correctly. This is normal behavior for HTTP/1.1 keep-alive sessions.

---

## Section 7: Security Findings from Traffic Analysis

| # | Finding | Risk Level | Detail |
|---|---------|-----------|--------|
| 1 | HTTP login credentials sent in plaintext | CRITICAL | POST to /dvwa/login.php over HTTP — password fully visible in packet |
| 2 | No HTTPS/TLS on DVWA | HIGH | All web application traffic is unencrypted on port 80 |
| 3 | Command injection visible in POST | HIGH | Exec module POST request exposes server-side command execution |
| 4 | ARP has no authentication | MEDIUM | MAC-to-IP mappings can be spoofed for MITM attacks |
| 5 | DHCP unencrypted | MEDIUM | DHCP starvation/rogue server attacks possible |
| 6 | PHP Easter Egg requests in traffic | LOW | PHP version fingerprinting via special query strings |
| 7 | Full directory structure enumerable | LOW | Attacker browsed all vulnerability modules undetected |

---

## Wireshark Filter Commands Used

```wireshark
# Filter only HTTP traffic
http

# Filter traffic between specific hosts
ip.addr == 10.0.2.3

# Filter only GET requests
http.request.method == "GET"

# Filter only POST requests
http.request.method == "POST"

# Filter ARP traffic
arp

# Filter TCP handshakes only
tcp.flags.syn == 1
```

---

## Conclusion

The Wireshark capture successfully recorded **213 packets** covering an entire DVWA browsing session. Key findings include login credentials transmitted in plaintext over HTTP, command injection activity visible in POST requests, full ARP table disclosure of the NAT network, and systematic browsing of all DVWA vulnerability modules. This analysis demonstrates how network traffic capture is a powerful tool for both attackers and defenders — defenders can use it to detect unauthorized access patterns, while attackers use it to intercept sensitive data on unencrypted networks.

---

## Repository Structure

```
Task8_Wireshark/
├── wireshark_capture.pcap   # Raw packet capture file
├── screenshots/             # Wireshark UI screenshots
└── README.md                # This analysis report
```

---

## Disclaimer

This traffic capture was performed in a controlled lab environment on an intentionally vulnerable machine (Metasploitable 2) connected via a VirtualBox NAT Network. This activity was conducted solely for educational purposes as part of the Oasis Infobyte Security Analyst Internship.
