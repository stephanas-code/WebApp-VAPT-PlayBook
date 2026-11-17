# IDOR Reproduction Lab and Steps

This guide demonstrates how to reproduce and validate IDOR/BOLA vulnerabilities safely using test accounts and Burp Suite/Postman.

## Lab Setup
- Accounts: userA (basic), userB (basic), admin (optional)
- Target app: endpoints that expose object IDs (e.g., /api/users/{id}, /api/orders/{id})
- Tools: Burp Suite (Community/Pro), browser, Postman

## Test Cases

### 1) Horizontal Access (User-to-User)
Goal: userA accesses userB’s resources.
Steps:
1. Login as userA, capture a baseline request, e.g.: GET /api/users/1001/profile
2. Send to Repeater.
3. Change ID to 1002 (userB).
4. Compare responses (status, body, size). Evidence of success: 200 with userB data, or detailed error.

### 2) Vertical Access (User-to-Admin)
Goal: userA accesses admin-only objects.
Steps:
1. Login as userA, request: GET /api/admin/reports/2024
2. If blocked, try indirect paths: /api/reports/2024?scope=admin
3. Attempt header/cookie changes: X-Role: admin (should be ignored server-side).

### 3) Body Parameter ID Switch
1. Normal flow: userA updates own profile via POST /api/profile/update with { userId: 1001 }.
2. Change userId to 1002 and resend.
3. Check if update occurs or is rejected with 403.

### 4) File/Object Access
1. Download own invoice: GET /invoices/INV-1001.pdf
2. Change to another: /invoices/INV-1002.pdf
3. Verify if direct access is blocked.

### 5) Mass Assignment
1. Submit profile update including forbidden fields (role, organizationId, ownerId).
2. Confirm server rejects/ignores changes.

## Verification Checklist
- [ ] Different user’s data returned (PII, order details, messages)
- [ ] State-changing action on another user succeeds
- [ ] Server returns distinct errors or timing indicating existence
- [ ] Logs show missing/insufficient object-level checks

## Proof of Concept Template

Example Request:
```http
GET /api/orders/ORD-12346 HTTP/1.1
Host: target
Cookie: session=...
```

Expected Vulnerable Behavior:
- Status 200
- Response contains another user’s order data

Mitigation Verification:
- Status 403/404 or redaction
- Access decision logged

## Reporting
Include:
- Affected endpoints and object types
- User roles and accounts used
- Repro steps with paired requests (own vs target)
- Impact assessment and severity
- Recommended fixes (link to Remediation.md)
