# Reproduce: Business Logic Flaws

This guide walks through step-by-step procedures to discover and validate common business logic weaknesses. Perform in a test or sandbox environment.

## Preparation
- Identify target flows (checkout, refunds, KYC, subscription, approvals)
- Create accounts with distinct roles/tenants
- Capture a clean baseline: HAR, Burp project, screenshots, expected totals

## 1) Map Workflow and State
1. Record each step, required artifacts, and tokens
2. Note state parameters: step, status, approval, payment_captured, kyc_status
3. Identify server-only calculations vs client-computed fields

Deliverable: diagram of states and allowed transitions

## 2) Price/Discount Integrity
1. Add item to cart; intercept requests
2. Modify price/discount fields; remove monetary fields to force server calc
3. Try multiple coupon codes, negative discount, currency flips
4. Complete checkout or reach summary

Validate: totals, ledger results, inventory changes, invoice

## 3) Step/Approval Bypass
1. Navigate to penultimate step (summary)
2. Call finalization endpoint directly (confirm/complete) without payment token
3. Remove/alter flags: agreed_tos=false, 2fa_token=null
4. Check if order completes or feature enables

Evidence: order id created, status advanced, missing payment

## 4) Role and Ownership Checks
1. With user A, create object (invoice/order/profile)
2. With user B, change id/tenant field to access A’s object
3. Attempt admin-only actions from user role by changing feature flags/role_id

Expected: 403; if 200/204 and side effects occur -> flaw

## 5) Quotas and Rate Limits
1. Send bursts to sensitive endpoints (OTP, coupon apply)
2. Rotate headers (X-Forwarded-For, device_id) to probe scoping
3. Measure responses for 429 or soft blocks

Failing indicators: unlimited actions, shared quotas across tenants

## 6) Concurrency and Replay
1. Identify critical endpoint (apply-coupon, refund, confirm)
2. Launch 5-20 parallel requests (Turbo Intruder)
3. Replay success callback (payment/refund webhooks) twice

Result: double discounts/credits or inconsistent state

## 7) Subscription/Billing Edge Cases
1. Start trial; change payment method; cancel and re-enable
2. Upgrade then immediate downgrade; observe pro-ration
3. Change billing_anchor/next_invoice date via API

Outcome: financial gain without equivalent payment

## 8) Inventory and Order Lifecycle
1. Modify order_status without prior states
2. Attempt return/refund before shipped
3. Reserve stock via cart locks; abandon checkout

Look for: stuck reserved stock, negative totals

## Reporting Template
- Title: Concise summary
- Impact: Financial loss/data integrity/tenant isolation
- Affected Endpoints: list
- Steps to Reproduce: numbered steps with requests and responses
- Evidence: screenshots, request/response pairs, ledger entries
- Severity: using repo rubric
- Remediation: link to Remediation.md sections

## Safe Testing Notes
- Use $0 SKUs or sandbox tokens
- Coordinate with business owners for refund-like actions
- Avoid mass impact; throttle tests
