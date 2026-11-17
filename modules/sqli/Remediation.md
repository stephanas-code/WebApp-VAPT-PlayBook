# SQL Injection Remediation Guide

This guide outlines defensive controls to prevent SQL injection across languages and frameworks.

## Core Principles

1. Use parameterized queries (prepared statements) everywhere
2. Avoid dynamic SQL concatenation with user input
3. Validate and sanitize inputs with allowlists
4. Apply least-privilege to database accounts
5. Centralize DB access patterns (ORM/repository) and enforce secure defaults
6. Fail securely: no verbose DB errors to users; log safely
7. Defense-in-depth: WAF, RASP, query allowlisting where feasible

## Language/Framework Patterns

### PHP (mysqli/PDO)

- Do: Prepared statements
```php
$stmt = $pdo->prepare('SELECT * FROM products WHERE id = :id');
$stmt->execute(['id' => (int)$id]);
```
- Don’t: string concatenation
```php
$query = "SELECT * FROM products WHERE id = $id"; // vulnerable
```

### Python (psycopg2/MySQL Connector)
```python
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
```

### Java (JDBC)
```java
PreparedStatement ps = con.prepareStatement("SELECT * FROM orders WHERE user_id=?");
ps.setInt(1, userId);
```

### .NET (SqlClient)
```csharp
using var cmd = new SqlCommand("SELECT * FROM u WHERE id=@id", conn);
cmd.Parameters.AddWithValue("@id", id);
```

### Node.js (mysql2/pg)
```js
await db.execute('SELECT * FROM posts WHERE slug = ?', [slug]);
```

## Validation & Encoding

- Numeric IDs: ^[0-9]+$
- Slugs: ^[a-z0-9-]{1,100}$
- Dates: strict parsing (e.g., ISO 8601)
- Encode values only for the sink: SQL uses parameters, not escaping as primary defense

## ORM Safe Usage

- Prefer ORM query builders
- Avoid raw SQL; if necessary, use bound parameters
- Disable/avoid string interpolation in ORM raw methods

## Database Hardening

- Separate accounts: readonly vs readwrite
- Revoke dangerous privileges (e.g., xp_cmdshell, FILE)
- Enforce TLS in transit; encrypt sensitive columns at rest
- Enable auditing and anomaly detection (e.g., excessive errors, time delays)

## Error Handling & Logging

- Generic user messages
- Structured logs with context and correlation IDs
- Redact PII/credentials

## Testing & CI

- Add unit tests to assert parameterized calls
- SAST for injection patterns
- DAST with SQLMap/Burp in pre-prod
- Secure code reviews

## Example Secure Refactor

Before:
```python
query = f"SELECT * FROM accounts WHERE name = '{name}'"
```
After:
```python
query = "SELECT * FROM accounts WHERE name = %s"
cursor.execute(query, (name,))
```

## Checklist

- [ ] All DB calls are parameterized
- [ ] No concatenated SQL with user input
- [ ] DB user permissions are least-privilege
- [ ] Central input validation
- [ ] Errors do not leak DB info
- [ ] Security tests in CI/CD

## 2026 Modern & AI-Assisted Defensive Updates

- **Compile-Time Type-Safe Query Builders & Tagged Templates**: Enforce compile-time or linter-verified query construction (e.g., Go `sqlc`, Rust `SQLx` compile-time macros, TypeScript Prisma / Drizzle ORM parameterized tagged templates, or Java `JOOQ`) where raw string interpolation fails compilation or static type checking (`trusted-sql` types).
- **Dynamic Identifier Allowlisting (ORDER BY / Column Names)**: Because SQL parameters cannot bind identifiers (table or column names), enforce strict enum/map allowlists before query assembly:
  ```typescript
  const SORT_COLUMNS: Record<string, string> = { created: 'created_at', price: 'unit_price' };
  const col = SORT_COLUMNS[userSortKey] ?? 'created_at';
  ```
- **AI-Assisted Natural-Language-to-SQL (Text-to-SQL) Guardrails**: If an application uses an LLM to generate analytical queries, never execute LLM-generated SQL over a privileged connection; parse the AST (e.g., via `sqlglot`) to allowlist `SELECT`-only statements, enforce PostgreSQL Row-Level Security (RLS), and run against a read-only replica with strict statement timeouts.
- **AI-Assisted SAST & Automated Parameterization Refactoring**: Use AI-augmented SAST in CI/CD (CodeQL, Semgrep Assistant) to detect second-order SQL injection and dynamic ORM `.raw()` / `FromSqlRaw()` sinks, automatically generating pull requests that convert string interpolation into bound parameters.

---

See also: [README.md](./README.md), [Payloads.md](./Payloads.md), [Reproduce.md](./Reproduce.md), [BurpSuite-Guide.md](./BurpSuite-Guide.md)
