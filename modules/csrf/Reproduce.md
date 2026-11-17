# Reproduce CSRF Vulnerabilities

This guide provides step-by-step procedures to reliably reproduce CSRF issues across typical web application flows. Use a separate attacker domain for PoC hosting when possible.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Identify Targets](#identify-targets)
- [Capture Baseline Requests](#capture-baseline-requests)
- [Token Validation Tests](#token-validation-tests)
- [Header Validation Tests](#header-validation-tests)
- [Method/Content-Type Tests](#methodcontent-type-tests)
- [PoC Generation and Validation](#poc-generation-and-validation)
- [Edge Cases](#edge-cases)
- [Reporting Template](#reporting-template)

## Prerequisites
- Burp Suite running with browser proxy configured
- Logged-in victim session in a proxied browser profile
- Ability to host an HTML file (attacker page) or use Burp's CSRF PoC generator

## Identify Targets
1. Browse the application and list state-changing actions:
   - Profile updates, email/phone change
   - Password reset/change
   - Add/remove payment methods
   - Create/modify/delete resources
   - Transactions, transfers
2. Prioritize high-impact actions.

## Capture Baseline Requests
1. Perform each action once legitimately.
2. In Burp Proxy > HTTP history, locate the request.
3. Send to Repeater. Note:
   - URL, method
   - All parameters (query, body, JSON)
   - Cookies and headers
   - CSRF token locations (form field, header, cookie)

## Token Validation Tests
Run these tests in Repeater and record outcomes:

- Remove token parameter entirely
- Empty token value
- Random token value
- Reuse the same token multiple times
- Token from a prior session
- Token from another user/session
- Token in wrong location (header vs parameter)

Expected secure behavior: All above should fail with 403 or error, without processing state change.

## Header Validation Tests
- Remove Referer header; send request
- Change Origin to a foreign site
- Use subdomain (evil.target.com) vs main domain
- Use open redirect on target to influence Referer

Expected secure behavior: Requests with missing or foreign Origin/Referer are rejected.

## Method/Content-Type Tests
- Change POST to GET using Repeater
- Change Content-Type between application/x-www-form-urlencoded, multipart/form-data, text/plain
- Try JSON endpoints with text/plain or multipart to bypass JSON-only checks

Expected secure behavior: Server requires correct method and content type; rejects others.

## PoC Generation and Validation
1. Use Repeater: Right-click the request > Engagement tools > Generate CSRF PoC.
2. Save the HTML PoC.
3. Open PoC in a separate browser profile logged in as victim.
4. Validate: Did the action execute? Capture evidence (before/after screenshots, HTTP traces).

### Manual PoC Template
```html
<form action="https://target.com/path" method="POST">
  <input type="hidden" name="param1" value="value1">
  <input type="submit" value="Submit">
</form>
<script>document.forms[0].submit()</script>
```

## Edge Cases
- SameSite=Lax cookies allow top-level GET navigation; test GET endpoints
- Double-submit cookie: ensure cookie and parameter alignment is required
- CORS misconfiguration: JSON endpoints processing cross-site without preflight
- SPA/CSR apps: token generated dynamically in DOM; verify token refresh and scope
- Mobile/desktop API differences: test mobile endpoints as well

## Reporting Template

Title: CSRF on [Function/Endpoint]

Summary:
- State-changing action lacks effective CSRF protection, enabling cross-site requests to succeed.

Impact:
- Describe what an attacker can do and business impact (e.g., account takeover, financial loss).

Steps to Reproduce:
1. Log in as victim and stay authenticated.
2. Host the following PoC at attacker.com/poc.html
3. Visit the PoC while logged in.
4. Observe that the action executes.

PoC:
```html
<form action="https://target.com/..." method="POST">
  <input type="hidden" name="..." value="...">
</form>
<script>document.forms[0].submit()</script>
```

Evidence:
- HTTP request/response demonstrating successful state change
- Before/after screenshots

Recommendations:
- Implement CSRF tokens, enforce Origin/Referer, set SameSite cookies
- See Remediation.md for full guidance

References:
- PortSwigger Web Security Academy - CSRF: https://portswigger.net/web-security/csrf
- OWASP CSRF Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
