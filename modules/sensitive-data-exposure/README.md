# Sensitive Data Exposure

## Overview
Sensitive Data Exposure occurs when applications fail to adequately protect sensitive information such as passwords, credit card numbers, health records, personal data, and business secrets. This vulnerability has become increasingly critical in the modern web, where vast amounts of sensitive data are collected, processed, and stored.

## What is Sensitive Data Exposure?

Sensitive Data Exposure represents a broad category of vulnerabilities where applications fail to protect sensitive information from unauthorized access. This includes:

- **Cryptographic Failures**: Weak or missing encryption, improper key management, use of deprecated algorithms
- **Plaintext Storage**: Storing sensitive data without encryption in databases, logs, or configuration files
- **Inadequate Transport Security**: Transmitting sensitive data over unencrypted connections (HTTP instead of HTTPS)
- **Information Leakage**: Exposing sensitive data through error messages, API responses, or client-side code
- **Insufficient Access Controls**: Failing to properly restrict access to sensitive data

## Common Vulnerability Scenarios

### 1. Weak Cryptographic Algorithms
```
Application uses MD5 or SHA1 for password hashing
Use of deprecated SSL/TLS versions (SSLv2, SSLv3, TLS 1.0)
Weak cipher suites that are vulnerable to attacks
```

### 2. Plaintext Data Storage
```
Passwords stored in plaintext in database
Credit card numbers stored without encryption
API keys and secrets committed to version control
Sensitive data in application logs
```

### 3. Insecure Data Transmission
```
Sensitive forms submitted over HTTP
Missing Secure flag on cookies containing session tokens
API endpoints transmitting PII without TLS
Downgrade attacks from HTTPS to HTTP
```

### 4. Information Leakage
```
Detailed error messages revealing system information
Source code comments containing credentials
Backup files (.bak, .old) accessible via web
Git repositories exposed (.git directory)
```

### 5. Client-Side Data Exposure
```
Sensitive data in JavaScript variables
API keys hardcoded in mobile apps
Personal information cached in browser storage
Sensitive data in URL parameters
```

## Risk Assessment

### Severity: **HIGH to CRITICAL**

### Likelihood Factors:
- Extremely common in legacy applications
- Often overlooked during development
- Poor key management practices
- Lack of security awareness among developers

### Impact:
- **Data Breaches**: Mass exposure of user data
- **Identity Theft**: Stolen personal information used fraudulently
- **Financial Loss**: Direct theft, regulatory fines, lawsuits
- **Compliance Violations**: GDPR, PCI-DSS, HIPAA penalties
- **Reputation Damage**: Loss of customer trust and business

## Business Impact

### Financial Consequences
- **Regulatory Fines**: GDPR fines up to €20 million or 4% of annual revenue
- **PCI-DSS Violations**: Fines from $5,000 to $500,000 per incident
- **Legal Costs**: Class-action lawsuits and settlements
- **Forensic Investigation**: Cost of incident response and analysis

### Operational Impact
- **Service Disruption**: Downtime during incident response
- **Customer Churn**: Loss of users due to trust issues
- **Increased Costs**: Enhanced security measures post-breach
- **Brand Damage**: Long-term reputation harm

### Compliance Impact
- **GDPR**: European data protection regulation
- **PCI-DSS**: Payment card industry standards
- **HIPAA**: Healthcare data protection requirements
- **SOX**: Financial data integrity requirements

## Detection Methods

### Manual Testing
1. **Traffic Analysis**
   - Intercept requests with Burp Suite/OWASP ZAP
   - Check for sensitive data in HTTP traffic
   - Verify HTTPS implementation and certificate validity
   - Test for protocol downgrade vulnerabilities

2. **Source Code Review**
   - Search for hardcoded credentials
   - Identify weak cryptographic functions
   - Check for plaintext logging of sensitive data
   - Review error handling mechanisms

3. **Database Analysis**
   - Examine how passwords are stored
   - Check encryption of sensitive columns
   - Review access control mechanisms
   - Identify backup and log storage practices

4. **Configuration Review**
   - Check SSL/TLS configuration
   - Review cipher suite selections
   - Verify secure cookie flags
   - Examine security headers

### Automated Testing
1. **SSL/TLS Scanners**
   - SSLyze
   - testssl.sh
   - Qualys SSL Labs

2. **Static Analysis**
   - Semgrep
   - SonarQube
   - Bandit (Python)
   - Brakeman (Ruby)

3. **Dynamic Scanners**
   - Burp Suite Pro
   - OWASP ZAP
   - Acunetix
   - Nessus

4. **Secret Scanning**
   - TruffleHog
   - GitLeaks
   - GitHub Secret Scanning

## Real-World Examples

### Case Study 1: Adobe Data Breach (2013)
- **Issue**: 150 million user records exposed with weakly encrypted passwords
- **Cause**: Use of ECB mode encryption (reversible)
- **Impact**: $1.1 million settlement, massive reputation damage

### Case Study 2: Equifax Breach (2017)
- **Issue**: 147 million records compromised
- **Cause**: Unpatched vulnerability + unencrypted data transmission
- **Impact**: $575 million settlement, executive resignations

### Case Study 3: British Airways (2018)
- **Issue**: 400,000 customer payment cards stolen
- **Cause**: Compromised script harvesting data from checkout forms
- **Impact**: £20 million GDPR fine

### Case Study 4: Capital One (2019)
- **Issue**: 100 million credit applications exposed
- **Cause**: Misconfigured firewall + inadequate access controls
- **Impact**: $80 million fine, ongoing lawsuits

## Testing Checklist

- [ ] Verify HTTPS is enforced on all sensitive pages
- [ ] Check SSL/TLS configuration (no weak ciphers/protocols)
- [ ] Test for sensitive data in HTTP traffic
- [ ] Examine cookie security flags (Secure, HttpOnly, SameSite)
- [ ] Review password hashing mechanisms
- [ ] Check for hardcoded credentials in source code
- [ ] Test error messages for information disclosure
- [ ] Verify sensitive data is not cached client-side
- [ ] Check for sensitive data in URL parameters
- [ ] Examine backup files and .git directory exposure
- [ ] Test for sensitive data in logs and error traces
- [ ] Verify proper key management practices
- [ ] Check autocomplete settings on sensitive forms
- [ ] Test for sensitive data in API responses
- [ ] Review data retention and deletion policies

## Prevention Guidelines

### For Developers
1. **Use Strong Encryption**
   - AES-256 for symmetric encryption
   - RSA-2048+ or ECC for asymmetric encryption
   - Bcrypt, Argon2, or PBKDF2 for password hashing

2. **Secure Data in Transit**
   - Enforce HTTPS everywhere
   - Use TLS 1.2 or higher
   - Implement HSTS headers
   - Use secure cipher suites

3. **Secure Data at Rest**
   - Encrypt sensitive database columns
   - Never store plaintext passwords
   - Use secure key management systems
   - Encrypt backups and logs

4. **Minimize Data Exposure**
   - Collect only necessary data
   - Implement data retention policies
   - Avoid logging sensitive information
   - Sanitize error messages

### For Organizations
1. **Data Classification**
   - Identify and classify sensitive data
   - Implement appropriate controls per classification
   - Regular data inventory audits

2. **Key Management**
   - Use dedicated key management systems (KMS)
   - Rotate keys regularly
   - Separate key storage from encrypted data
   - Implement proper key lifecycle management

3. **Security Training**
   - Regular developer security training
   - Security champions program
   - Secure coding standards

4. **Compliance**
   - Regular security audits
   - Penetration testing
   - Compliance assessments (PCI-DSS, GDPR, HIPAA)

## Related Vulnerabilities
- **Broken Authentication**: Often involves weak password storage
- **Security Misconfiguration**: Can lead to data exposure
- **Insufficient Logging & Monitoring**: Prevents detection of data breaches
- **Broken Access Control**: May allow unauthorized data access

## Tools & Resources

### Testing Tools
- **Burp Suite**: Web proxy for traffic analysis
- **OWASP ZAP**: Open-source security scanner
- **SSLyze**: SSL/TLS configuration scanner
- **testssl.sh**: Command-line SSL/TLS testing
- **TruffleHog**: Secret scanning in git repositories

### Learning Resources
- OWASP Cryptographic Storage Cheat Sheet
- NIST Cryptographic Standards
- CWE-311: Missing Encryption of Sensitive Data
- CWE-312: Cleartext Storage of Sensitive Information
- CWE-319: Cleartext Transmission of Sensitive Information

### Standards & Compliance
- **PCI-DSS**: Payment Card Industry Data Security Standard
- **GDPR**: General Data Protection Regulation
- **HIPAA**: Health Insurance Portability and Accountability Act
- **NIST SP 800-53**: Security and Privacy Controls

## References
- [OWASP Top 10 - A02:2021 Cryptographic Failures](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)
- [OWASP Sensitive Data Exposure](https://owasp.org/www-community/vulnerabilities/)
- [CWE-359: Exposure of Private Information](https://cwe.mitre.org/data/definitions/359.html)
- [NIST Cryptographic Standards](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines)

## Next Steps
1. Review [Payloads.md](./Payloads.md) for testing scenarios
2. Study [Reproduce.md](./Reproduce.md) for hands-on exercises
3. Learn [Remediation.md](./Remediation.md) for fixing vulnerabilities
4. Explore [BurpSuite-Guide.md](./BurpSuite-Guide.md) for professional testing

---
**Note**: This module is for educational purposes only. Always obtain proper authorization before testing systems.
