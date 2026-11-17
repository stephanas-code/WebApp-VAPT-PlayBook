# Cross-Site Request Forgery (CSRF)

## Introduction
CSRF forces a victim’s browser to execute unwanted actions on a web application in which they are authenticated.

## Contents
- Types and Preconditions
- Examples
- Business Impact
- Detection
- Risk
- Remediation
- Related Docs

## Types and Preconditions
- Stored vs. Reflected CSRF vectors (HTML forms, IMG/JS, CORS/XHR)
- Preconditions: victim authenticated, predictable endpoints, missing CSRF controls

## Examples
Simple state-changing request without token:
```
POST /user/email HTTP/1.1
Host: app.local
Content-Type: application/x-www-form-urlencoded

email=attacker%40mail.com
```
Malicious HTML:
```html
<form action="https://app.local/user/email" method="POST">
  <input type="hidden" name="email" value="attacker@mail.com">
  <input type="submit">
</form>
```

## Business Impact
- Unauthorized transactions/profile changes
- Account takeover (when combined with XSS/CSRF token leak)
- Regulatory exposure and fraud losses

## Detection
- Lack of CSRF tokens or SameSite=None cookies
- Identify state-changing endpoints accepting GET/POST without verification
- Burp Scanner issues; manual crafted PoC works without user intent

## Risk
CVSS v3.1: 8.2 (High) typically if critical actions affected

## Remediation
- Require anti-CSRF tokens (unpredictable, per-request)
- Enforce SameSite=Lax/Strict cookies
- Verify Origin/Referer headers for sensitive requests
- Use double-submit cookie or JWT claim binding
- Block state-changing actions over GET; use POST/PUT/DELETE with CSRF checks
- Re-authentication or step-up auth for high-risk actions

## Related
- [Payloads.md](./Payloads.md)
- [Remediation.md](./Remediation.md)
- [Reproduce.md](./Reproduce.md)
- [BurpSuite-Guide.md](./BurpSuite-Guide.md)
