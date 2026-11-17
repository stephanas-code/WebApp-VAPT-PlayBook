# Remediation: Security Misconfiguration

This guide provides actionable hardening steps, prioritization, and validation checks to remediate security misconfigurations.

## Principles
- Secure by default; opt-in to risky features
- Least privilege everywhere (users, services, files, buckets)
- Defense in depth and minimize attack surface
- Automate configuration and drift detection

## 1) Security Headers Baseline
Add the following headers on all responses (adjust per app):
- Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
- Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
- X-Content-Type-Options: nosniff
- X-Frame-Options: DENY (or via CSP frame-ancestors)
- Referrer-Policy: no-referrer
- Permissions-Policy: camera=(), microphone=(), geolocation=()

Validate with Mozilla Observatory and securityheaders.com.

## 2) Disable Dangerous HTTP Methods
- Limit to GET, POST, HEAD (others only if strictly required)
- Disable TRACE, TRACK, CONNECT
- Disable WebDAV if not needed; block PUT/DELETE on web roots

Web server snippets:
- Nginx: limit_except GET POST { deny all; }
- Apache: <LimitExcept GET POST> deny from all </LimitExcept>

## 3) Remove Directory Listing and Sensitive Files
- Disable autoindex (Nginx) / Options -Indexes (Apache)
- Block access to .git, .svn, .env, backups, build artifacts
- Store secrets outside web root; never commit secrets

## 4) Harden Authentication and Admin Panels
- Change/remove default credentials immediately
- Enforce MFA for admin interfaces
- Restrict access by IP/VPN; place behind SSO
- Rate limit and lockout policies; CAPTCHA where appropriate
- Ensure secure cookies (Secure, HttpOnly, SameSite=Lax/Strict)

## 5) CORS Configuration
- Avoid wildcard origins; explicitly allow known origins
- Do not use Access-Control-Allow-Credentials with wildcard or broad patterns
- Preflight: validate methods and headers strictly
- Separate public and private APIs by domain

## 6) Error Handling
- Disable detailed error pages in production
- Return generic messages to users, log detailed errors server-side with correlation IDs
- Centralized logging with PII scrubbing; alert on recurring errors

## 7) Dependency and Platform Hygiene
- Maintain SBOM and automatic dependency updates (Dependabot/Renovate)
- Patch OS, runtime, containers regularly
- Remove unused packages, plugins, sample apps
- Pin image tags; use minimal, non-root containers

## 8) Cloud Storage and Metadata
- Set private as default for buckets/containers
- Apply bucket policies blocking public access, enable object ACL restrictions
- Enable server-side encryption and versioning
- Use VPC endpoints; block metadata service v1, require IMDSv2 (AWS)

## 9) TLS/HTTPS
- Enforce HTTPS; redirect HTTP->HTTPS
- Disable TLS1.0/1.1, weak ciphers; prefer TLS1.2+ and modern suites
- Use HSTS (with preload if ready)
- Automate certificate rotation (ACME/Let’s Encrypt)

## 10) Infrastructure and CI/CD
- IaC with code review (Terraform, CloudFormation); policy-as-code checks (OPA/Conftest, Checkov)
- Secrets management (Vault, AWS Secrets Manager, SOPS); no plaintext secrets in repos
- Protect build pipelines; sign artifacts; use provenance (SLSA)

## 11) Logging, Monitoring, and Alerting
- Enable web/access logs and WAF; centralize in SIEM
- Alert on suspicious methods, spikes, 4xx/5xx anomalies
- Store logs immutably with retention and least privilege

## 12) Validation and Verification
- Run regular config audits (Nessus/OpenVAS), header scans, TLS scans
- Pentest/bug bounty for external validation
- Playbooks for misconfiguration response and rollback

## Prioritized Remediation Plan (90-day)
1. Week 1-2: Disable dangerous methods, remove directory listing, protect admin panels
2. Week 3-4: Header baseline + HTTPS/HSTS; rotate default creds, enforce MFA
3. Week 5-6: CORS tightening; error handling; secrets out of repos/webroot
4. Week 7-8: Cloud storage lockdown, IMDSv2; logging/alerts
5. Week 9-12: Dependency updates, container hardening, IaC policy gates

## Acceptance Criteria
- No public buckets/containers; scans show 0 critical findings
- All prod endpoints return required headers; TLS score A/A+
- No dangerous HTTP methods on internet-exposed hosts
- Admin endpoints require MFA and IP/VPN restrictions

## 2026 Modern & AI-Assisted Defensive Updates
- **Cross-Origin Isolation & Modern Browser Hardening Headers**: Supplement baseline headers with modern isolation headers (`Cross-Origin-Opener-Policy: same-origin`, `Cross-Origin-Resource-Policy: same-origin`, `Cross-Origin-Embedder-Policy: require-corp`) where applicable to prevent cross-origin leaks (XS-Leaks).
- **Continuous Cloud Security Posture Management (CSPM) & GitOps Drift Remediation**:
  - Enforce Policy-as-Code gates (OPA/Gatekeeper, Kyverno, Checkov, Trivy) in CI/CD so misconfigured Kubernetes manifests, Terraform plans, or public storage ACLs cannot merge.
  - Use GitOps controllers (ArgoCD, Flux) with automated drift reconciliation so manual out-of-band console changes are automatically reverted and alerted.
- **Supply Chain Attestation & Ephemeral Workload Identities**:
  - Eliminate long-lived cloud keys by enforcing OIDC Workload Identity Federation across CI/CD pipelines and Kubernetes pods, paired with SLSA Build Level 3 provenance and Sigstore Cosign image verification.
- **AI-Assisted Configuration Auditing & Auto-Remediation**:
  - Use AI-assisted IaC reviewers in pull requests to detect subtle cross-resource misconfigurations (e.g., an ingress controller exposing an internal management port or overly permissive IAM role trust policies) and generate least-privilege policy patches automatically.

## References
- OWASP ASVS 4.0.3 / 5.0: V1, V2, V7, V9, V14
- CIS Benchmarks (Web Server, Linux, Docker, Kubernetes)
- NCSC/NSA hardening guides & SLSA v1.0 specification
