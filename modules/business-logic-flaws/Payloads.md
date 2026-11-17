# Payloads and Test Scenarios for Business Logic Flaws

This document provides curated payloads, manipulations, and test scenarios to probe typical business logic weaknesses. Adapt payloads to the target’s exact workflow, parameters, and state machine.

## Index
- Price/Discount manipulation
- Quantity and negative values
- Workflow/state bypass
- Role/authorization edge-cases
- Rate limiting and quotas
- Time-based and sequence exploits
- Abuse of free trials/coupons/referrals
- Idempotency and replay
- Concurrency and race conditions
- Inventory and stock logic
- Order lifecycle tampering
- Subscription/billing logic

## General Tips
- Always capture a baseline valid flow first.
- Identify all state-bearing parameters (ids, state, step, flags, totals) at client and server.
- Compare UI-visible totals with server-calculated totals.
- Look for trust on client: hidden fields, JS-computed price, disabled inputs.
- Try cross-step parameter reuse/substitution.

---

## 1) Price and Discount Manipulation

Parameters to watch: price, unit_price, amount, subtotal, tax, shipping, discount, coupon, promo, loyalty, currency, exchange_rate.

Examples:
- Modify price in cart item PATCH/PUT:
  - Original: {"sku":"A1","qty":1,"price":1999}
  - Test: {"sku":"A1","qty":1,"price":1}
- Force currency mismatch:
  - currency=USD while unit_price set in cents vs dollars
- Stacked coupons:
  - Add multiple coupons where UI allows one: coupon=SPRING10&coupon=WELCOME20
- Negative discount overflow:
  - discount=-999999 leading to credit/refund
- Floating point rounding:
  - price=0.009 with qty=10000; tax rounding per-line vs per-order
- Change price server ignores:
  - Remove price field entirely to see if server trusts client-subtotal

Expected Signals:
- Total decreases incorrectly; negative order totals; gift card auto-credit; refund creation.

---

## 2) Quantity, Negative and Boundary Values

- qty=0, -1, -9999, 999999
- Min/max overflows: min_qty=1 but server accepts 0
- Weight/size-based shipping: set weight=0 or negative to remove shipping cost
- Tier pricing thresholds: qty=99 when discount at 100; race two requests to cross threshold

Observe: inventory changes, price tiers, shipping/tax anomalies.

---

## 3) Workflow and State Bypass

Target multi-step flows: checkout, onboarding, KYC, approval, password reset, consent.

- Skip steps: change step=4 directly or omit required step tokens
- Reuse pre-approval token to call finalization endpoint directly
- Modify hidden flags: isVerified=true, kyc_status=approved
- Jump APIs:
  - GET /checkout/summary then POST /checkout/confirm without payment
- Omit required artifacts: e.g., "agreed_tos": false; "2fa_token": null

Signals: order completion without payment, KYC bypass, feature enablement.

---

## 4) Role/Authorization Edge-Cases

- Perform admin action with normal user by changing role_id, plan, feature flags
- Access cross-tenant resources: org_id, tenant_id, company_id swapped to another value
- Horizontal privilege logic: change user_id in business operation (approve, refund, transfer)
- Vertical privilege logic: invoke /admin/* endpoints discovered in UI flows

Check: response data differences, side effects (approvals, status changes).

---

## 5) Rate Limiting, Quotas, and Abuse of Limits

- Rapid invoke sensitive endpoints (OTP/email code, password reset, coupon apply)
- Parameter rotate to bypass naive IP or user limit (X-Forwarded-For, device_id)
- Bulk creation/cleanup gaps: create 1000 drafts, carts, or free artifacts
- Account lockout business rules: lock victim then launch secondary attack

Evidence: missing 429s, shared pools across tenants, resource exhaustion.

---

## 6) Time-based and Sequence Exploits

- Backdate effective_date to bypass new pricing
- Exploit cutoff windows (refund within 30 days): set purchase_time just inside boundary
- Reordering steps to lock in discount before removing qualifying item
- Change time_zone to manipulate daily limits

---

## 7) Free Trials, Coupons, Referrals Abuse

- Trial reset: new email same payment_token or device_id
- Coupon scope mismatch: apply category coupon to excluded SKU via API
- Referral self-invite: refer self or circular referrals
- Abuse pro-rated refunds: upgrade then immediate downgrade for credit gain

---

## 8) Idempotency and Replay

- Reuse idempotency-key with changed body to see which field is honored
- Replay payment confirmation callback to duplicate credits
- Resubmit cancel/refund/approval endpoints

Indicators: duplicate orders, double credits, inconsistent ledger.

---

## 9) Concurrency and Race Conditions

Technique: send parallel requests with same cart/order id.

- Double-spend coupon; consume same gift card code twice
- Simultaneous add/remove item to desync totals
- Parallel refund + ship to get both money and product

Tools: Turbo Intruder (Burp), race.sh, hey/ab with reused cookies.

---

## 10) Inventory and Stock Logic

- Purchase when stock=0 via direct POST
- Reserve inventory without checkout completion
- Manipulate warehouse_id or fulfillment_priority
- Backorder flag toggles

---

## 11) Order Lifecycle Tampering

- Change order_status=shipped before payment_captured=true
- Trigger return without shipped state
- Set delivery_address after fraud checks
- Swap payment_method to COD after approval

---

## 12) Subscription and Billing Logic

- Prorate misuse: repeatedly upgrade/downgrade around billing cycle
- Billing_anchor changes via API to delay charges
- Quantity-based seats: allocate more seats than paid
- Cancel-at-period-end abuse with continued premium access

---

## Annotated Examples

1) Skipping Payment, Completing Order
- Flow: cart -> shipping -> payment -> confirm
- Attack:
  - GET /checkout/summary
  - POST /checkout/confirm {"order_id":"123","payment_token":null}
- Expected: order completes; payment missing; inventory decremented; invoice created
- Risk: direct financial loss, reconciliation issues

2) Coupon Double-Apply via Race
- Prepare two POST /apply-coupon requests using same coupon
- Fire concurrently
- Expected: coupon applied twice, total < expected or negative
- Mitigation: atomic checks, unique constraint per order+coupon, idempotency key

3) Tenant Boundary Bypass
- Change org_id in PATCH /invoices/{id} to target another tenant
- Expected: cross-tenant data manipulation
- Mitigation: enforce ownership checks on server based on session context

4) Referral Abuse
- POST /referral {"referrer":"me","invitee":"me"}
- Expected: credit granted to self
- Mitigation: graph-based checks, anti-fraud heuristics, device/payment correlation

---

## Logging and Evidence Collection
- Capture request/response pairs and server-calculated totals
- Record timestamps, idempotency keys, and correlation ids
- Note any discrepancy between UI and server results

## Safety and Scope
- Respect legal scope and rate limits provided by owner
- Avoid causing financial harm; simulate with test envs, $0 items, or sandbox tokens

---

## Quick Payload Snippets

- Negative qty: qty=-1
- Remove shipping: weight=0 or shipping_fee=0
- Force free: price=0&coupon=100PCT
- Bypass step: step=confirm&payment_token=
- Cross-tenant: org_id=other-uuid
- Replay: reuse Idempotency-Key: abc-123 with modified body

