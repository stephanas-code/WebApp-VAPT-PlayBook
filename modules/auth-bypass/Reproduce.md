# Authentication Bypass: Reproduce Guide

This guide provides step-by-step procedures to reproduce common authentication bypass scenarios in a safe test environment. Use only on systems you are authorized to test.

## Scope and Pre-reqs
- Target: login, session, MFA, password reset, SSO (OIDC/OAuth), and key authenticated APIs.
- Test accounts: at least one valid user, and consent for lockout/MFA testing.
- Tools: Browser with devtools, Burp Suite (Community or Pro), wordlists, and test SMTP.

## Scenario 1: Brute Force / Credential Stuffing
1) Observe login request and identify parameters: username/email, password, CSRF token, cookies.
2) Attempt invalid logins with controlled rate (e.g., 1 req/sec) and monitor responses.
3) Validate protections:
   - Rate-limit/lockout triggers after N attempts per account/IP/device.
   - Response body/status are identical for wrong-username vs wrong-password.
   - CSRF token validated on POST; missing/invalid yields 403.
4) Credential stuffing test: replay pairs from small synthetic set; verify detection/alerts.
5) Edge cases: parallel attempts, multiple endpoints (/login, /api/login), password reset throttling.

Expected: Incremental delay or block; account temporarily locked; no enumeration; alert raised.

## Scenario 2: Session Fixation
1) Obtain a pre-auth session cookie by visiting login page.
2) Log in normally and check if session cookie value rotates. Capture before/after.
3) Try setting a fixed cookie value before login via devtools or Burp.
4) If cookie is not rotated, confirm that post-login retains attacker-chosen ID (vulnerable).

Expected: New session ID after login; old session invalidated.

## Scenario 3: Logic Flaws in Multi-step Auth (MFA)
1) Walk through login -> MFA flow capturing each request, nonce/state, and cookies.
2) Attempt to skip steps (direct POST to /mfa/verify without prior step).
3) Replay earlier step tokens/nonces or modify step= parameter.
4) Change redirect/callback target parameters; check for open redirect then consent bypass.

Expected: Server rejects out-of-order, replayed, or tampered requests with 400/403 and fresh nonces.

## Scenario 4: Password Reset Abuse
1) Trigger password reset for a known email; capture tokenized link.
2) Test TTL: use link after expiry; verify invalidation.
3) Test single-use: reuse the same link; expect failure.
4) Test enumeration: compare timing/messages for existing vs non-existing email.
5) Attempt host header injection to redirect reset link to attacker domain (should be blocked).

Expected: No enumeration; token short-lived and single-use; strict host validation.

## Scenario 5: Forced Browsing / Direct Object Access
1) Access authenticated endpoints without login (GET/POST /account, /orders, /api/me).
2) Modify role/flags in client requests and hidden fields; verify server-side checks.
3) Change path/query to admin-only routes; ensure 401/403 without proper authZ.

Expected: Uniform 401/403; no leakage of privileged data/actions.

## Scenario 6: OAuth/OIDC Bypass Attempts
1) Initiate OIDC login; capture authorize request with state and nonce.
2) Attempt to remove/alter state/nonce; check if server enforces integrity.
3) Test code flow with PKCE: change code_verifier; verify token exchange fails.
4) Validate ID token audience/issuer and signature; attempt alg=none downgrade (should fail).

Expected: Strict state/nonce checks; PKCE required; tokens validated; JWKS pinned.

## Evidence and Logging
- Capture request/response pairs, timestamps, and server log entries for each scenario.
- Document protection behavior and any deviations from expected controls.
- Provide reproducible PoC steps with sanitized payloads and screenshots where allowed.

## Cleanup
- Unlock any test accounts; invalidate sessions; remove test MFA devices; purge tokens.
