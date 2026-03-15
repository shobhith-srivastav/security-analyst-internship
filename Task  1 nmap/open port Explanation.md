Nmap Scan Port Explanation

Scan performed using Nmap.

The scan detected multiple open ports running different services on the target machine.


---

1️⃣ Port 21 — FTP

Service: FTP (vsftpd 2.3.4)

Purpose

Used for file transfer between computers.


Security Risk

The version vsftpd 2.3.4 contains a known backdoor vulnerability.

Attackers can gain remote shell access.



---

2️⃣ Port 22 — SSH

Service: OpenSSH

Purpose

Secure remote login to a system.


Security Risk

Weak passwords can allow brute-force login attacks.



---

3️⃣ Port 23 — Telnet

Service: Telnet

Purpose

Remote command line access.


Security Risk

Sends credentials in plain text.

Easily intercepted by attackers.



---

4️⃣ Port 25 — SMTP

Service: Postfix SMTP

Purpose

Used for sending emails.


Security Risk

Can be abused for spam relay attacks if misconfigured.



---

5️⃣ Port 53 — DNS

Service: BIND DNS Server

Purpose

Resolves domain names into IP addresses.


Example
google.com → IP address

Security Risk

DNS misconfiguration can lead to DNS spoofing or cache poisoning.



---

6️⃣ Port 80 — HTTP

Service: Apache Web Server

Purpose

Hosts websites.


Security Risk

Web applications may contain vulnerabilities like

SQL Injection

XSS

File Upload vulnerabilities.




---

7️⃣ Port 111 — RPCBind

Service: RPC service

Purpose

Helps other services locate RPC services.


Security Risk

Can expose internal services to attackers.



---

8️⃣ Port 139 — NetBIOS

Service: Samba

Purpose

Windows file and printer sharing.


Security Risk

Allows enumeration of:

users

shares

system information.




---

9️⃣ Port 445 — SMB

Service: Samba SMB

Purpose

Network file sharing.


Security Risk

Historically exploited by worms like:

EternalBlue

WannaCry ransomware.




---

🔟 Port 512 — rexec

Service: Remote execution service

Purpose

Execute commands remotely.


Security Risk

Very insecure authentication system.



---

1️⃣1️⃣ Port 513 — rlogin

Service: Remote login

Purpose

Remote access similar to SSH.


Security Risk

Transmits credentials unencrypted.



---

1️⃣2️⃣ Port 514 — rsh

Service: Remote shell

Purpose

Execute commands remotely.


Security Risk

Trusted host authentication can be abused.



---

1️⃣3️⃣ Port 1099 — Java RMI

Service: Java Remote Method Invocation

Purpose

Allows Java programs to communicate remotely.


Security Risk

Can be exploited for remote code execution.



---

1️⃣4️⃣ Port 1524 — Bind Shell

Service: Metasploitable Root Shell

Purpose

Backdoor shell.


Security Risk

Anyone connecting to this port may get root access.



---

1️⃣5️⃣ Port 2049 — NFS

Service: Network File System

Purpose

Share files over a network.


Security Risk

Misconfigured NFS allows unauthorized file access.



---

1️⃣6️⃣ Port 2121 — FTP

Service: ProFTPD

Purpose

Alternative FTP service.


Security Risk

Some versions vulnerable to command execution.



---

1️⃣7️⃣ Port 3306 — MySQL

Service: MySQL Database

Purpose

Stores application data.


Security Risk

Weak credentials may expose the database.



---

1️⃣8️⃣ Port 5432 — PostgreSQL

Service: PostgreSQL Database

Purpose

Another relational database.


Security Risk

Exposed database may leak sensitive data.



---

1️⃣9️⃣ Port 5900 — VNC

Service: Virtual Network Computing

Purpose

Remote desktop access.


Security Risk

Weak passwords allow full remote control.



---

2️⃣0️⃣ Port 6000 — X11

Service: Linux graphical display server

Purpose

Remote graphical interface.


Security Risk

Attackers may capture keyboard and screen data.



---

2️⃣1️⃣ Port 6667 — IRC

Service: Internet Relay Chat

Purpose

Chat communication server.


Security Risk

Often used for botnet command & control.



---

2️⃣2️⃣ Port 8009 — AJP13

Service: Apache JServ Protocol

Purpose

Connects Apache with Tomcat.


Security Risk

Vulnerable to Ghostcat attack.



---

2️⃣3️⃣ Port 8180 — HTTP (Tomcat)

Service: Apache Tomcat

Purpose

Hosts Java web applications.


Security Risk

Default credentials may allow admin access.



---

📊 Conclusion 

The scan shows many open ports running outdated services, indicating the system is intentionally vulnerable. Such exposure allows attackers to perform:

Remote login attacks

Database exploitation

Web application attacks

Remote shell access


This environment is commonly used to practice penetration testing techniques.


 