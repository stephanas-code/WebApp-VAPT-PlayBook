# Sensitive Data Exposure - Burp Suite Guide

Purpose: Provide a repeatable Burp workflow to detect and evidence sensitive data exposure, cryptographic failures, weak secrets, and insecure transmission/storage.

---

## 1) Project Setup
- Create a new Burp project with a dedicated target scope (Target > Scope).
- Install Burp CA in the browser and enable invisible proxying if testing mobile/API clients.
- Configure Proxy listeners; enable HTTP/2 and ALPN support as needed.

Checklist:
- [ ] Scope restricted to target domains
- [ ] Interception on, CA installed
- [ ] Upstream proxy/VPN configured if required

---

## 2) Target and Site Map Hygiene
- Add target hosts to Scope to filter noise.
- Use Site map to identify sensitive endpoints: /login, /account, /api/*, /checkout, /reset, /admin.
- Right-click nodes to include/exclude; apply labels for “Sensitive.”

---

## 3) HTTPS, HSTS, and Mixed Content
- Use Proxy history filter: show only HTTP or only HTTPS.
- Identify any HTTP requests to sensitive paths; verify redirects to HTTPS.
- In response headers, confirm Strict-Transport-Security and Referrer-Policy.
- Use Issue activity (Scanner/Advisor) for “Mixed content” or “Insecure transport” findings.

Evidence: Screenshots of HTTP 200 on sensitive path, missing HSTS, or mixed content.

---

## 4) Cookie and Session Analysis
- In Proxy history, highlight Set-Cookie headers; check for Secure, HttpOnly, SameSite.
- Compare cookie presence over HTTP vs HTTPS requests.
- Use Decoder to inspect URL/Base64 encoded cookie values; identify JWTs and analyze claims in JWT Editor (Burp extension).
- Use Sequencer to assess token entropy: capture session tokens and analyze randomness.

Evidence: Set-Cookie without Secure/HttpOnly, low-entropy results, predictable tokens.

---

## 5) Response Minimization and Excessive Data
- Use Repeater to replay GET/POST to /api/user and enumerate fields.
- Switch Accept headers (application/json, application/xml, text/html) to detect variant outputs.
- Compare authenticated vs unauthenticated outputs to detect over-sharing.
- Record any password_hash, tokens, SSN, PAN, internal_notes fields.

---

## 6) Error Handling and Verbose Messages
- Send edge-case inputs via Repeater: negative IDs, very large numbers, special chars, malformed JSON.
- Observe stack traces, file paths, SQL errors, debug headers (X-Debug, Server, X-Powered-By).
- Use Intruder to fuzz parameters for error-based disclosures.

---

## 7) URL and Referer Leakage
- Track flows that put sensitive data into URLs (reset tokens, OAuth state/code, email, PII).
- In Proxy history, sort by URL and search for key terms: token, key, secret, passwd, ssn.
- From a sensitive URL, navigate to an external link; verify Referer header contains sensitive query.

---

## 8) Client-Side Storage and Caching
- Use Burp’s Browser to access DevTools; check Application storage for localStorage/sessionStorage contents.
- Inspect Cache-Control and Pragma headers for sensitive pages.
- After logout, use the Back button to verify cache replay; confirm whether server-side revalidation occurs.

---

## 9) Static Asset and Secret Discovery
- Use the Content Discovery (or add-on like “Param Miner”, “Asset Discoverer”) to enumerate hidden paths.
- Fetch JS/CSS and search within Burp (Find in responses) for patterns: key|secret|token|api|passwd.
- Check for backup/config artifacts: /.env, .git/, *.bak, backup.zip, .DS_Store.

---

## 10) TLS and Cipher Checks (adjacent tools)
- While Burp records traffic, run testssl.sh/sslscan externally; attach results to Burp issue notes.
- Confirm TLS minimum version and HSTS preload eligibility.

---

## 11) Automation with Burp Scanner (Pro)
- Run an active scan on authenticated areas with careful throttling.
- Enable checks for Information disclosure, Sensitive data in URLs, Insecure cookies, Mixed content.
- Review advisory details; validate manually in Repeater.

---

## 12) Reporting Evidence
- For each finding, add to Burp’s issue tracker or use Extensions (Logger++, CSA Reporter):
  - Title, severity, affected endpoints
  - Steps to reproduce (link to Repeater tabs)
  - Request/response pairs with sensitive elements highlighted
  - Business impact and recommended remediation

---

## Useful Extensions
- JWT Editor, Logger++, Param Miner, Hackvertor, Additional Scanner Checks, Collaborator Everywhere.

---

## Quick Filters and Snippets
- Proxy history search terms: password, passwd, token, key, secret, ssn, pan, credit, Authorization.
- Repeater header toggles:
  - Accept: application/xml
  - Accept: text/html
  - Cache-Control: no-cache
  - Pragma: no-cache
- Cookie flags checklist: Secure, HttpOnly, SameSite

---

Outcome: A consistent Burp-driven methodology to detect sensitive data exposure and cryptographic failures with reproducible evidence for remediation.
