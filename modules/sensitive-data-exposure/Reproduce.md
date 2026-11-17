# Sensitive Data Exposure - Reproduction Steps

Purpose: Provide clear, step-by-step procedures to reliably reproduce sensitive data exposure issues, including plaintext leaks, insecure transport/storage, weak secrets, and cryptographic failures.

---

## Pre-requisites
- Authorized test scope and accounts (user + admin if applicable)
- Intercepting proxy (Burp Suite) configured with CA installed
- Command-line tools: curl, openssl, testssl.sh, nmap, gobuster/ffuf
- Browser with DevTools; separate profiles for HTTP/HTTPS testing

---

## 1) Insecure Transport (HTTP / TLS)
1. Attempt HTTP access to sensitive endpoints (login, checkout, API):
   - curl -v http://target.tld/login
   - curl -v http://target.tld/api/user
2. Observe behavior:
   - Expected: 301/308 redirect to HTTPS, no sensitive data in response.
   - Vulnerable: Direct 200 over HTTP or mixed content on HTTPS pages.
3. Validate HSTS:
   - Check header on HTTPS response: Strict-Transport-Security with preload.
4. Test TLS versions/ciphers:
   - testssl.sh target.tld; openssl s_client -connect target.tld:443 -tls1 -tls1_1
   - Vulnerable if TLS 1.0/1.1/SSLv3 or weak ciphers accepted.

Evidence: capture responses, screenshots of headers and SSL scan.

---

## 2) Sensitive Cookies and Cache
1. Log in and intercept Set-Cookie headers.
2. Verify flags: Secure, HttpOnly, SameSite.
3. Switch to HTTP (if possible) and repeat; observe cookie sent over cleartext.
4. Test cache policy on sensitive pages:
   - curl -I https://target.tld/account -> Cache-Control, Pragma
5. Logout, press back; verify if sensitive page is retrievable from cache.

Evidence: HTTP history in proxy, headers, back-button behavior.

---

## 3) Sensitive Data in URLs and Referer
1. Trigger flows like password reset, OAuth redirects, search, or profile update.
2. Check if URLs contain tokens, credentials, PII (e.g., reset?token=..., email=... ).
3. Click outbound links to external domains and inspect Referer header leakage in proxy.

Evidence: request URLs and Referer headers showing sensitive values.

---

## 4) Excessive Data in API Responses
1. Browse user endpoints (e.g., /api/user/me, /api/user/{id}).
2. Inspect JSON for fields like password_hash, tokens, SSN, credit_card, internal_notes.
3. Force content negotiation variants (Accept: text/html, application/xml).
4. Attempt mass assignment/over-posting to reveal hidden fields via validation errors.

Evidence: JSON snapshots with redacted-but-visible field names/values.

---

## 5) Client-Side Storage and Source Leaks
1. Open DevTools > Application and inspect localStorage/sessionStorage/IndexedDB.
2. Search JavaScript sources and HTML comments for keys/secrets/URLs.
3. Fetch static assets and grep for patterns (key|token|secret|password).

Evidence: screenshots/logs of stored secrets or credentials in code.

---

## 6) Backup/Config/Log Exposure
1. Enumerate common backup/config paths (/.env, config.php.bak, backup.zip, .git/).
2. Discover directories with listing enabled (/uploads/, /backups/).
3. Attempt to fetch log files (/logs/access.log, /debug.log).

Evidence: downloaded files, listings, sensitive content excerpts.

---

## 7) Error Handling and Verbose Messages
1. Send invalid inputs (negative IDs, overflow, malformed JSON, quotes/apostrophes).
2. Observe error responses for stack traces, file paths, SQL details, internal IPs.

Evidence: captured error responses.

---

## 8) Weak Cryptography Indicators
1. Identify password hashes (MD5/SHA1) in responses or databases if in scope.
2. Detect Base64-encoded credentials; decode samples to confirm exposure.
3. Inspect session/auth token entropy and predictability.

Evidence: hash examples, decoded strings, entropy metrics.

---

## 9) Data at Rest
1. If permitted, review database dumps or ORM models for plaintext storage of sensitive fields.
2. Confirm encryption strategy (field-level vs TDE) and key management approach.

Evidence: schema/docs showing encryption gaps.

---

## Reproduction Report Template
- Finding title: [e.g., Session token sent over HTTP]
- Affected endpoint/page:
- Impacted data elements:
- Steps to reproduce:
- Observed vs expected behavior:
- Technical impact and business risk:
- Evidence (requests/responses, screenshots):
- Suggested remediation:
- References (ASVS controls, policies):

---

Notes:
- Minimize exposure during testing; redact secrets in artifacts.
- Use test accounts; avoid accessing out-of-scope data.
