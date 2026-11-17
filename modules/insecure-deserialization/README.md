# Insecure Deserialization

## Overview

Insecure Deserialization is a critical vulnerability that occurs when untrusted data is used to reconstruct objects in memory. When an application deserializes user-controlled data without proper validation, attackers can manipulate serialized objects to execute arbitrary code, bypass authentication, perform privilege escalation, or achieve denial of service.

## What is Serialization?

**Serialization** is the process of converting an object's state into a format that can be stored or transmitted (typically as a byte stream or text). **Deserialization** is the reverse process—reconstructing the object from the serialized format.

### Common Serialization Formats:
- **Java**: Native Java serialization, XML (XMLDecoder), JSON
- **PHP**: `serialize()` / `unserialize()`, Phar archives
- **.NET**: BinaryFormatter, NetDataContractSerializer, JSON.NET
- **Python**: Pickle, PyYAML, JSON
- **Ruby**: Marshal, YAML

## Why is it Dangerous?

When deserializing untrusted data, the application reconstructs objects with their properties and methods. Attackers can craft malicious serialized objects that:

1. **Execute Arbitrary Code**: By leveraging "magic methods" or "gadget chains" that execute during deserialization
2. **Manipulate Application Logic**: Modify object properties to bypass authentication or authorization
3. **Cause Denial of Service**: Create objects that consume excessive resources
4. **Read/Write Files**: Access sensitive files or overwrite critical data

## Common Attack Scenarios

### 1. Remote Code Execution (RCE)
The most severe impact—attackers execute system commands by exploiting gadget chains in application dependencies.

**Example Flow**:
```
1. Application accepts serialized user session data
2. Attacker crafts malicious serialized object with RCE payload
3. Application deserializes the object
4. Magic methods trigger during deserialization
5. Gadget chain executes arbitrary code
6. Attacker gains server access
```

### 2. Authentication Bypass
Modifying serialized user objects to escalate privileges or impersonate other users.

**Example**:
```php
// Serialized user object:
O:4:"User":3:{s:2:"id";i:42;s:8:"username";s:5:"alice";s:7:"isAdmin";b:0;}

// Attacker modifies to:
O:4:"User":3:{s:2:"id";i:1;s:8:"username";s:5:"admin";s:7:"isAdmin";b:1;}
```

### 3. SQL Injection via Object Injection
Injecting SQL commands through object properties that are used in database queries.

### 4. Arbitrary File Operations
Manipulating file path properties to read sensitive files or write malicious content.

## Vulnerable Code Examples

### Java (Vulnerable)
```java
// Reading untrusted serialized data
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject(); // VULNERABLE!
```

### PHP (Vulnerable)
```php
// Deserializing user-controlled cookie
$userData = unserialize($_COOKIE['user']); // VULNERABLE!
```

### Python (Vulnerable)
```python
import pickle

# Deserializing untrusted data
user_data = pickle.loads(request.data)  # VULNERABLE!
```

### .NET (Vulnerable)
```csharp
// Using BinaryFormatter on untrusted data
BinaryFormatter formatter = new BinaryFormatter();
Object obj = formatter.Deserialize(stream); // VULNERABLE!
```

## How to Identify

### Black-Box Testing
1. **Look for serialized data** in:
   - Cookies
   - Hidden form fields
   - API request/response bodies
   - HTTP headers
   - URL parameters
   - Local/session storage

2. **Recognize serialization formats**:
   - Java: Starts with `rO0` (base64) or `AC ED 00 05` (hex)
   - PHP: Pattern like `O:4:"User":2:{...}`
   - .NET: Binary data in ViewState or cookies
   - Python: Pickle format with opcodes
   - JSON with `$type` or `__class__` properties

3. **Test for vulnerabilities**:
   - Modify serialized values
   - Inject known gadget chain payloads
   - Monitor for unexpected behavior or errors

### White-Box Testing (Code Review)
Search for:
- Java: `readObject()`, `readUnshared()`, `XMLDecoder`
- PHP: `unserialize()`, `phar://` wrapper
- .NET: `BinaryFormatter`, `NetDataContractSerializer`, `ObjectStateFormatter`
- Python: `pickle.loads()`, `yaml.load()`, `jsonpickle`
- Ruby: `Marshal.load()`, `YAML.load()`

## Business Impact

### Critical Impacts:
- **Complete System Compromise**: Attackers gain full control of the server
- **Data Breach**: Access to all application and database data
- **Lateral Movement**: Compromise of internal network systems
- **Service Disruption**: Denial of service or ransomware attacks

### Business Consequences:
- **Financial Loss**: Incident response costs, regulatory fines, legal fees
- **Reputational Damage**: Loss of customer trust and brand value
- **Compliance Violations**: GDPR, PCI-DSS, HIPAA breaches
- **Operational Impact**: System downtime and recovery efforts

### CVSS Considerations:
- Typically rated **9.0-10.0 (Critical)** when RCE is possible
- **7.0-8.9 (High)** for authentication bypass or data manipulation
- **4.0-6.9 (Medium)** for denial of service scenarios

## Real-World Examples

### Notable CVEs:
- **Apache Commons Collections (CVE-2015-4852)**: Java deserialization RCE affecting WebLogic, JBoss, Jenkins
- **Java RMI Registry (CVE-2017-3241)**: Remote code execution via RMI
- **PHP Phar Deserialization (CVE-2018-19296)**: File operation-based deserialization
- **.NET BinaryFormatter**: Multiple CVEs leading to RCE
- **Python PyYAML (CVE-2017-18342)**: Arbitrary code execution

### Attack Statistics:
- Exploited in 5% of web applications during penetration tests
- Average time to exploit: 15-30 minutes for experienced attackers
- 85% of successful exploits lead to full system compromise

## Testing Checklist

- [ ] Identify all input points that accept serialized data
- [ ] Determine the serialization format and library used
- [ ] Test for basic object manipulation (property modification)
- [ ] Research known gadget chains for the technology stack
- [ ] Test with ysoserial, phpggc, or other exploitation tools
- [ ] Attempt to trigger error messages revealing internal paths
- [ ] Test for denial of service via resource exhaustion
- [ ] Verify if input validation/signature checking exists
- [ ] Check for secure deserialization alternatives
- [ ] Document all findings with proof of concept

## Related Files

- **[Payloads.md](./Payloads.md)**: Comprehensive payload collection for Java, .NET, PHP, Python with detailed annotations
- **[Remediation.md](./Remediation.md)**: Secure coding practices and mitigation strategies
- **[Reproduce.md](./Reproduce.md)**: Step-by-step exploitation guides for different platforms
- **[BurpSuite-Guide.md](./BurpSuite-Guide.md)**: Using Burp Suite for detection and exploitation

## References

### Standards & Documentation:
- [OWASP Top 10 2021 - A08:2021 Software and Data Integrity Failures](https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/)
- [OWASP Deserialization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)
- [CWE-502: Deserialization of Untrusted Data](https://cwe.mitre.org/data/definitions/502.html)
- [PortSwigger Web Security Academy - Insecure Deserialization](https://portswigger.net/web-security/deserialization)

### Tools & Frameworks:
- [ysoserial](https://github.com/frohoff/ysoserial) - Java deserialization exploitation
- [ysoserial.net](https://github.com/pwntester/ysoserial.net) - .NET deserialization payloads
- [phpggc](https://github.com/ambionics/phpggc) - PHP gadget chains
- [GadgetProbe](https://github.com/BishopFox/GadgetProbe) - Java gadget detection
- [Java Deserialization Scanner](https://github.com/federicodotta/Java-Deserialization-Scanner) - Burp extension

### Research Papers:
- "Marshalling Pickles" - Understanding Python deserialization attacks
- "Java Unmarshaller Security" - Oracle security guide
- "Automated Discovery of Deserialization Gadget Chains" - Academic research

## Quick Reference

### Severity: 🔴 CRITICAL
### Prevalence: Medium (5-10% of applications)
### Detectability: Moderate to Difficult
### Exploitation Difficulty: Moderate to Hard
### Impact: Complete System Compromise

---

**Note**: This module focuses on practical vulnerability assessment and penetration testing techniques. Always obtain proper authorization before testing.
