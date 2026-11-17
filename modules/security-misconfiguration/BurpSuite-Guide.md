# Burp Suite Guide: Security Misconfiguration

Practical workflow to detect, validate, and report security misconfigurations with Burp Suite (Community/Professional).

## 1) Project Setup and Scope
- Target tab: define scope (domain, in-scope paths). Exclude logout, payment actions
- Proxy your browser; install Burp CA certificate for HTTPS interception
- Turn on Logger/HTTP history for full evidence chain

## 2) Passive Recon (No requests needed)
- Dashboard/Issues: watch automatic findings (missing headers, mixed content)
- Proxy > HTTP history: review responses for security headers and server banners
- Repeater/Inspector: quickly view header sets across endpoints
- Tools: Use Engagement tools > Content discovery (Pro) for common files

Checklist (Passive):
- HSTS, CSP, XFO, XCTO, Referrer-Policy, Permissions-Policy present?
- Server/version banners removed?
- Cookies flagged Secure, HttpOnly, SameSite?
- Mixed content warnings?

## 3) Active Recon and Probing
- Repeater templates:
  - OPTIONS / -> check Allow header (PUT/DELETE/TRACE)
  - Origin: https://evil.tld -> check CORS headers
  - X-Forwarded-Host poisoning probes
- Intruder (or Turbo Intruder extension):
  - Fuzz common paths: /.git/, /.env, /backup/, /admin/, /config/
  - Fuzz default credentials on basic auth endpoints (rate-limit aware)

## 4) CORS Testing
- Send Origin variations: https://evil.tld, null, random subdomains
- Observe Access-Control-Allow-Origin and -Credentials
- If ACAC: true and origin reflected/ wildcard -> attempt credentialed fetch proof (in-browser)

## 5) Dangerous Methods and WebDAV
- OPTIONS to enumerate methods
- PUT probe: upload text file to non-executable path (test.txt); verify retrieval
- If WebDAV suspected, try PROPFIND, MOVE, COPY and directory operations

## 6) Error Handling and Verbose Messages
- Send invalid types, long values, missing params; watch for stack traces, SQL errors
- Negotiate content types via Accept: application/json, text/xml to surface debug formats

## 7) Directory Listing and Sensitive Artifacts
- Use Content discovery (Pro) or extensions (Param Miner, Content Discovery) to enumerate
- Manually try /.git/, /.svn/, /backup/, /old/, /debug/, /actuator
- If .git exposed, capture evidence and note use of git-dumper offline

## 8) Cloud and Metadata Indicators
- Inspect JS/HTML for bucket names and endpoints
- Check signed URLs and ACL hints; do not alter/abuse production cloud assets

## 9) TLS/HTTPS Observations
- Note mixed content in Proxy history
- Use extensions to pull SSL/TLS info or rely on external tools (testssl.sh, SSL Labs)

## 10) Reporting in Burp
- Add Issues (User) with evidence: request/response, screenshots
- Tag items by category (Headers, Methods, CORS, Directory, Cloud)
- Export HTML report (Pro) or save annotated history

## 11) Example Repeater Requests
- OPTIONS Probe
  OPTIONS / HTTP/1.1
  Host: target.tld
  Origin: https://evil.tld
  Accept: */*

- CORS Probe
  GET /api HTTP/1.1
  Host: target.tld
  Origin: https://evil.tld
  Accept: application/json

- PUT Probe (non-executable)
  PUT /test.txt HTTP/1.1
  Host: target.tld
  Content-Length: 6

  owned

## 12) Extensions to Install
- Param Miner (header and parameter discovery)
- CSP Auditor/Scanner, Logger++
- Active Scan++ (Pro), Java Deserialization Scanner (contextual)
- Reflected Parameters, Content Type Converter

## 13) Safe Practice
- Prefer passive first; avoid destructive actions without approval
- Rate-limit intrusions; set throttle in Intruder
- Don’t upload executables; use text proof files
- Capture exact evidence; keep timestamps and scope clear

## Cross-Links
- See: Payloads.md for ready payloads
- See: Reproduce.md for step-by-step lab
- See: Remediation.md for fixes to recommend
