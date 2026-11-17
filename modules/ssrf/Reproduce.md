# SSRF Reproduce

## Objective
Provide reproducible steps to identify and confirm SSRF vulnerabilities in the target application while minimizing impact.

## Preconditions
- Burp Suite installed and intercepting traffic
- Test account (non-privileged)
- Target environment URL and scope defined
- Out-of-band interaction service (Burp Collaborator or webhook.site)

## Discovery Workflow
1. Map features that fetch remote resources (URL fetchers, image loaders, webhooks, PDF renderers, importers).
2. Identify parameters likely to be URLs or network locations, e.g. url, link, feed, callback, webhook, image, avatar, import, fetch, source.
3. Send candidate requests to Burp Repeater.
4. Try swapping the parameter value with a benign external URL you control. Confirm server-side fetch by observing:
   - Reflected content from that URL
   - Changes in server response times
   - Outbound DNS/HTTP hit on your listener

## Test Cases

### Case 1: Localhost Access
- Replace URL parameter with:
  - http://127.0.0.1
  - http://localhost:80
  - http://[::1]
- Expected indicators: error messages from internal service, timeouts, or headers revealing internal server.

### Case 2: Private Network Targets
- Try http://10.0.0.1, http://172.16.0.1, http://192.168.0.1 on common ports (22, 80, 443, 8080, 8443).
- Observe status codes, timing, and any leaked banners.

### Case 3: Cloud Metadata
- If app runs in cloud, test metadata endpoints:
  - AWS: http://169.254.169.254/latest/meta-data/
  - GCP: http://metadata.google.internal/computeMetadata/v1/ with header Metadata-Flavor: Google
  - Azure: http://169.254.169.254/metadata/instance?api-version=2021-02-01 with header Metadata: true
- Success criteria: any 200 OK or metadata key listing. Stop and sanitize results; do not exfiltrate secrets beyond proof-of-concept.

### Case 4: Alternate Protocols
- Try file://, gopher://, dict:// schemes (if not blocked) to assess deserializers or fetchers.
- For blind conditions, prefer out-of-band techniques rather than destructive endpoints.

### Case 5: Redirect Chains
- Host a redirector that 302s to internal IP (e.g., /r?to=http://127.0.0.1).
- Provide initial external URL; confirm whether server follows redirect to internal.

### Case 6: DNS Rebinding/Host Confusion
- Use a DNS rebinding service; present hostname resolves public -> private.
- Validate whether the application validates IP after DNS resolution.

### Case 7: Blind SSRF (OAST)
- Supply http://<your-collab-id>.burpcollaborator.net
- Confirm callbacks in Collaborator; correlate request ID and timing.

## Evidence Collection
- Save request/response pairs from Burp Repeater
- Record out-of-band interaction logs (timestamp, source IP, HTTP method/path)
- Screenshots of metadata listings or internal service banners (mask sensitive data)
- Note whether redirects were followed and how many hops

## Impact Validation
- Enumerate reachable internal services (ports and endpoints) without disrupting availability
- Safely list high-value targets if accessible (e.g., Elasticsearch _cluster/health)
- Confirm cloud metadata exposure at least to top-level directories

## Safe Handling
- Use rate limits and backoff to avoid scanning aggressively
- Do not write data to internal services (read-only probes)
- Coordinate with stakeholders before cloud metadata tests

## Reporting Template
- Parameter/endpoint: <path>
- HTTP method: GET/POST/...
- SSRF type: Internal/External/Cloud metadata/Blind
- Proof of exploitation: evidence summary
- Business impact: lateral movement, data access, service control
- Severity rationale: likelihood + impact + compensating controls
- Recommended remediation: see Remediation.md

## Regression Steps
- After fix, repeat Case 1–7 and ensure:
  - Non-HTTP schemes rejected with 4xx
  - Private/IP ranges blocked
  - Redirects re-validated each hop
  - Metadata endpoints unreachable
  - Only allowlisted hosts reachable
