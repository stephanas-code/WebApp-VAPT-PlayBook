# Burp Suite Guide for SQL Injection

This guide outlines a structured approach to discovering and exploiting SQLi using Burp Suite.

## Setup

1. Configure your browser to use Burp Proxy (127.0.0.1:8080)
2. Install CA certificate to intercept HTTPS
3. Turn Intercept on for first requests

## Methodology

- Map the application with Proxy/Target -> Site map
- Identify parameters: query, body, JSON, headers, cookies
- Send candidates to Repeater for controlled testing
- Use Comparer to analyze boolean/time-based differences
- Optionally use Intruder for automation (rate-limit safely)

## Common Tests in Repeater

1) Quote tests: ' and " to trigger errors
2) Boolean toggles: AND 1=1 vs AND 1=2
3) Time-based: SLEEP(5) / WAITFOR DELAY / pg_sleep(5)
4) UNION enumeration with NULL padding
5) Stacked queries (only if safe and supported)

## Example Requests

### GET parameter
```
GET /product?id=1' AND SLEEP(5)--%20- HTTP/1.1
Host: target.local
User-Agent: Mozilla/5.0
Cookie: session=...
```

### POST login
```
POST /login HTTP/1.1
Host: target.local
Content-Type: application/x-www-form-urlencoded

username=admin'--&password=x
```

### JSON body
```
POST /search HTTP/1.1
Host: target.local
Content-Type: application/json

{"q":"test' UNION SELECT NULL-- -"}
```

## Interpreting Results

- 200 with inflated results -> possible UNION/boolean success
- 500/DB error -> injectable point; pivot to parameterization check
- Consistent >5s delay on time payloads -> blind SQLi

## 2026 Modern & AI-Assisted SQLi Workflows
- **Dynamic Identifier & ORDER BY Injection**:
  - Since SQL parameters cannot parameterize table or column names, test sorting parameters (`/api/items?sort=created_at` or `order=ASC`).
  - In Repeater, test boolean conditional expression injection: `?sort=(CASE+WHEN+(1=1)+THEN+created_at+ELSE+id+END)`.
- **Text-to-SQL (Natural Language AI) Prompt Injection**:
  - For applications featuring AI analytics queries ("Ask AI about your data"), intercept the prompt dispatch request.
  - Test if prompt injection causes the LLM to output unsanitized SQL statements (e.g. `SELECT * FROM users WHERE 1=1`) that bypass UI restrictions.
- **AI-Assisted Burp Automation**:
  - Right-click suspicious SQLi response in Repeater → copy as cURL → prompt an AI assistant: *"Analyze this response diff and write an asynchronous Python `httpx` script confirming time-based blind injection with a 99% confidence interval using 5 baseline requests and 5 delayed requests."*

## Useful Burp Features

- Logger, Target scope, Repeater, Intruder (Pitchfork/Cluster Bomb), Decoder, Comparer
- Extensions: SQLiPy, Autorize (for IDOR cross-check), Turbo Intruder (careful)

## Ethics and Safety

- Only test with permission
- Prefer non-destructive payloads
- Rate-limit to avoid DoS

---

Related: [README](./README.md) | [Payloads](./Payloads.md) | [Remediation](./Remediation.md) | [Reproduce](./Reproduce.md)
