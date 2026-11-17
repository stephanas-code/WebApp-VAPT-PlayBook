# Authentication Bypass: Remediation Guide

## Objective
Concrete defensive controls to prevent authentication bypass via brute force, session fixation, and logic flaws. Map fixes to OWASP ASVS and provide implementation guidance with testable acceptance criteria.

## Risk Summary
- Impact: Account takeover, data exfiltration, fraud, regulatory penalties.
- Likelihood: High on internet-facing apps without layered controls.
- Affected areas: Login, session management, MFA, password reset, SSO flows, API auth.

## Design Principles
- Deny-by-default at every auth boundary.
- Defense-in-depth across client, server, and identity provider.
- Fail securely (no partial logins, no privilege on error).
- Deterministic state machine for auth flows.

## Controls Checklist (Engineer-Ready)
1) Brute-force and Credential Stuffing
- Rate limit by account, IP/subnet, device fingerprint, and token bucket per endpoint.
- Progressive delays and lockout with alerting (temporary, user-notified, unlock via out-of-band).
- Require MFA after N failed attempts or high-risk signals.
- Use CAPTCHA only as secondary friction; never as the sole control.
- Monitor and block known bad passwords and breached credentials (HIBP/SCIM, NIST 800-63B).

2) Session Management
- Rotate session ID on login, privilege change, and logout (session fixation prevention).
- Set cookies: Secure, HttpOnly, SameSite=Lax or Strict; short idle + absolute expiry.
- Bind session to user agent + IP reputation (not strict IP pinning), and device key.
- Implement server-side session invalidation on logout and password/MFA changes.
- CSRF-protect all state-changing authenticated endpoints.

3) Logic and Flow Integrity
- Centralize authorization checks; never infer auth from hidden fields/client flags.
- Enforce step ordering with signed, expiring nonces per step (e.g., pre-auth -> MFA -> post-auth).
- Validate redirect/callback targets against allowlist; use state parameter with integrity.
- Prevent forced browsing: protect every endpoint with server-side authz gates.
- Block username enumeration: identical messages and timing for valid/invalid users.

4) MFA and Recovery
- Offer phishing-resistant MFA (FIDO2/WebAuthn) and TOTP; avoid SMS where possible.
- Enforce step-up MFA for sensitive actions and new devices/locations.
- Protect recovery channels: signed links, short TTL, single-use, device/IP risk checks.

5) Password and Account Policy
- NIST 800-63B aligned: allow long passphrases, block common/compromised passwords.
- Store with Argon2id/scrypt/bcrypt, per-credential salt, strong work factor.
- Secure password reset: require proven possession (email or existing MFA) and recent auth.

6) API and Mobile
- Prefer OIDC/OAuth 2.1 with PKCE for public/mobile clients; rotate refresh tokens.
- Use mutual TLS or DPoP for high-risk APIs. Avoid static API keys for user auth.
- Validate JWTs: issuer, audience, exp, nbf, alg=RS256/ES256; pin JWKS kid/keys.

7) Telemetry and Detection
- Centralize auth logs with structured events (success, fail, lockout, reset, MFA, device).
- Alert on anomalies: velocity, impossible travel, spray patterns, bypass attempts.
- Build playbooks for SOC response and user notification.

## Implementation Snippets
- Session cookie
  - Set-Cookie: sid=…; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=1800
- Rotate session on login
  - On successful auth: old_session.invalidate(); new_session = createSession(user)
- Rate limiting (pseudo)
  - key = hash(ip,/login,user?) -> tokenBucket(5/min, burst=10); deny when empty

## Testing and Acceptance Criteria
- Session ID changes on login and privilege elevation; old ID invalid.
- 10 failed logins cause incremental delay and temporary lockout; user notified.
- Password reset link single-use, <= 15 minutes, invalid after use.
- MFA required on new device or risk score > threshold.
- All authenticated POST/PUT/PATCH/DELETE require CSRF token; missing/invalid -> 403.
- Replay of pre-login step tokens is rejected.

## 2026 Modern & AI-Assisted Defensive Updates
- **Passkeys & WebAuthn Level 3 First**: Default to phishing-resistant passkeys (synced and device-bound credentials) and enforce Conditional UI flows to eliminate password-based credential stuffing and phishing vectors.
- **Token Binding & Sender-Constrained Tokens**: Require OAuth 2.1 with DPoP (RFC 9449) or Mutual TLS (mTLS) so stolen bearer tokens cannot be replayed from unauthorized clients.
- **Continuous Access Evaluation Profile (CAEP) & Shared Signals Framework (SSF)**: Continuously evaluate session risk and revoke sessions in real time upon IP/device posture shifts or identity provider revocation events.
- **AI-Assisted Defensive Code Review & State-Machine Verification**:
  - Use AI-assisted static analysis (e.g., Semgrep Assistant, CodeQL + LLM triage) in CI/CD to detect missing authentication decorators/middleware across newly added routes and GraphQL resolvers.
  - Use LLM-assisted property-based test generation to automatically synthesize negative unit tests for multi-step authentication state machines (verifying that step $N$ rejects requests lacking cryptographically signed proof of step $N-1$).
  - Deploy behavioral telemetry models at the identity edge to score anomalous login velocity and residential proxy rotation without relying solely on static IP blocklists.

## References
- OWASP ASVS v4.0.3 / v5.0: V2 (Authentication), V3 (Session Management), V7 (Error Handling and Logging)
- OWASP Cheat Sheets: Authentication, Session Management, Forgot Password, Credential Stuffing
- NIST SP 800-63B-4 Digital Identity Guidelines & RFC 9449 (OAuth 2.0 DPoP)
