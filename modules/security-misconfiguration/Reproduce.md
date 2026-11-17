# Reproduce: Security Misconfiguration

Hands-on steps to reproduce and validate common misconfigurations in a safe lab. Use a local DVWA, Juice Shop, or a containerized vulnerable stack.

## Lab Setup (Quick)
- Docker: docker run -d -p 8080:3000 bkimminich/juice-shop
- Or DVWA: docker run --rm -it -p 8080:80 vulnerables/web-dvwa
- Ensure you have: curl, nmap, testssl.sh, Nikto, Burp Suite

## 1) Missing Security Headers
1. curl -I http://localhost:8080/
2. Observe absence of HSTS, CSP, XFO, XCTO, Referrer-Policy
3. Risk: Clickjacking, MIME sniffing, XSS surface
4. Evidence: Raw response headers screenshot

## 2) Dangerous HTTP Methods (PUT/DELETE/TRACE)
1. curl -i -X OPTIONS http://localhost:8080/
2. If PUT allowed: curl -i -X PUT --data 'owned' http://localhost:8080/test.txt
3. Navigate to /test.txt; confirm content served [impact: write to web root]
4. Mitigation validation: server returns 405 for PUT/DELETE

## 3) Directory Listing and Sensitive Files
1. Try common paths: /backup/, /.git/, /.env, /config/, /admin/
2. If directory listing enabled, list files; download backups/configs
3. Use git-dumper on exposed .git to reconstruct source
4. Impact: credential leakage, business logic disclosure

## 4) Default Credentials on Admin Panels
1. Discover panels: /admin, /phpmyadmin, /jenkins, /grafana
2. Try defaults (admin/admin, admin/password, root/toor)
3. If login succeeds, capture proof and enumerate version/plugins
4. Mitigation: change defaults, MFA, IP restrict

## 5) CORS Misconfiguration
1. Send request with Origin: https://evil.tld via Burp Repeater
2. If ACAO echoes origin and ACAC: true, perform credentialed fetch in browser console
3. Document ability to read sensitive API data cross-origin
4. Fix: strict allowlist; no ACAC with broad origins

## 6) Verbose Error Messages
1. Send malformed inputs to endpoints (e.g., id=AAAA, or large payloads)
2. Observe stack traces, framework versions, SQL errors
3. Risk: info leak aiding exploitation (SQLi, RCE)
4. Fix: generic errors in prod; detailed logs server-side

## 7) Cloud Storage Exposure (S3 Example)
1. Identify bucket name (from assets URLs or code)
2. Test listing: https://bucket.s3.amazonaws.com/?list-type=2
3. If public, list/download objects; attempt PUT if allowed
4. Fix: block public access, private ACLs, bucket policies

## 8) TLS Weaknesses
1. testssl.sh https://target
2. Note weak ciphers, TLS1.0/1.1, missing HSTS
3. Fix: modern TLS config, HSTS, automatic certs

## 9) Framework Defaults (Spring Boot, Django, Laravel)
1. Check /actuator, /actuator/env, /health; or /_profiler, /debug
2. If accessible, enumerate sensitive data or actions
3. Fix: disable in prod, secure with auth/IP allowlist

## 10) Cache/Proxy Issues
1. Send poisoned headers: X-Forwarded-Host: evil.tld
2. Observe reflected URLs or cache keys; try storing variant
3. Impact: cache poisoning, host header injection
4. Fix: canonical host, sanitize hop-by-hop headers

## Evidence Template
- Title: Misconfiguration name
- Endpoint/Scope: URL(s)
- Steps: Repro steps executed
- Observations: Responses, headers, screenshots
- Impact: Business and technical
- Severity: CVSS vector (if applicable)
- Recommendations: Link to Remediation.md

## Cleanup
- Remove test files uploaded (test.txt)
- Revert containers; rotate any exposed secrets
- Confirm 405/403 for previously vulnerable actions
