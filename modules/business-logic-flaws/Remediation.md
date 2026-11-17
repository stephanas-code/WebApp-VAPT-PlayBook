# Remediation Guide: Business Logic Flaws

This document describes engineering controls, process defenses, and monitoring to prevent and detect business logic vulnerabilities.

## Principles
- Server is the source of truth for all business rules and totals
- Explicitly model workflow states and legal transitions
- Deny-by-default on missing or inconsistent state
- Idempotency and atomicity for state-changing operations
- Consistent authorization: object ownership and role checks on server

## Design and Requirements
- Threat model business processes; document abuse cases per feature
- Define invariants (must always hold) and guard them centrally
- Define rate/quantity/financial limits with business owners
- Specify fraud and abuse detection requirements (signals, thresholds)

## Implementation Controls

### 1) Server-side Validation of Business Rules
- Recalculate price, tax, discount on server using trusted catalog
- Ignore client-submitted monetary fields except identifiers
- Validate coupon scope, single-apply, and exclusivity sets
- Enforce min/max qty, stock, and per-user purchase limits
- Validate cross-entity relationships (tenant, owner, org)

### 2) Workflow State Machine
- Represent workflows via finite state machine (FSM)
- Validate allowed transitions: e.g., payment_captured before order_status=confirmed
- Require mandatory artifacts per state (KYC docs, 2FA, TOS consent)
- Use unguessable, single-use step tokens bound to session and state

### 3) Authorization and Ownership
- Enforce object-level checks based on server-derived subject (user/tenant)
- Never accept tenant_id/user_id from client to decide access
- Use row-level security or scoped queries for multi-tenant isolation

### 4) Limits and Anti-automation
- Per-identity rate limits and anomaly detection (IP, device, payment instrument)
- Quotas for drafts, carts, orders, coupons
- Exponential backoff and lockouts with safe UX
- Bot/fraud protections: proof-of-work or CAPTCHA as last resort

### 5) Concurrency and Idempotency
- Require Idempotency-Key for POST/PUT/PATCH of payments, refunds, approvals
- Atomic operations using transactions and unique constraints
- Optimistic concurrency control (version field) for updates
- Distributed locks for scarce resources (inventory, coupons)

### 6) Monetary Safety Rails
- Clamp totals to >= 0; refuse negative payouts unless explicit refund flow
- Round consistently (banker’s rounding) at correct precision
- Prevent float errors with fixed-point decimals
- Ledger with double-entry accounting for financial movements

### 7) Observability and Audit
- Structured logs with correlation/idempotency keys and state transitions
- Immutable audit trail for financial/privileged actions
- Dashboards and alerts for anomalies: negative totals, coupon double-apply, cross-tenant access denials

### 8) Secure Defaults and Defense in Depth
- Feature flag risky flows; soft-launch with monitoring
- Kill switches for coupons/referrals/payments
- Server-side session binding to device/risk score

## Code Patterns

- Central Pricing Service
  - Input: sku ids, qty, user_id, coupon codes
  - Output: authoritative totals; reject on rule violation
- Transition Guard Middleware
  - Assert legal transition(current_state, action, context)
- Policy Enforcement
  - hasOwnership(subject, object) + hasPermission(role, action)

## Testing and QA
- Abuse-case test suites per feature
- Property-based tests for pricing and limits
- Fuzz boundary values for qty, time windows, currency
- Parallel tests for race conditions (Turbo Intruder scripts)

## Deployment and Ops
- Rate limit configs as code; fast rollback
- Canary releases for logic changes; verify KPIs
- Runbooks for fraud/abuse spikes; pause risky features quickly

## Remediation Workflow
1. Identify affected flows and quantify business impact
2. Add server-side guards and monitoring first
3. Patch state machine and authorization gaps
4. Add concurrency/idempotency controls
5. Backfill audit/ledger for inconsistencies
6. Add regression tests; deploy behind flag; monitor

## 2026 Modern & AI-Assisted Defensive Updates
- **Declarative Policy-as-Code & Formal State Invariants**: Externalize complex business rules and transition guards into verifiable policy engines (e.g., Open Policy Agent / Rego, AWS Cedar) and Temporal/durable execution workflows so state transitions are deterministic and auditable.
- **Database-Enforced Concurrency & Ledger Integrity**: Use PostgreSQL `SERIALIZABLE` / `SELECT ... FOR UPDATE SKIP LOCKED` or dedicated double-entry ledger stores with non-negative check constraints (`CHECK (balance >= 0)`) to prevent race-condition over-redemption at the storage layer.
- **AI-Assisted Threat Modeling & Invariant Synthesis**:
  - Integrate AI-assisted architecture review into pull requests to infer implicit business invariants (e.g., "refund total must never exceed captured payment total across split transactions") and generate property-based tests (Hypothesis, fast-check).
  - Use LLM-guided specification checkers to compare API handler implementations against product requirement documents (PRDs) and OpenAPI specs, flagging missing state transition checks before deployment.
  - Deploy real-time anomaly detection on business KPIs (coupon redemption velocity, negative margin checkout alerts, abnormal state transition sequences) with automated circuit-breaker feature flags.

## References
- OWASP BLT (Business Logic Testing) & OWASP Top 10 API8 / API6 (Unrestricted Access to Sensitive Business Flows)
- CWE-840 Business Logic Errors
- Stripe Idempotency patterns, Martin Fowler on sagas/transactions
