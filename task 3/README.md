 # SQL Injection Demonstration using DVWA

## Objective
To demonstrate an SQL Injection vulnerability in a web application using Damn Vulnerable Web Application (DVWA) with the security level set to low.

## Tool Used
DVWA (Damn Vulnerable Web Application)

## Environment
- Localhost
- DVWA running on Apache
- Security Level: Low

## Steps Performed

1. Installed and configured DVWA on a local server.
2. Logged into DVWA using default credentials.
3. Set the DVWA security level to **Low**.
4. Navigated to the SQL Injection vulnerability page.
5. Entered the following payload in the input field:
1' OR '1'='1
Copy code

6. Submitted the request.

## Result

The application returned multiple user records from the database.  
This confirms that the web application is vulnerable to SQL Injection.

## Explanation

SQL Injection occurs when user input is directly included in a SQL query without proper validation or sanitization.

The payload:
1' OR '1'='1
Copy code

modifies the SQL query so that the condition **'1'='1'** is always true.  
Because of this, the database returns all records instead of a single user.

## Impact

- Unauthorized access to application data
- Exposure of sensitive database information
- Possible authentication bypass

## Mitigation

SQL Injection vulnerabilities can be prevented by:

- Using prepared statements
- Parameterized queries
- Input validation
- Proper escaping of user input

## Files Included

- `sql_injection_exploit.sh` — Script demonstrating the SQL Injection request
- `sql injection images` — Images showing the attack and results