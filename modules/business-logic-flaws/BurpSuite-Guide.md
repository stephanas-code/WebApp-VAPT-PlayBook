# Burp Suite Guide: Business Logic Flaws Testing

Burp Suite offers powerful tools for intercepting, modifying, and repeating requests—ideal for probing business logic weaknesses. This guide focuses on workflows and extensions for logic testing.

## Configuration

1. Proxy Setup: ensure browser targets Burp proxy (default 127.0.0.1:8080)
2. Scope: add target to Scope (Target > Add to scope)
3. History Filters: enable "Hide items without responses" and filter by status/extension
4. Extensions:
   - Turbo Intruder (race conditions)
   - Autorize (authorization context)
   - Logger++ (advanced logging)
   - JSON Beautifier, Param Miner

## Workflow: Baseline Capture
1. Perform a clean happy path through target flow (checkout, signup, subscription)
2. In Proxy > HTTP History, filter to in-scope items
3. Right-click chain of requests > Save selected items or create Macro for replay

## Tampering Price/Discount Fields
1. Intercept POST/PUT/PATCH at cart or order update
2. Locate JSON body with price, unit_price, discount, subtotal
3. Change values:
   - price: 1999 -> 1
   - discount: 10 -> 999999 or -999
   - qty: 1 -> -5
4. Forward; observe response for new total or error
5. Repeat Request (Ctrl+R) to iterate quickly

Checks:
- Does server recalculate or trust client?
- Does negative qty produce refund?
- Multiple coupon stacking?

## Workflow/Step Bypass
1. Identify multi-step flows via sequence of requests
2. Note step or state param (step=2, status=pending, payment_captured=false)
3. Jump directly to finalization endpoint, skipping payment step
4. Remove or nullify required artifacts (payment_token, kyc_token, agreed_tos)

In Repeater:
- Copy finalization request
- Strip payment_token or set step=confirm
- Send; check order completion, inventory decrement

## Role and Tenant Logic Testing with Autorize
1. Install Autorize extension
2. Configure two sessions: user A (attacker) and user B (victim)
3. Browse as user B; Autorize auto-replays requests with A’s session
4. Check for 200/data leakage indicating broken authorization

Additionally:
- Manually swap user_id/tenant_id/org_id in JSON/query
- Use Intruder to brute-force ids (see below)

## Concurrency and Race Condition Testing
Use Turbo Intruder for parallel requests.

1. Send target request (e.g., apply-coupon) to Turbo Intruder
2. Select race.py or custom script:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(..., concurrentConnections=10)
    for i in range(20):
        engine.queue(target.req)
        
def handleResponse(req, interesting):
    table.add(req)
```

3. Launch attack; observe duplicated effects (discount applied twice, double credit)

## Intruder for Fuzzing Quantities and Limits
1. Send request to Intruder (Ctrl+I)
2. Mark parameter positions: qty=§1§
3. Payload sets:
   - Numbers: -999999, -1, 0, 1, 9999999
   - Nulls: null, empty string
   - Type juggling: "1", 1.0, true, [], {}
4. Start attack; analyze responses for totals, errors, stock changes

## Idempotency and Replay Testing
1. Find critical request (payment confirm, refund, credit)
2. Note Idempotency-Key or similar headers
3. Replay with same key but modified body (amount, recipient)
4. Check if second replay succeeds or returns cached response
5. Replay callbacks/webhooks from payment provider to app endpoint

## Match and Replace Rules
Automate modifications for repeated testing.

1. Proxy > Options > Match and Replace
2. Add rule:
   - Type: Request body
   - Match: "price":(\d+)
   - Replace: "price":1
3. Enable rule; all requests automatically modified

## Logging and Evidence
1. Logger++ extension logs all traffic with custom fields
2. Export filtered results as CSV/JSON for reporting
3. Use Comparer to diff before/after tampering

## Advanced: Macros for Multi-step Flows
1. Project Options > Sessions > Macros > Add
2. Select sequence of requests (login, add-to-cart, checkout)
3. Define parameter extraction from intermediate responses (CSRF, session tokens)
4. Use in Intruder or Scanner for authenticated/stateful attacks

## Common Pitfalls
- Forgetting to update dynamic tokens (CSRF, nonce)
- Testing on production without coordination
- Not capturing baseline for comparison

## Quick Checklist
- [ ] Capture clean flow in Proxy history
- [ ] Test price/discount manipulation in Repeater
- [ ] Skip steps, remove tokens; replay finalization
- [ ] Use Autorize for cross-user/tenant checks
- [ ] Race critical endpoints with Turbo Intruder
- [ ] Fuzz qty/limits/currency in Intruder
- [ ] Replay callbacks and verify idempotency
- [ ] Log all findings; export requests/responses

## Extensions Reference
- Autorize: context-based auth testing
- Turbo Intruder: race/concurrency
- Logger++: comprehensive request logging
- Param Miner: discover hidden params
- JSON Beautifier: readable JSON in Repeater

## Summary
Burp Suite's interception and replay capabilities make it ideal for business logic testing. Combine Repeater for manual exploration, Intruder for fuzzing, and Turbo Intruder for race conditions. Always coordinate scope and test safely.
