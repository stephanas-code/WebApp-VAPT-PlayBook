# IDOR with Burp Suite: Detection and Exploitation Guide

This guide provides practical steps to find and validate IDOR/BOLA using Burp Suite. Mirror these steps across web and API testing.

## Setup
- Proxy your browser through Burp Suite (or use the built-in browser).
- Ensure you can log in as multiple roles (userA, userB, admin if permitted).
- Configure project scope to target domain only.

## Discovery Workflow
1. Crawl/Map
   - Use Proxy + Target + Engagement tools to map endpoints with object identifiers (id, userId, orderId, invoiceId, fileId, uuid, slug).
   - Send promising requests to Repeater for manual probing.
2. Identify object references
   - Note path segments and parameters that look like resource selectors.
   - Watch for predictable patterns and sequences.
3. Baseline requests
   - Capture “owned” requests (e.g., userA -> /api/users/1001/profile) for each object type.

## Manual Testing in Repeater
1. Horizontal ID Switch
   - Duplicate the request and change the object ID from userA’s to userB’s.
   - Compare status codes, body content, response lengths, and error messages.
2. Vertical Escalation Attempts
   - Try accessing admin-only resources as a basic user.
   - Observe whether responses leak metadata (403 with object info) or allow access.
3. Alternate Selectors
   - Try email/username/slug/uuid in place of numeric IDs if accepted.
4. Header/Cookie Tampering
   - Modify headers like X-User-ID, X-Org-ID; alter cookies like user_id or role.
   - Expect secure apps to ignore these changes or return 403.
5. Body Parameter Manipulation
   - For JSON endpoints, change userId/ownerId/organizationId fields.
   - Check for mass assignment.

## Automation with Intruder (Optional)
- Position payload markers around the object ID.
- Use simple number payloads (1..N), UUID wordlists, or custom IDs.
- Grep-Match/Grep-Extract
  - Match unique victim strings or response size deltas to flag potential IDOR.

## Using Comparer
- Send both “owned” and “target” responses to Comparer to diff content.
- Significant diffs with 200/206 responses indicate possible IDOR.

## Collaborator (Edge Cases)
- Rarely needed for IDOR directly; useful to detect SSRF or blind behaviors during broader testing.

## Logging Evidence
- Save two requests per finding: A) authorized own object, B) unauthorized target object.
- Include cookies/JWT claims for context (redact secrets in reports).

## Common Pitfalls
- Cached responses masking access control: disable cache or add cache-busting headers.
- Soft-deleted vs hard-deleted resources: 404 vs 403 inconsistencies.
- Multi-tenant scopes: ensure tests cross tenant boundaries only if authorized.

## Remediation Verification
- Re-test after fixes: confirm 403/404 and no sensitive data leakage.
- Ensure logs show deny decisions and that indirect references are used.

## Quick Checklist
- [ ] Multiple roles tested (userA, userB, admin if allowed)
- [ ] Path, query, body, header, cookie selectors manipulated
- [ ] Response codes, sizes, and bodies compared
- [ ] Evidence captured and reported
- [ ] Fixes validated against Remediation.md recommendations
