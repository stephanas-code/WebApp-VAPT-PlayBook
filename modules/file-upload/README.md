# File Upload Vulnerabilities - README

## Overview

File upload vulnerabilities represent a critical class of web application security issues that occur when applications fail to properly validate, sanitize, or restrict files uploaded by users. These vulnerabilities can lead to severe consequences including remote code execution (RCE), server compromise, data theft, and complete system takeover.

## What are File Upload Vulnerabilities?

File upload vulnerabilities arise when:
- Applications accept file uploads without proper validation
- File type restrictions can be bypassed
- Uploaded files are stored in web-accessible directories
- File content is not properly sanitized
- Server-side execution of uploaded files is possible

## Types of File Upload Vulnerabilities

### 1. **Unrestricted File Upload**
- **Description**: Application accepts any file type without validation
- **Impact**: Allows attackers to upload malicious files (web shells, executables, scripts)
- **Common Scenarios**:
  - No file type validation
  - Missing MIME type checks
  - No file extension restrictions

### 2. **Remote Code Execution via File Upload**
- **Description**: Uploaded files can be executed on the server
- **Impact**: Complete server compromise, data theft, backdoor installation
- **Common Scenarios**:
  - PHP/JSP/ASP files uploaded to executable directories
  - Web shells providing remote access
  - Malicious scripts executed server-side

### 3. **Extension Bypass Techniques**
- **Description**: Circumventing file extension restrictions
- **Impact**: Upload of malicious files despite security controls
- **Common Techniques**:
  - Double extensions (e.g., shell.php.jpg)
  - Null byte injection (e.g., shell.php%00.jpg)
  - Case variation (e.g., shell.PhP)
  - Alternative extensions (e.g., .phtml, .php3, .php5)
  - Content-Type manipulation

### 4. **MIME Type Bypass**
- **Description**: Bypassing MIME type validation
- **Impact**: Upload malicious files with spoofed content types
- **Common Techniques**:
  - Modifying Content-Type header
  - Magic byte manipulation
  - Polyglot files (valid as multiple file types)

### 5. **Path Traversal via Upload**
- **Description**: Controlling upload destination through filename manipulation
- **Impact**: Overwriting critical files, placing files in sensitive locations
- **Common Techniques**:
  - Directory traversal sequences (../ or ..\)
  - Absolute path specification
  - Overwriting configuration files

### 6. **Archive-Based Attacks**
- **Description**: Exploiting archive extraction vulnerabilities
- **Impact**: Zip slip attacks, path traversal, resource exhaustion
- **Common Techniques**:
  - Malicious ZIP files with traversal paths
  - Zip bombs (compression bombs)
  - Symlink attacks in archives

## Risk Assessment

### Severity: **CRITICAL to HIGH**

### CVSS Base Scores:
- **RCE via File Upload**: 9.8-10.0 (Critical)
- **Unrestricted Upload**: 7.5-8.9 (High)
- **Extension Bypass**: 7.0-8.5 (High to Critical)

### Likelihood: **HIGH**
- Common in web applications with user content
- Frequently misconfigured upload mechanisms
- Often overlooked in security reviews

## Business Impact

### Direct Impacts:
1. **Server Compromise**
   - Complete control over application server
   - Lateral movement to internal networks
   - Installation of persistent backdoors

2. **Data Breach**
   - Access to sensitive databases
   - Theft of customer information
   - Exposure of proprietary data

3. **Service Disruption**
   - Website defacement
   - Denial of service attacks
   - Resource exhaustion

4. **Malware Distribution**
   - Hosting malicious content
   - Phishing pages
   - Drive-by download attacks

### Financial Consequences:
- **Incident Response**: $500K - $2M+
- **Regulatory Fines**: GDPR violations up to €20M or 4% annual revenue
- **Reputation Damage**: Loss of customer trust and business
- **Legal Liability**: Lawsuits from affected parties

## Real-World Examples

### Case Study 1: Government Portal Breach
- **Vulnerability**: Unrestricted PHP file upload
- **Impact**: Complete server compromise, data theft of citizen records
- **Root Cause**: No file type validation, executable directory

### Case Study 2: E-commerce Platform RCE
- **Vulnerability**: Extension bypass via double extension
- **Impact**: Web shell upload, payment data theft
- **Root Cause**: Client-side only validation, weak extension check

### Case Study 3: Social Media Platform
- **Vulnerability**: MIME type bypass in profile picture upload
- **Impact**: Stored XSS, phishing attacks
- **Root Cause**: Insufficient content validation

## Attack Prerequisites

- Access to file upload functionality
- Ability to craft malicious files
- Knowledge of server-side technology (PHP, ASP, JSP, etc.)
- Understanding of file system structure
- (Optional) Ability to trigger/access uploaded files

## Detection Indicators

### Application-Level:
- Upload endpoints accepting multiple file types
- No clear file type restrictions in UI
- Client-side only validation (JavaScript)
- Predictable upload directories
- Direct access to uploaded files via URL

### Security Testing:
- Missing Content-Type validation
- No file extension whitelist
- Executable directories for uploads
- Insufficient file content inspection
- No malware scanning

## Module Contents

This module contains comprehensive documentation on file upload vulnerabilities:

1. **[Payloads.md](./Payloads.md)**: Annotated payloads and attack vectors
   - Web shell payloads for various technologies
   - Extension bypass techniques
   - MIME type manipulation examples
   - Path traversal payloads

2. **[Reproduce.md](./Reproduce.md)**: Step-by-step reproduction guides
   - Manual testing procedures
   - Automated testing approaches
   - Validation methods

3. **[BurpSuite-Guide.md](./BurpSuite-Guide.md)**: Burp Suite testing methodology
   - Request interception and modification
   - Extension manipulation
   - Content-Type bypasses
   - Intruder-based testing

4. **[Remediation.md](./Remediation.md)**: Secure coding and mitigation strategies
   - Input validation best practices
   - Secure file handling
   - Configuration hardening
   - Defense in depth approaches

## Quick Start

### For Security Testers:
1. Review [Payloads.md](./Payloads.md) for attack vectors
2. Follow [BurpSuite-Guide.md](./BurpSuite-Guide.md) for hands-on testing
3. Use [Reproduce.md](./Reproduce.md) for detailed test scenarios

### For Developers:
1. Study [Remediation.md](./Remediation.md) for secure implementation
2. Implement defense-in-depth controls
3. Review code against vulnerability patterns

### For Security Teams:
1. Integrate upload testing into SDLC
2. Implement automated scanning
3. Conduct regular security assessments

## Testing Checklist

- [ ] Test with various file extensions (.php, .jsp, .asp, etc.)
- [ ] Attempt extension bypass techniques
- [ ] Manipulate Content-Type headers
- [ ] Test with polyglot files
- [ ] Try path traversal in filenames
- [ ] Upload oversized files (DoS testing)
- [ ] Test archive extraction (if applicable)
- [ ] Verify uploaded file is not executable
- [ ] Check for direct file access via URL
- [ ] Test with malformed/corrupted files

## Common Vulnerable Patterns

### Pattern 1: Client-Side Only Validation
```javascript
// VULNERABLE: JavaScript validation only
if (!file.name.endsWith('.jpg')) {
  alert('Only JPG files allowed');
  return;
}
```

### Pattern 2: Blacklist-Based Extension Filtering
```php
// VULNERABLE: Blacklist approach
$blocked = ['.php', '.exe', '.sh'];
if (in_array($extension, $blocked)) {
  die('File type not allowed');
}
```

### Pattern 3: Insufficient MIME Validation
```php
// VULNERABLE: Trusting client-provided MIME type
if ($_FILES['file']['type'] != 'image/jpeg') {
  die('Invalid file type');
}
```

## References

- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [OWASP Top 10 2021 - A04:2021 Insecure Design](https://owasp.org/Top10/A04_2021-Insecure_Design/)
- [PortSwigger: File Upload Vulnerabilities](https://portswigger.net/web-security/file-upload)

## Contributing

Contributions to improve this module are welcome. Please refer to the main repository's [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.

## License

This documentation is part of the Web Application VAPT Guide and is licensed under the same terms. See [LICENSE](../../LICENSE) for details.

---

**Last Updated**: October 2025  
**Module Version**: 1.0  
**Maintainer**: Security Research Team
