# Sensitive Data Exposure - Remediation Guide

## Objective
Eliminate plaintext leaks, insecure transmission/storage, weak secrets, and cryptographic failures. Ensure confidentiality, integrity, and compliance for sensitive data in transit, at rest, and in use.

---

## 1) Data Classification and Minimization
- Inventory and classify data: PII, PCI, PHI, secrets, keys, tokens.
- Collect only necessary data; remove unused fields and legacy PII.
- Apply least privilege and purpose limitation; segregate by sensitivity.
- Implement data retention/TTL with automatic purge and defensible deletion.

KPIs: Data inventory coverage, reduction in PII fields, retention policy compliance rate.

---

## 2) Transport Layer Security (In-Transit)
- Enforce HTTPS everywhere (HSTS with preload):
  - HTTP->HTTPS 301 redirects; block cleartext endpoints.
  - Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
- TLS configuration:
  - TLS 1.2+ (prefer TLS 1.3), disable SSLv2/3, TLS 1.0/1.1.
  - Strong ciphers (AES-GCM/CHACHA20-POLY1305), ECDHE forward secrecy.
  - OCSP stapling, secure renegotiation; disable compression.
- Cookie security:
  - Set-Cookie: Secure; HttpOnly; SameSite=Strict/Lax as appropriate.
  - Split tokens: non-sensitive id in cookie, secret in HttpOnly cookie.
- Referrer policy and mixed content:
  - Referrer-Policy: no-referrer or strict-origin-when-cross-origin.
  - Block mixed content; upgrade-insecure-requests CSP.

Validation: testssl.sh, SSL Labs A+, verify HSTS preload.

---

## 3) Secure Storage (At Rest)
- Encrypt sensitive data with vetted algorithms:
  - AES-256-GCM or XChaCha20-Poly1305 for data; RSA/ECDH only for key exchange.
- Key management:
  - Use KMS/HSM (AWS KMS, GCP KMS, Azure Key Vault); never hardcode keys.
  - Rotate keys regularly; distinct keys per tenant/data domain; key derivation via HKDF.
  - Enforce envelope encryption; restrict KMS IAM with least privilege.
- Password storage:
  - Argon2id (preferred) or scrypt/bcrypt with per-user unique salt, tuned cost.
  - Use pepper stored in HSM/KMS; no plaintext/reversible encryption.
- Database hardening:
  - Transparent encryption (TDE) plus application-level encryption for highly sensitive fields.
  - Disable verbose logs; scrub PII before logging; enable audit trails.

Validation: cryptographic architecture review, key rotation runbooks, redaction in logs.

---

## 4) Secrets Management
- Centralize secrets in a vault (HashiCorp Vault, AWS Secrets Manager, Doppler):
  - Dynamic credentials, short TTL, automatic rotation.
  - Access via IAM roles/workload identity, not static credentials.
- Remove secrets from code, images, build logs, and client-side JS.
- Implement secret scanning in CI (gitleaks, TruffleHog, GitHub secret scanning).
- Employ environment-specific scoping and deny-all default policies.

Validation: periodic vault access reviews, CI scan reports, zero hardcoded findings.

---

## 5) API and Response Hardening
- Data minimization in responses; design DTOs to expose only required fields.
- Implement allowlists/field-level filtering and output encoding.
- Disable debug/error detail in production; map internal errors to generic messages.
- Pagination and filtering to avoid bulk data exposure.
- Content negotiation: provide consistent minimal schemas across formats (JSON/XML/CSV).

Validation: contract tests for response shape; snapshot tests for sensitive fields.

---

## 6) Client-Side Protections
- Never store secrets, tokens, or PII in localStorage/sessionStorage/IndexedDB.
- Use HttpOnly cookies for session/auth tokens; set short lifetimes and rotation.
- Prevent caching of sensitive pages:
  - Cache-Control: no-store, no-cache, must-revalidate; Pragma: no-cache.
- Disable autocomplete for sensitive inputs and use masking for PAN/SSN.
- Implement CSP, X-Content-Type-Options, X-Frame-Options/Frame-Ancestors, Permissions-Policy.

Validation: DevTools Application/Network audit; Lighthouse security checks.

---

## 7) Logging, Monitoring, and Incident Response
- Pseudonymize/anonymize logs; never log secrets, tokens, or PAN/PII.
- Implement centralized logging with redaction pipelines; protect logs with ACLs and encryption.
- Monitor for anomalous data egress and secret use; alert on policy violations.
- Establish incident runbooks: key revocation, token invalidation, customer notification.

Validation: red-team drills; log sampling shows no sensitive values.

---

## 8) Access Control and Data Governance
- Enforce RBAC/ABAC for data access; separate administrative and customer data planes.
- Apply row/column-level security; implement tenant isolation controls.
- Use mTLS or signed requests for service-to-service communication.
- Data sharing and exports require explicit approvals and masking.

Validation: access reviews, policy-as-code tests (OPA), data exfiltration guardrails.

---

## 9) Build/Deploy and Supply Chain
- Shift-left: SAST/DAST/IAST and secret scanning in CI; block on high severity leaks.
- SBOM and dependency scanning; pin versions; verify signatures; minimize transitive exposure.
- Secure build artifacts; no secrets in container layers; use distroless images.
- Pre-prod gates: security checks for headers, TLS, and response schemas.

Validation: CI quality gates; reproducible builds; environment drift detection.

---

## 10) Compliance and Privacy
- Align with GDPR, PCI DSS, HIPAA, SOC 2; perform DPIAs where required.
- Implement DSAR processes; encryption keys must support selective erasure.
- Tokenize or format-preserving encrypt regulated data (PCI PAN, PHI identifiers).

Validation: audit evidence, policy mappings, quarterly control reviews.

---

## Code Examples

### Express.js (Node) Security Headers
```js
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('Permissions-Policy', 'geolocation=(), microphone=()');
  res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate');
  res.setHeader('Pragma', 'no-cache');
  next();
});
```

### Password Hashing (Argon2id)
```python
import argon2
ph = argon2.PasswordHasher(time_cost=3, memory_cost=65536, parallelism=2)
hashed = ph.hash('user-password')
ph.verify(hashed, 'user-password')
```

### AES-GCM Encryption (Go)
```go
ciphertext, err := sealWithAESGCM(plaintext, key, aad)
if err != nil { return err }
```

---

## Verification Checklist
- TLS-only, HSTS preloaded, no mixed content
- Cookies: Secure, HttpOnly, appropriate SameSite
- No sensitive data in URLs, headers, or client storage
- Responses expose only necessary fields; errors generic
- Secrets centralized with rotation and IAM controls
- Sensitive data encrypted at rest; passwords via Argon2id/bcrypt/scrypt
- Logs redacted; cache disabled on sensitive pages
- Access controls enforced; tenant isolation validated

---

## 11) 2026 Modern & AI-Assisted Defensive Updates
- **Post-Quantum Cryptography (PQC) Hybrid Readiness**: Enable hybrid post-quantum key exchange (`X25519MLKEM768` / NIST FIPS 203 ML-KEM) on TLS 1.3 ingress terminators and internal service meshes to protect sensitive data-in-transit against "harvest now, decrypt later" threats.
- **Confidential Computing & Field-Level Envelope Encryption**: Use KMS-backed envelope encryption with Authenticated Encryption with Associated Data (AEAD, binding `tenant_id` and `record_id` as AAD) so stolen database rows cannot be transplanted across records.
- **AI/LLM Data Loss Prevention (DLP) & Redaction Guardrails**:
  - Scrub PII, PHI, payment tokens, and API secrets via deterministic + NER-based redaction pipelines (e.g., Microsoft Presidio, OpenTelemetry collector processors) before data enters logs, telemetry lakes, or AI/RAG vector databases.
  - Enforce automated pre-commit and CI push protection with real-time partner revocation (GitHub Push Protection, TruffleHog/Gitleaks live verification) and AI-assisted DTO schema auditing to prevent accidental over-serialization of sensitive database columns (OWASP API3: BOPLA).

---

References: OWASP ASVS V2, V3, V6, V7, V8 (v4.0.3 / v5.0); OWASP Top 10 A02: Cryptographic Failures; NIST FIPS 203 (ML-KEM).
