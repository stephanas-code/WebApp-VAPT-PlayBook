# SSRF Remediation

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Root Causes](#root-causes)
3. [Remediation Strategy](#remediation-strategy)
4. [Secure Coding Guidelines](#secure-coding-guidelines)
5. [Infrastructure Hardening](#infrastructure-hardening)
6. [Monitoring and Detection](#monitoring-and-detection)
7. [Verification and Regression Tests](#verification-and-regression-tests)
8. [References](#references)

---

## Executive Summary
Server-Side Request Forgery (SSRF) occurs when the server fetches a remote resource based on user-controlled input. Attackers can pivot to internal networks, access cloud metadata, or reach administrative interfaces. Proper input validation, allowlisting, network egress controls, and metadata protections are required to mitigate.

---

## Root Causes
- Unsafely using user-supplied URLs in server-side fetchers (curl, URLConnection, HttpClient)
- Missing allowlist validation of destination hosts/schemes/ports
- Following redirects to internal or sensitive targets
- Permitting dangerous protocols (file://, gopher://, dict://, ftp://)
- Lack of egress firewalling from application subnets
- Cloud metadata endpoints accessible from the app runtime

---

## Remediation Strategy
1) Validate and constrain destination requests at the application layer
2) Block internal, link-local, and loopback ranges at multiple layers
3) Restrict protocols to HTTP/HTTPS only and enforce standard ports
4) Disable server-side redirects or re-validate each redirect hop against policy
5) Enforce egress network policies and segmented VPCs/VNETs
6) Protect cloud metadata endpoints (IMDSv2 on AWS, required headers on GCP/Azure)
7) Add monitoring, alerting, and pre-deployment security tests

---

## Secure Coding Guidelines

### General Policy
- Use a strict allowlist of hostnames or service IDs. Avoid wildcards.
- Resolve DNS and validate the resolved IP against allowlisted CIDR blocks.
- Block private, loopback, link-local, multicast, and reserved ranges.
- Deny non-HTTP schemes. Only allow http and https.
- Restrict ports to 80 and 443 unless explicitly needed.
- Do not allow fragment identifiers, userinfo, or embedded credentials.
- Disable or limit redirects; if allowed, re-validate after each hop.

### Pseudocode (Language-agnostic)
```
function isSafeUrl(inputUrl):
    url = parseURL(inputUrl)
    if url.scheme not in ["http", "https"]:
        return false
    if url.port not in [80, 443, null]:
        return false
    if url.userinfo is not null:
        return false

    host = url.hostname
    if host not in ALLOWED_HOSTS and not host.endsWith(ALLOWED_DOMAIN_SUFFIX):
        return false

    ips = dnsResolve(host)
    for ip in ips:
        if ip in BLOCKED_CIDRS: # RFC1918, loopback, link-local, multicast, metadata
            return false
    return true

function safeFetch(inputUrl):
    if not isSafeUrl(inputUrl):
        raise ValidationError("Destination not permitted")
    response = httpClient.fetch(
        url=inputUrl,
        timeout=5s,
        followRedirects=false,
        headers={"User-Agent": FIXED_UA}
    )
    return response
```

### Java (OkHttp) Example
```java
HttpUrl url = HttpUrl.parse(input);
if (url == null || !(url.scheme().equals("http") || url.scheme().equals("https"))) throw new IllegalArgumentException();
int port = url.port();
if (!(port == 80 || port == 443)) throw new IllegalArgumentException();
String host = url.host();
if (!ALLOWED_HOSTS.contains(host)) throw new IllegalArgumentException();
for (InetAddress addr: InetAddress.getAllByName(host)) {
  if (isPrivateOrMetadata(addr)) throw new IllegalArgumentException();
}
Request req = new Request.Builder().url(url).build();
client.newBuilder().followRedirects(false).build().newCall(req).execute();
```

### Node.js (axios) Example
```js
import dns from 'node:dns/promises';
import axios from 'axios';
const BLOCKED_CIDRS = [ '127.0.0.0/8', '10.0.0.0/8', '172.16.0.0/12', '192.168.0.0/16', '169.254.0.0/16' ];
const ALLOWED_HOSTS = ['api.example.com'];

function isPrivate(ip) { /* CIDR check implementation */ }

export async function safeFetch(input) {
  const u = new URL(input);
  if (!['http:', 'https:'].includes(u.protocol)) throw new Error('scheme');
  if (![80, 443, ''].includes(u.port ? Number(u.port) : '')) throw new Error('port');
  if (!ALLOWED_HOSTS.includes(u.hostname)) throw new Error('host');
  const addrs = await dns.lookup(u.hostname, { all: true });
  if (addrs.some(a => isPrivate(a.address))) throw new Error('private');
  return axios.get(u.toString(), { maxRedirects: 0, timeout: 5000 });
}
```

### Python (requests) Example
```py
import socket, ipaddress, requests
ALLOWED = { 'api.example.com' }
BLOCKED_CIDRS = [ipaddress.ip_network(n) for n in (
  '127.0.0.0/8','10.0.0.0/8','172.16.0.0/12','192.168.0.0/16','169.254.0.0/16','::1/128','fc00::/7'
)]

def is_blocked(host):
    for r in socket.getaddrinfo(host, None):
        ip = ipaddress.ip_address(r[4][0])
        if any(ip in n for n in BLOCKED_CIDRS):
            return True
    return False

def safe_get(url):
    u = requests.utils.urlparse(url)
    if u.scheme not in ('http','https'):
        raise ValueError('scheme')
    if u.port and u.port not in (80, 443):
        raise ValueError('port')
    if u.username or u.password:
        raise ValueError('userinfo')
    if u.hostname not in ALLOWED or is_blocked(u.hostname):
        raise ValueError('host')
    return requests.get(url, timeout=5, allow_redirects=False)
```

---

## Infrastructure Hardening
- Egress filtering: deny-by-default; allow only required destinations from app subnets
- Network segmentation: separate web, app, data, and admin networks; no RFC1918 reachability from web tier
- Metadata protection:
  - AWS: enforce IMDSv2 and hop limit=1; block 169.254.169.254 via security groups/NACL if not needed
  - GCP: require Metadata-Flavor header, disable legacy endpoints where possible
  - Azure: require Metadata:true header; apply NSG rules
- Block dangerous protocols at proxies (gopher, file, ftp, dict)
- Proxy outbound traffic through a controlled egress proxy with URL allowlist
- DNS controls: prevent DNS rebinding via split-horizon DNS and short TTLs for internal names

---

## Monitoring and Detection
- Log all server-side outbound requests (destination host, IP, port, X-Forwarded-For)
- Alert on requests to private/link-local ranges and metadata IPs
- Detect unusual protocols or non-standard ports
- Use eBPF/NIDS to detect internal lateral scans from app hosts
- Canary endpoints: instrument a fake internal service and alert on access

---

## Verification and Regression Tests
- Unit tests for URL validator covering schemes, ports, host allowlist, CIDR checks, redirects
- Integration tests making requests to known-blocked addresses (127.0.0.1, 169.254.169.254)
- Negative tests for gopher://, file://, dict:// scheme rejection
- Penetration test checklist mirrors Payloads.md coverage
- Add SAST/DAST rules to flag unsafe fetches without validators

---

## 2026 Modern & AI-Assisted Defensive Updates
- **Socket-Level IP Validation (DNS Rebinding Elimination)**: Validating DNS before calling `fetch()`/HTTP client is vulnerable to Time-of-Check to Time-of-Use (TOCTOU) DNS rebinding. Always validate the resolved IP address inside a custom socket dialer/connector hook (e.g., Go `net.Dialer.Control`, Python `urllib3` custom connection pool, or Node.js `undici` custom `lookup`/`connect` dispatcher) so the exact IP connected to is verified against blocked CIDRs (`127.0.0.0/8`, `10.0.0.0/8`, `169.254.0.0/16`, `fc00::/7`, IPv4-mapped IPv6 `::ffff:0:0/96`).
- **Dedicated Egress Proxy (Smokescreen / Envoy)**: Route all user-triggered outbound HTTP requests (webhooks, URL previews, OIDC discovery, AI tool/plugin fetches) through a hardened forward egress proxy located in an isolated DMZ subnet with zero routes to internal VPCs or cloud metadata endpoints.
- **AI Agent & RAG Fetch Hardening**: Modern AI-assisted features that fetch user-supplied URLs or headless-browser previews must run in isolated egress-filtered sandboxes and enforce strict `Content-Type` and response byte-size limits.
- **AI-Assisted Sink Discovery in CI**: Use AI-assisted taint analysis in CI/CD to identify indirect SSRF sinks (headless Chromium/Playwright PDF generators, SVG/XML external entity processors, image/video transcoders, webhook testers) and enforce usage of a single centralized, socket-hardened HTTP client wrapper.

---

## References
- OWASP SSRF Prevention Cheat Sheet
- PortSwigger SSRF Academy
- Cloud provider IMDS documentation (AWS, GCP, Azure)
