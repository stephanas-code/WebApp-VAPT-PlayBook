# SSRF Burp Suite Guide

## Objective
Provide a consistent Burp Suite workflow to identify, exploit, and confirm SSRF vulnerabilities and gather high-quality evidence.

## Tools & Setup
- Burp Suite Professional/Community
- Extensions (recommended):
  - Collaborator Everywhere
  - Logger++
  - Active Scan++ (Pro)
  - Param Miner
- Target scope configured to the application domain

## Workflow Overview
1. Discovery: Identify candidate endpoints and parameters handling URLs
2. Validation: Confirm server-side fetching and visibility
3. Exploitation: Attempt internal, cloud metadata, and protocol vectors
4. Evidence: Capture definitive artifacts and correlate Collaborator hits
5. Reporting: Map to business impact and remediation

## Step-by-Step

### 1) Crawl and Parameter Discovery
- Use Proxy + Target + Site map to locate features that ingest URLs (webhooks, imports, image fetchers, PDF renderers).
- Use Param Miner to guess hidden params like url, link, src, feed, callback, webhook.
- Send relevant requests to Repeater.

### 2) Confirm SSRF Behavior
- Replace suspected parameter with a URL you control (e.g., https://<collab>.oast.live/test).
- Observe in Repeater response:
  - Content from your server reflected or referenced
  - Timing differences (timeouts, fast failures)
  - New headers (Via, X-Forwarded-For) indicating server-side proxying
- Check Collaborator for DNS/HTTP interaction logs.

### 3) Enumerate Vectors
- Internal access: http://127.0.0.1, http://localhost, http://10.0.0.1:80, http://192.168.0.1:8080
- Cloud metadata:
  - AWS: http://169.254.169.254/latest/meta-data/
  - GCP: http://metadata.google.internal/computeMetadata/v1/ (add Metadata-Flavor: Google)
  - Azure: http://169.254.169.254/metadata/instance?api-version=2021-02-01 (add Metadata: true)
- Protocol tests: file://etc/passwd, gopher://, dict:// (only in safe sandboxes)
- Redirect chains: host a 302 redirector to internal IP and see if followed
- DNS rebinding: use Collaborator or dedicated service to rebind to private IP

### 4) Blind SSRF with Collaborator
- Insert URL: http://<collab-id>.burpcollaborator.net
- Trigger the server-side fetch and monitor Collaborator for: Interaction type (DNS/HTTP), source IP, User-Agent, timestamp.
- Correlate with application response time to prove causality.

### 5) Automation (Optional)
- Use Burp Intruder to iterate internal IPs/ports for light-touch discovery (respect rate limits):
  - Positions: payload marker around the URL host or port
  - Payloads: 127.0.0.1, 10.0.0.1, 169.254.169.254, ports 22,80,443,8080,8443
  - Grep - Match: timeouts, specific banners, status codes 200/302/500

### 6) Evidence Packaging
- Save Repeater requests/responses with comments per test case.
- Export Collaborator logs showing inbound callbacks.
- Include screenshots of metadata directory listings (mask sensitive values).
- Document redirect behavior and number of hops.

### 7) Severity and Impact
- Tie findings to potential lateral movement and data exposure:
  - Access to cloud metadata could yield credentials/tokens
  - Internal admin interfaces may enable configuration tampering
  - Access to internal data stores (Elasticsearch/Redis) can expose data

### 8) Retesting After Fixes
- Confirm non-HTTP schemes rejected server-side
- Verify private IP ranges and metadata IP blocked
- Ensure redirects are validated at each hop
- Attempt Collaborator callback again – should be silent
- Validate only allowlisted hosts reachable

## Useful Burp Filters
- Response time > 3s
- Status code anomalies per endpoint
- Responses containing internal hostnames or service banners

## References
- PortSwigger SSRF Academy
- OWASP SSRF Prevention Cheat Sheet
- Payloads.md and Reproduce.md in this module
