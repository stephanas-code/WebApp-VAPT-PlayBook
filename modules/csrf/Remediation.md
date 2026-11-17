# CSRF Remediation

This document outlines effective strategies to prevent Cross-Site Request Forgery in web applications, with concrete implementation guidance and verification steps.

## Principles
- Require a per-request, unpredictable anti-CSRF token for all state-changing actions
- Bind tokens to the user session and intended action/context
- Enforce SameSite cookie policies appropriately
- Validate Origin and/or Referer on state-changing requests
- Use proper HTTP methods (avoid state changes via GET)
- Harden session and CORS configurations

## Implementation Patterns

### 1) Synchronizer Token Pattern (STP)
- Generate a cryptographically secure token per user session
- Embed token in forms or custom header for AJAX requests
- Validate token server-side on every state-changing request

Example (server-side pseudocode):
```python
# on login / session creation
def issue_token(session):
    session['csrf'] = random_32_bytes()

# on GET form render
def render_form(session):
    token = session['csrf']
    return f'<input type="hidden" name="csrf_token" value="{token}">'

# on POST handler
def handle_post(request, session):
    assert constant_time_equals(request.form['csrf_token'], session['csrf'])
```

### 2) Double Submit Cookie
- Set a CSRF cookie with a random value on first visit
- Require the same value in a request parameter or header
- Compare cookie and parameter values server-side
- Scope cookie to the site: `Secure; HttpOnly; SameSite=Lax/Strict` as appropriate

Example (Express.js):
```javascript
app.post('/profile', (req, res) => {
  const cookie = req.cookies['csrf_token']
  const param  = req.body.csrf_token || req.get('X-CSRF-Token')
  if (!cookie || !param || !tsecEqual(cookie, param)) return res.status(403).end()
  // proceed
})
```

### 3) Framework built-ins
- Use and configure built-in CSRF middleware correctly
  - Django: {% csrf_token %}, CsrfViewMiddleware; set CSRF_COOKIE_HTTPONLY=false if reading from DOM, else use header
  - Spring Security: CsrfFilter; ensure all state-changing endpoints are covered
  - Rails: protect_from_forgery with: :exception
  - Express: csurf middleware across all routers

## Cookie Strategy

- Set session cookies with:
  - HttpOnly; Secure; SameSite=Lax (or Strict if UX allows)
- Avoid SameSite=None unless you have a strong cross-site need and always pair with Secure
- For sensitive actions that must allow cross-site POST (e.g., payment), implement explicit user interaction and robust token validation

## Header Validation

- Validate Origin and/or Referer for POST/PUT/PATCH/DELETE
  - Accept only exact origins that match your application
  - Reject if header is absent (with narrow exceptions like legacy clients)
- Log and alert on mismatches to detect attacks

Example (Java/Spring):
```java
String origin = request.getHeader("Origin");
String referer = request.getHeader("Referer");
if (!allowed(origin, referer)) return 403;
```

## Method and Route Hygiene

- Disallow state changes via GET
- Use idempotent methods appropriately
- Ensure JSON APIs also enforce CSRF protections (not just HTML forms)
- Document and test exceptions (e.g., webhooks) with signatures instead of CSRF tokens

## CORS Configuration

- Do not rely on CORS to mitigate CSRF
- For cross-site needs, restrict CORS:
  - Access-Control-Allow-Origin: explicit domains only
  - Access-Control-Allow-Credentials: true only when needed
  - Validate preflight and apply CSRF protections on actual requests

## UI and Clickjacking

- Combine CSRF defenses with clickjacking mitigation:
  - X-Frame-Options: DENY or SAMEORIGIN
  - Content-Security-Policy: frame-ancestors 'none' or allowed list

## Verification Checklist

- [ ] All state-changing routes require CSRF tokens
- [ ] Tokens are random, per-session (or per-request), and validated server-side
- [ ] Tokens are bound to user and context (action, nonce)
- [ ] Origin/Referer enforced for unsafe methods
- [ ] Session cookies set with Secure, HttpOnly, SameSite=Lax/Strict
- [ ] No state-changing GET endpoints
- [ ] JSON/AJAX endpoints include CSRF header/token
- [ ] CORS not used as a CSRF control; explicit origins only
- [ ] Double-submit cookie (if used) is validated correctly and not attacker-controlled
- [ ] Clickjacking protections enabled
- [ ] Automated tests cover CSRF checks

## Developer Tips

- Centralize CSRF middleware to minimize gaps
- Provide helpers to inject tokens into all forms and AJAX requests
- Add logging around CSRF failures for incident response
- Educate teams on SameSite impacts and user flows

## 2026 Modern & AI-Assisted Defensive Updates
- **Fetch Metadata Request Headers (`Sec-Fetch-*`)**: Enforce browser-native Resource Isolation Policies by inspecting `Sec-Fetch-Site`, `Sec-Fetch-Mode`, and `Sec-Fetch-Dest` middleware-wide. Reject state-changing requests where `Sec-Fetch-Site` is `cross-site` unless explicitly allowlisted:
  ```javascript
  function fetchMetadataCsrfGuard(req, res, next) {
    if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) return next();
    const site = req.get('Sec-Fetch-Site');
    if (site && !['same-origin', 'same-site', 'none'].includes(site)) {
      return res.status(403).json({ error: 'Cross-site request blocked by Fetch Metadata policy' });
    }
    next();
  }
  ```
- **Host-Prefixed Cookies (`__Host-`)**: Bind anti-CSRF and session cookies using the `__Host-` prefix (`Set-Cookie: __Host-csrf=...; Secure; Path=/; HttpOnly; SameSite=Lax`) to prevent cookie tossing from compromised sibling subdomains.
- **AI-Assisted Route Auditing & Automated Middleware Verification**:
  - Use AI-assisted SAST rules in CI/CD to scan route definitions and RPC/Server Action endpoints (e.g., Next.js Server Actions, Remix actions, GraphQL mutations) to verify that every state-changing handler enforces `Origin` validation, `Sec-Fetch-Site` checks, and cryptographic anti-CSRF tokens.
  - Automatically generate integration test matrices in CI that replay state-changing API routes with mismatched `Origin` headers, missing CSRF headers, and `Content-Type: text/plain` simple-request Downgrades to verify `403 Forbidden` enforcement.

## References
- OWASP CSRF Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- W3C Fetch Metadata Request Headers & Resource Isolation Policies: https://web.dev/articles/fetch-metadata
- PortSwigger Web Security Academy - CSRF: https://portswigger.net/web-security/csrf
- MDN Set-Cookie SameSite: https://developer.mozilla.org/docs/Web/HTTP/Headers/Set-Cookie/SameSite
