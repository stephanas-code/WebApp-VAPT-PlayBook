# Sensitive Data Exposure - Payloads

## Overview
This document contains quick-test payloads and techniques for detecting sensitive data exposure vulnerabilities during web application security assessments.

---

## 1. HTTP to HTTPS Downgrade Testing

### Basic HTTP Request
```http
GET /login HTTP/1.1
Host: example.com
```
**Purpose**: Check if sensitive endpoints accept unencrypted HTTP connections

### Force HTTP on Sensitive Endpoints
```bash
curl -v http://example.com/api/user/profile
curl -v http://example.com/checkout
curl -v http://example.com/password-reset
```
**Expected**: Should redirect to HTTPS or reject connection

---

## 2. Cookie Security Testing

### Check Cookie Attributes via Browser Console
```javascript
document.cookie.split(';').forEach(cookie => console.log(cookie));
```
**Look for**: Missing Secure, HttpOnly, or SameSite flags

### Intercept Cookies Without Secure Flag
```http
GET /dashboard HTTP/1.1
Host: example.com
Cookie: session=abc123; token=sensitive_data
```
**Risk**: Cookies without Secure flag can be transmitted over HTTP

---

## 3. Response Header Analysis

### Check for Sensitive Data in Headers
```bash
curl -I https://example.com/api/endpoint
```
**Look for**:
- `X-API-Key: secretkey123`
- `X-Database-Host: internal-db.local`
- `Server: Apache/2.4.1 (Ubuntu) PHP/7.2.1`
- `X-Debug-Token: debug_info`

### Verbose Error Headers
```http
GET /api/invalid HTTP/1.1
Host: example.com
```
**Look for**: Stack traces, database errors, internal paths in response headers

---

## 4. URL Parameter Exposure

### Sensitive Data in GET Requests
```
https://example.com/reset?password=newPass123
https://example.com/api?api_key=sk_live_1234567890
https://example.com/profile?ssn=123-45-6789
https://example.com/login?username=admin&password=secret
```
**Risk**: Logged in browser history, proxy logs, referrer headers

### Test Parameter Logging
```bash
curl "https://example.com/api?credit_card=4111111111111111"
```
**Check**: Server logs, analytics, third-party scripts

---

## 5. Client-Side Storage Testing

### Check localStorage for Sensitive Data
```javascript
for (let i = 0; i < localStorage.length; i++) {
    let key = localStorage.key(i);
    console.log(`${key}: ${localStorage.getItem(key)}`);
}
```
**Look for**: Passwords, tokens, PII, credit card data

### Check sessionStorage
```javascript
for (let i = 0; i < sessionStorage.length; i++) {
    let key = sessionStorage.key(i);
    console.log(`${key}: ${sessionStorage.getItem(key)}`);
}
```

### IndexedDB Inspection
```javascript
indexedDB.databases().then(dbs => console.log(dbs));
```

---

## 6. API Response Analysis

### Over-Verbose API Responses
```http
GET /api/user/1 HTTP/1.1
Host: example.com
Authorization: Bearer token123
```
**Look for excessive data**:
```json
{
  "id": 1,
  "username": "john",
  "email": "john@example.com",
  "password_hash": "$2a$10$...",
  "ssn": "123-45-6789",
  "credit_card": "4111-1111-1111-1111",
  "internal_notes": "VIP customer",
  "db_connection": "mysql://user:pass@host"
}
```

### Test Different Content-Type Headers
```bash
curl -H "Accept: application/xml" https://example.com/api/user
curl -H "Accept: text/html" https://example.com/api/user
```
**Purpose**: Some formats may expose more data than others

---

## 7. Error Message Testing

### Force Database Errors
```http
GET /user?id=1' HTTP/1.1
Host: example.com
```
**Look for**: Database names, table structures, query details

### Invalid Input Testing
```bash
curl https://example.com/api/user/-1
curl https://example.com/api/user/999999999
curl https://example.com/api/user/../../../../etc/passwd
```
**Look for**: Stack traces, file paths, internal IPs

### Null/Empty Parameter Testing
```http
POST /api/process HTTP/1.1
Host: example.com
Content-Type: application/json

{"data": null}
```

---

## 8. Source Code Comments

### View Page Source for Comments
```html
<!-- TODO: Remove hardcoded API key before production -->
<!-- Database password: dbPass123! -->
<!-- Admin panel: /secret-admin-portal -->
<!-- API endpoint: https://internal-api.company.local -->
```

### Check JavaScript Files
```bash
curl https://example.com/static/js/main.js | grep -i "password\|secret\|key\|token"
```

### Search for Credentials in JS
```javascript
// Common patterns
const API_KEY = "sk_live_1234567890";
var dbPassword = "admin123";
let secretToken = "secret_token_here";
```

---

## 9. Backup File Discovery

### Common Backup File Extensions
```bash
https://example.com/config.php.bak
https://example.com/database.sql.old
https://example.com/backup.zip
https://example.com/.env.backup
https://example.com/web.config.old
https://example.com/settings.py~
https://example.com/.git/config
https://example.com/.DS_Store
```

### Common Config Files
```bash
https://example.com/.env
https://example.com/config.json
https://example.com/settings.xml
https://example.com/database.yml
https://example.com/secrets.txt
```

---

## 10. Cache Testing

### Check Cache-Control Headers
```bash
curl -I https://example.com/account/profile
```
**Look for missing**:
```http
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
```

### Test Browser Cache
```
1. Login to application
2. Visit sensitive page
3. Logout
4. Press browser back button
```
**Risk**: Sensitive data visible from cache

---

## 11. Autocomplete Testing

### Check Form Autocomplete
```html
<!-- Insecure -->
<input type="password" name="password">
<input type="text" name="credit_card">

<!-- Secure -->
<input type="password" name="password" autocomplete="off">
<input type="text" name="credit_card" autocomplete="off">
```

---

## 12. TLS/SSL Configuration Testing

### Check SSL/TLS Version
```bash
nmap --script ssl-enum-ciphers -p 443 example.com
openssl s_client -connect example.com:443 -tls1
openssl s_client -connect example.com:443 -ssl3
```

### Test Weak Ciphers
```bash
testssl.sh example.com
sslscan example.com
```
**Look for**: SSLv3, TLS 1.0, weak cipher suites

---

## 13. Log File Exposure

### Common Log File Paths
```bash
https://example.com/logs/access.log
https://example.com/logs/error.log
https://example.com/debug.log
https://example.com/server.log
https://example.com/application.log
```

### Check for Verbose Logging
```bash
curl https://example.com/app.log | grep -i "password\|token\|secret"
```

---

## 14. Directory Listing

### Test for Open Directories
```bash
https://example.com/uploads/
https://example.com/backups/
https://example.com/config/
https://example.com/admin/
https://example.com/data/
```

---

## 15. Sensitive Data in URLs (Referer Leakage)

### Test Referer Header Leakage
```html
<!-- On page with sensitive data in URL -->
<a href="https://external-site.com">Click here</a>
```
**Check if**: `Referer: https://example.com/reset?token=secret123` is sent

### Mitigation Check
```html
<meta name="referrer" content="no-referrer">
```

---

## 16. Predictable Resource Location

### Sequential File Access
```bash
https://example.com/documents/report_2024_01.pdf
https://example.com/documents/report_2024_02.pdf
https://example.com/invoices/invoice_001.pdf
https://example.com/invoices/invoice_002.pdf
```

---

## 17. Weak Cryptography Detection

### Identify Weak Hashing in Responses
```json
{
  "password_hash": "5f4dcc3b5aa765d61d8327deb882cf99"
}
```
**Identify**: MD5 (32 hex chars), SHA1 (40 hex chars)

### Base64 Encoded Secrets
```bash
echo "YWRtaW46cGFzc3dvcmQxMjM=" | base64 -d
# Output: admin:password123
```

---

## 18. Session Token Analysis

### Weak Session Token Patterns
```
SESSION=user1_20241012
token=12345678
auth=admin:true
```

### Test Token Entropy
```python
import math
from collections import Counter

def calculate_entropy(token):
    counter = Counter(token)
    length = len(token)
    entropy = -sum((count/length) * math.log2(count/length) 
                   for count in counter.values())
    return entropy * length

# Low entropy = weak token
print(calculate_entropy("123456789012"))  # Low
print(calculate_entropy("a8$K9#mP2@qL5"))  # High
```

---

## 19. Mass Assignment / Over-Posting

### Expose Hidden Fields
```http
POST /api/user/update HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "username": "john",
  "email": "john@example.com",
  "is_admin": true,
  "credit_balance": 10000,
  "internal_notes": "test"
}
```
**Purpose**: See if API exposes internal fields in error messages

---

## 20. Cryptographic Storage Testing

### Check Password Storage
```sql
SELECT username, password FROM users;
```
**Look for**:
- Plaintext passwords
- Reversibly encrypted passwords
- Weak hashes (MD5, SHA1)

### Check Sensitive Data Encryption
```sql
SELECT credit_card, ssn FROM user_data;
```
**Should be**: Encrypted at rest, not plaintext

---

## Testing Checklist

- [ ] All sensitive data transmitted over HTTPS
- [ ] Sensitive cookies have Secure flag
- [ ] No sensitive data in URL parameters
- [ ] No sensitive data in localStorage/sessionStorage
- [ ] API responses don't expose excessive data
- [ ] Error messages don't reveal sensitive info
- [ ] No credentials in source code/comments
- [ ] No accessible backup files
- [ ] Proper cache headers on sensitive pages
- [ ] Autocomplete disabled on sensitive forms
- [ ] Strong TLS configuration (TLS 1.2+)
- [ ] No accessible log files
- [ ] No directory listing enabled
- [ ] Referrer policy implemented
- [ ] Strong cryptographic algorithms used
- [ ] High-entropy session tokens
- [ ] Sensitive data encrypted at rest
- [ ] No verbose debugging enabled in production

---

## Tools Reference

### Command Line Tools
```bash
# SSL/TLS testing
testssl.sh target.com
sslscan target.com
nmap --script ssl-* target.com

# Header analysis
curl -I https://target.com
wget --server-response https://target.com

# Content discovery
ffuf -w wordlist.txt -u https://target.com/FUZZ
gobuster dir -u https://target.com -w wordlist.txt
```

### Browser Developer Tools
```
1. Network Tab: Inspect all requests/responses
2. Application Tab: Check storage (cookies, localStorage, etc.)
3. Console: Run JavaScript to enumerate storage
4. Sources: Review JavaScript files for secrets
```

---

## Notes

- Always test in authorized environments only
- Document all findings with screenshots and evidence
- Verify findings to eliminate false positives
- Consider data classification (PII, PCI, PHI, etc.)
- Check compliance requirements (GDPR, PCI-DSS, HIPAA)
- Test both in-transit and at-rest encryption
- Review data retention and disposal practices

---

*Last Updated: October 2024*
