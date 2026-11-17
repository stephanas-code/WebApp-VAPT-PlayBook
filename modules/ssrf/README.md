# Server-Side Request Forgery (SSRF)

## Overview

Server-Side Request Forgery (SSRF) is a security vulnerability that allows an attacker to manipulate a server-side application to make unauthorized HTTP requests to arbitrary domains or internal resources. The attacker essentially tricks the server into acting as a proxy, accessing resources that would otherwise be inaccessible from the attacker's position.

## What is SSRF?

SSRF occurs when a web application fetches remote resources without properly validating user-supplied URLs. This vulnerability enables attackers to:

- **Access Internal Systems**: Bypass firewalls and access internal services, databases, and APIs that are not exposed to the internet
- **Exploit Cloud Metadata Services**: Retrieve sensitive information from cloud provider metadata endpoints (AWS, Azure, GCP)
- **Port Scanning**: Map internal networks and identify running services
- **Bypass Access Controls**: Access restricted resources using the server's trusted position
- **Data Exfiltration**: Read sensitive files and configurations from internal systems
- **Execute Remote Code**: Chain SSRF with other vulnerabilities for code execution

## How SSRF Works

### Attack Flow

```
1. Attacker identifies user-controllable URL parameter
   ↓
2. Attacker crafts malicious URL (internal IP, cloud metadata, etc.)
   ↓
3. Vulnerable application processes the URL without validation
   ↓
4. Server makes request to attacker-specified target
   ↓
5. Server returns response to attacker
   ↓
6. Attacker gains unauthorized access to internal resources
```

## Common SSRF Attack Vectors

### 1. Basic SSRF
- Direct manipulation of URL parameters
- Accessing localhost/127.0.0.1
- Targeting internal IP ranges (10.x.x.x, 192.168.x.x, 172.16.x.x)

### 2. Cloud Metadata Exploitation
- **AWS**: http://169.254.169.254/latest/meta-data/
- **Azure**: http://169.254.169.254/metadata/instance
- **GCP**: http://metadata.google.internal/computeMetadata/v1/

### 3. Protocol-Based SSRF
- file:// - Read local files
- dict:// - Query dictionary servers
- gopher:// - Multipurpose protocol exploitation
- ftp:// - Access FTP servers

### 4. Blind SSRF
- No direct response visible to attacker
- Detection via out-of-band techniques (DNS, HTTP callbacks)
- Time-based detection

## Vulnerable Code Examples

### PHP - Vulnerable
```php
<?php
// Vulnerable: No URL validation
$url = $_GET['url'];
$content = file_get_contents($url);
echo $content;
?>
```

### Python - Vulnerable
```python
import requests
from flask import Flask, request

app = Flask(__name__)

@app.route('/fetch')
def fetch_url():
    # Vulnerable: Direct use of user input
    url = request.args.get('url')
    response = requests.get(url)
    return response.content
```

### Node.js - Vulnerable
```javascript
const express = require('express');
const axios = require('axios');
const app = express();

app.get('/proxy', async (req, res) => {
    // Vulnerable: No validation
    const url = req.query.url;
    const response = await axios.get(url);
    res.send(response.data);
});
```

### Java - Vulnerable
```java
@GetMapping("/fetch")
public String fetchUrl(@RequestParam String url) throws IOException {
    // Vulnerable: No URL validation
    URL urlObj = new URL(url);
    URLConnection conn = urlObj.openConnection();
    return IOUtils.toString(conn.getInputStream(), StandardCharsets.UTF_8);
}
```

## Business Impact

### Critical Risks

1. **Data Breach**
   - Exposure of sensitive internal data
   - Access to customer information
   - Theft of credentials and API keys
   - **Financial Impact**: $100K - $5M+ in breach costs

2. **Infrastructure Compromise**
   - Unauthorized access to internal systems
   - Cloud account takeover via metadata
   - Database access and manipulation
   - **Operational Impact**: System downtime, service disruption

3. **Compliance Violations**
   - GDPR, PCI-DSS, HIPAA violations
   - Regulatory fines and penalties
   - **Legal Impact**: $10K - $20M+ in fines

4. **Reputation Damage**
   - Loss of customer trust
   - Media exposure
   - Stock price impact
   - **Business Impact**: Customer churn, reduced market valuation

### Real-World Examples

- **Capital One (2019)**: SSRF exploitation of AWS metadata led to breach of 100M+ customer records
- **Stripe (2020)**: SSRF vulnerability disclosed through bug bounty ($25,000 payout)
- **Uber**: SSRF used to access internal services and escalate privileges

## Risk Analysis

### CVSS v3.1 Scoring: **9.1 (Critical)**

**Vector String**: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

- **Attack Vector (AV:N)**: Network - Exploitable remotely
- **Attack Complexity (AC:L)**: Low - No special conditions required
- **Privileges Required (PR:N)**: None - No authentication needed
- **User Interaction (UI:N)**: None - No user action required
- **Scope (S:U)**: Unchanged - Impacts same security authority
- **Confidentiality (C:H)**: High - Total information disclosure
- **Integrity (I:H)**: High - Total compromise of integrity possible
- **Availability (A:N)**: None - No impact on availability

### OWASP Classification

- **OWASP Top 10 2021**: A10:2021 - Server-Side Request Forgery
- **CWE-918**: Server-Side Request Forgery (SSRF)

### Likelihood Assessment

| Factor | Rating | Justification |
|--------|--------|---------------|
| Exploitability | High | Simple to exploit with basic HTTP knowledge |
| Prevalence | Medium | Common in applications with URL fetching features |
| Detectability | Medium | Can be identified through parameter testing |
| Technical Impact | Critical | Access to internal systems and cloud metadata |

## Testing Methodology

### 1. Identify Potential SSRF Entry Points
- URL parameters
- File upload functions (remote fetch)
- Webhook implementations
- PDF generators (external resources)
- API integrations
- Image/document processors

### 2. Test for Basic SSRF
```bash
# Test localhost access
http://example.com/fetch?url=http://localhost:80
http://example.com/fetch?url=http://127.0.0.1:80

# Test internal IP ranges
http://example.com/fetch?url=http://192.168.1.1
http://example.com/fetch?url=http://10.0.0.1
```

### 3. Test Cloud Metadata Access
```bash
# AWS metadata
http://example.com/fetch?url=http://169.254.169.254/latest/meta-data/

# Azure metadata
http://example.com/fetch?url=http://169.254.169.254/metadata/instance?api-version=2021-02-01
```

### 4. Test Protocol Handlers
```bash
# File protocol
http://example.com/fetch?url=file:///etc/passwd

# Gopher protocol
http://example.com/fetch?url=gopher://127.0.0.1:25
```

### 5. Bypass Techniques
- URL encoding: `http://127.0.0.1` → `http://%31%32%37%2e%30%2e%30%2e%31`
- Decimal notation: `http://2130706433` (127.0.0.1)
- Hexadecimal: `http://0x7f.0x0.0x0.0x1`
- Domain redirection: Using services that redirect to internal IPs
- DNS rebinding attacks

## Detection Methods

### 1. Automated Scanning
- Use tools like Burp Suite, OWASP ZAP
- Custom scripts for parameter fuzzing
- Cloud-specific SSRF scanners

### 2. Manual Testing
- Parameter manipulation
- Response analysis
- Time-based detection for blind SSRF

### 3. Monitoring
- Network traffic analysis
- Unusual outbound connections
- Access to metadata endpoints
- Internal service access logs

## Prevention Best Practices

### Defense in Depth Strategy

1. **Input Validation**
   - Whitelist allowed protocols (http, https only)
   - Whitelist allowed domains/IPs
   - Validate URL format and structure

2. **Network Segmentation**
   - Restrict outbound connections from application servers
   - Use separate network zones for different services
   - Implement egress filtering

3. **Disable Unnecessary Protocols**
   - Block file://, gopher://, dict://, ftp://
   - Only allow http:// and https://

4. **Metadata Service Protection**
   - Block access to 169.254.169.254
   - Use IMDSv2 (AWS) with token authentication
   - Implement network-level blocks

5. **Response Handling**
   - Don't return raw responses to users
   - Sanitize and validate responses
   - Implement timeout mechanisms

## References

- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [CWE-918: Server-Side Request Forgery](https://cwe.mitre.org/data/definitions/918.html)
- [PortSwigger: Server-Side Request Forgery](https://portswigger.net/web-security/ssrf)
- [HackerOne SSRF Reports](https://hackerone.com/reports?filter=ssrf)

## Related Modules

- [Authentication Bypass](../auth-bypass/)
- [IDOR](../idor/)
- [SQL Injection](../sqli/)

## Additional Resources

- [Payloads.md](./Payloads.md) - Comprehensive SSRF payload collection
- [Remediation.md](./Remediation.md) - Detailed remediation strategies
- [Reproduce.md](./Reproduce.md) - Step-by-step exploitation guides
- [BurpSuite-Guide.md](./BurpSuite-Guide.md) - Burp Suite testing methodology
