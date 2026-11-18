# Authentication Bypass: Burp Suite Hands-on Guide

This guide provides step-by-step Burp workflows to discover and exploit authentication bypass scenarios, aligned with the module structure used in other folders.

## Lab Setup
- Tools: Burp Suite (Community/Pro), browser with Burp CA installed, wordlists, test mail server.
- Burp configuration: Intercept on, Proxy set, Logger/Extender optional (e.g., Autorize, Turbo Intruder, JSON Web Tokens, JWT Editor, Param Miner).

## Recon and Baseline
- Map auth surface with Proxy/Target/Sitemap: /login, /api/login, /session, /mfa, /password-reset, /oauth, /oidc.
- Identify cookies, CSRF tokens, headers, and any device or risk signals.
- Send key requests to Repeater for controlled testing; save a clean baseline.

## Brute Force and Credential Stuffing
- In Repeater, observe consistent responses for invalid creds; in Comparer, verify bodies/statuses are indistinguishable.
- Use Intruder (or Turbo Intruder):
  - Positions: set on username and password.
  - Payloads: small curated list; avoid production-scale spray in testing.
  - Throttling: 1-3 rps; add Grep - Match to detect success markers (e.g., redirect, JWT present).
- Test protections: lockout timing, CAPTCHA requirement, MFA challenges, CSRF enforcement.

## Session Fixation and Cookie Issues
- Capture pre-auth cookie; after login confirm rotation in Repeater requests.
- Attempt fixation: set Cookie: session=attacker_id before login; verify if post-auth sticks.
- Check flags via Proxy history and Response headers: Secure, HttpOnly, SameSite, Path, Domain.
- Logout invalidation: in Repeater, call logout then retry an authenticated endpoint to ensure 401/403.

## Logic Flaws and Flow Bypass
- Track state across steps: parameters like step, nonce, state, flowId.
- Replay or skip steps with Repeater by forging direct POSTs to MFA or final endpoints.
- Use Autorize extension to detect missing server-side checks when toggling roles/headers.
- Tamper redirect/callback parameters and observe open redirect or consent skipping.

## Password Reset Workflow Tests
- Intercept reset request, note token delivery mechanism.
- Verify single-use and TTL by reusing the link and delaying tests.
- Attempt host header manipulation and check link generation.
- Ensure enumeration resistance: compare responses for unknown vs known emails with Comparer.

## OAuth/OIDC Tests
- Proxy through the entire authorize -> token -> callback flow.
- In Repeater:
  - Remove/alter state and nonce; expect 400/403.
  - With PKCE, send wrong code_verifier; token should be denied.
  - Validate ID token in JWT Editor: iss/aud/sub/exp/nbf, signature, alg not none.
- Inspect JWKS cache and kid rotation behavior (Pro tip: record 304 and caching headers).

## API & Modern Authentication Workflows
- **OAuth 2.1 & DPoP (Demonstrating Proof-of-Possession)**:
  - In Repeater, inspect for `DPoP` request headers containing signed JWT proofs.
  - Test replay: attempt replaying the bearer token without the corresponding private key signature or with an expired DPoP proof.
- **Passkeys & WebAuthn Interception**:
  - In Burp's embedded Chromium browser, use DevTools (`F12`) → **More tools** → **WebAuthn** to inject virtual authenticators.
  - Capture the `/webauthn/register` and `/webauthn/authenticate` JSON payloads in Proxy.
  - Test clientDataJSON challenge tampering and origin mismatch checks.
- **JWT Tampering (via JWT Editor Extension)**:
  - Open target request in Repeater → navigate to the **JSON Web Token** sub-tab.
  - Test signature exclusion: select `alg: none` and remove the signature component.
  - Test key confusion (RS256 to HS256): re-sign using the server's public key as the HMAC shared secret.
  - Inject attacker JWK parameters (`jwk` header injection) to verify whether the backend trusts untrusted keys.

## 2026 AI-Assisted Burp Automation
- **Turbo Intruder Scripting**: Automate multi-step authentication state fuzzing using asynchronous Python scripts within Turbo Intruder.
- **AI Triage & Synthesis**: Export Repeater request/response pairs directly (right-click → *Copy as curl*) into AI code review prompts to automatically synthesize reproducible test scripts and remediation pull requests.

## Reporting Evidence
- Save Repeater tabs and Proxy histories; annotate with comments.
- Include screenshots of response diffs (Comparer) and cookie rotations.
- Map findings to OWASP ASVS sections and include remediation references.

## Safety and Cleanup
- Respect rate limits; avoid account lockouts unless approved.
- Revoke tokens, log out, remove test MFA devices; provide artifacts to engineering.
