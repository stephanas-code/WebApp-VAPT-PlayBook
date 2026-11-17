# IDOR Remediation Guide

## Principles
- Deny by default: all object access requires explicit allow.
- Enforce on the server: never trust client-side checks.
- Consistent, centralized authorization: one way to check access everywhere.
- Defense in depth: layered controls (authn, authz, validation, monitoring).

## Access Control Patterns

### 1) Enforce Ownership/Scope Checks
- Always verify the requester is allowed to access the object by ownership, role, or policy.

Pseudocode:
```python
# Flask-like example
@app.get('/api/orders/<order_id>')
def get_order(order_id):
    user = current_user()
    order = Order.get(order_id)
    if not order or not can(user, 'read', order):
        raise Forbidden()
    return serialize(order)
```

### 2) Replace Direct IDs with Indirect References
- Use opaque references or contextual endpoints:
  - /api/users/me/profile instead of /api/users/{id}/profile
  - Server resolves identity from session/JWT

### 3) Consistent Authorization Middleware
```javascript
// Express.js
function authorize(action, resourceGetter) {
  return async (req, res, next) => {
    const resource = await resourceGetter(req);
    const user = req.user;
    if (!policy.can(user, action, resource)) return res.sendStatus(403);
    req.resource = resource;
    next();
  };
}

app.get('/api/accounts/:id', authorize('read', req => Account.findById(req.params.id)), handler)
```

### 4) Least Privilege and Scoping
- Narrow JWT scopes/claims; avoid broad org-wide claims unless needed.
- Implement tenant isolation for multi-tenant systems.

### 5) Prevent Mass Assignment
- Whitelist fields; ignore or reject unexpected fields.
```ruby
# Rails
params.require(:user).permit(:name, :bio)
```
- Validate server-side that role, ownerId, orgId cannot be changed by clients.

### 6) Validate Input and Normalize
- Normalize IDs before checks (case, unicode, trimming).
- Rate-limit enumeration attempts.

### 7) Secure File/Object Access
- Store files outside web root.
- Authorize before generating signed, time-limited URLs.
- Bind URLs to requester and scope.

### 8) Logging and Monitoring
- Log object access with subject, object, decision, and reason.
- Alert on abnormal access patterns (high 403/404, sequence scans).

## Implementation Examples

### Spring Security (Java)
```java
@PreAuthorize("#id == authentication.principal.id || hasRole('ADMIN')")
@GetMapping("/api/users/{id}")
public UserDto getUser(@PathVariable Long id) { ... }
```

### Django
```python
from rest_framework.permissions import BasePermission
class IsOwnerOrAdmin(BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.owner == request.user or request.user.is_staff
```

### Node/NestJS
```ts
@UseGuards(JwtAuthGuard, PoliciesGuard)
@CheckPolicies((user, resource) => user.id === resource.ownerId || user.isAdmin)
@Get('orders/:id')
findOne() {}
```

## Process Controls
- Threat modeling for object access.
- Unit/integration tests for authz on sensitive endpoints.
- Pre-merge security review for routes exposing object IDs.
- Pen tests focused on BOLA/IDOR.

## Developer Checklist
- [ ] Does the endpoint use requester context (session/JWT) instead of client-passed IDs where possible?
- [ ] Are object-level checks performed for every read/write/delete?
- [ ] Are indirect references used or strong policy checks in place?
- [ ] Are privileged fields protected from mass assignment?
- [ ] Are multi-tenant boundaries enforced?
- [ ] Are access decisions logged and monitored?

## 2026 Modern & AI-Assisted Defensive Updates
- **Database-Level Row-Level Security (RLS) & Scoped ORM Contexts**: Enforce tenant and owner isolation directly at the data layer (e.g., PostgreSQL Row-Level Security policies bound to session variables or Prisma/Hibernate automatic tenant-scoping extensions) so queries cannot return cross-tenant records even if an application-layer check is omitted.
- **Fine-Grained Authorization Engines (ReBAC / ABAC)**: Centralize authorization using Relationship-Based Access Control (Google Zanzibar-style engines such as OpenFGA, SpiceDB) or policy engines (AWS Cedar, OPA) instead of ad-hoc inline conditionals.
- **AI-Assisted BOLA/BOPLA Auditing & Automated Matrix Testing**:
  - Use AI-assisted static dataflow analysis in CI to trace route parameters (`req.params.id`, GraphQL node IDs) to database queries and flag any query lacking a join or filter on the authenticated `principal.id` or `tenant_id`.
  - Automatically generate multi-persona authorization test suites (User A, User B in same tenant, User C in different tenant, unauthenticated) in CI/CD to assert `403/404` responses across all REST/GraphQL/gRPC object endpoints and field-level mutations (BOPLA).

## References
- OWASP Top 10 A01: Broken Access Control
- OWASP ASVS V4 / V5.0: Access Control
- OWASP API Security Top 10: API1 (BOLA) & API3 (Broken Object Property Level Authorization - BOPLA)
