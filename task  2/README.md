# Task 2: Basic Firewall Configuration with UFW

## Internship: Oasis Infobyte — Security Analyst Internship
**Level:** Beginner  
**Tool Used:** UFW (Uncomplicated Firewall) v0.36.2   

---

## Objective

Set up a basic firewall on a Linux system using UFW to allow SSH traffic and deny HTTP traffic, then verify the active firewall rules are correctly in place.

---

## Lab Environment

| Component        | Details                        |
|-----------------|-------------------------------|
| Machine          | Kali Linux                    |
| UFW Version      | 0.36.2 (Canonical Ltd.)       |
| Interface        | All network interfaces        |

---

## Commands Used

```bash
# Step 1: Install UFW
sudo apt install ufw

# Step 2: Check UFW version
ufw version

# Step 3: Enable the firewall
sudo ufw enable

# Step 4: Allow SSH (port 22)
sudo ufw allow ssh

# Step 5: Deny HTTP (port 80)
sudo ufw deny http

# Step 6: Verify active rules
sudo ufw status
```

---

## Step-by-Step Execution

### Step 1: Install UFW
```
sudo apt install ufw
ufw is already the newest version (0.36.2-9).
```
UFW was already present on the system at version 0.36.2. No installation required.

---

### Step 2: Verify UFW Version
```
ufw version
ufw 0.36.2
Copyright 2008-2023 Canonical Ltd.
```

---

### Step 3: Enable the Firewall
```
sudo ufw enable
Firewall is active and enabled on system startup
```
UFW is now active and will automatically start on every system boot.

---

### Step 4: Allow SSH (Port 22)
```
sudo ufw allow ssh
Rule added
Rule added (v6)
```
Two rules were added — one for IPv4 and one for IPv6. SSH traffic on port 22/tcp is now permitted from any source.

---

### Step 5: Deny HTTP (Port 80)
```
sudo ufw deny http
Rule added
Rule added (v6)
```
Two deny rules were added for HTTP traffic on port 80/tcp — blocking both IPv4 and IPv6 HTTP connections.

---

### Step 6: Verify Firewall Status
```
sudo ufw status
Status: active

To          Action      From
--          ------      ----
22/tcp      ALLOW       Anywhere
80/tcp      DENY        Anywhere
22/tcp (v6) ALLOW       Anywhere (v6)
80/tcp (v6) DENY        Anywhere (v6)
```
All 4 rules are confirmed active and correctly configured.

---

## Firewall Rules Explanation

| Rule | Port | Protocol | Action | Reason |
|------|------|----------|--------|--------|
| SSH Allow (IPv4) | 22 | TCP | ALLOW | Permits secure remote administration |
| HTTP Deny (IPv4) | 80 | TCP | DENY  | Blocks unencrypted web traffic |
| SSH Allow (IPv6) | 22 | TCP | ALLOW | Same as above for IPv6 networks |
| HTTP Deny (IPv6) | 80 | TCP | DENY  | Same as above for IPv6 networks |

---

## Why Allow SSH and Deny HTTP?

### Why Allow SSH (Port 22)?
SSH (Secure Shell) is the standard encrypted protocol for remote system administration. It uses strong encryption (AES, ChaCha20) and public-key authentication to protect all communication. Allowing SSH ensures administrators can securely access and manage the system remotely without exposing credentials over the network.

### Why Deny HTTP (Port 80)?
HTTP transmits all data — including usernames, passwords, form data, and session cookies — in plaintext. Any attacker on the same network can intercept this traffic using tools like Wireshark (demonstrated in Task 8). Denying HTTP forces users toward HTTPS (port 443) which encrypts all traffic using TLS. Blocking unnecessary open ports also reduces the overall attack surface.

---

## UFW Default Policy

By default, UFW operates with:
- **Incoming traffic:** DENY all (unless explicitly allowed)
- **Outgoing traffic:** ALLOW all

This default-deny posture means any port not explicitly permitted is automatically blocked. Our rules build on top of this by explicitly allowing SSH and explicitly denying HTTP for clarity and auditability.

---

## Security Significance

Firewalls are the first line of defense in network security. Without a firewall, every open port on a system is directly reachable by any host on the network. As demonstrated in Task 1, Metasploitable 2 exposed 23 open ports with zero firewall protection — every one of those services was directly attackable from the network. Proper UFW configuration enforcing minimum necessary access would have blocked the majority of those attack vectors entirely.

**Key principle:** Default-deny — only allow what is explicitly required, block everything else.

---

## Repository Structure

```
Task2_UFW_Firewall/
├── ufw_configuration.sh         # Shell script with all UFW commands
├── screenshots/
│   └── ufw_config_results.jpeg  # Screenshot of ufw status output
└── README.md                    # This report
```

---

## Disclaimer

This firewall configuration was performed on a Kali Linux machine in a controlled lab environment for educational purposes as part of the Oasis Infobyte Security Analyst Internship.
