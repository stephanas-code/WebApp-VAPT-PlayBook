# 🛠️ Modern Burp Suite Pro & Community PlayBook (2026 Edition)

A hands-on, fast-reference guide designed for web application security assessments, API penetration testing, and bug bounty workflows using modern Burp Suite features, BApp extensions, and automation.

---

## 📑 Table of Contents
1. [Target Scope & Noise Reduction](#1-target-scope--noise-reduction)
2. [Modern Browser Integration & TLS Handling](#2-modern-browser-integration--tls-handling)
3. [Essential BApp Store Extensions Matrix](#3-essential-bapp-store-extensions-matrix)
4. [Modern HTTP/2 & HTTP/3 Protocol Testing](#4-modern-http2--http3-protocol-testing)
5. [Single-Packet Attack & Race Conditions (Turbo Intruder)](#5-single-packet-attack--race-conditions-turbo-intruder)
6. [API & Single Page App (SPA) Testing](#6-api--single-page-app-spa-testing)
7. [Automated Macro / Session Handling Rules](#7-automated-macro--session-handling-rules)
8. [Match and Replace Quality-of-Life Rules](#8-match-and-replace-quality-of-life-rules)
9. [AI & Modern Assistant Integration](#9-ai--modern-assistant-integration)

---

## 1. Target Scope & Noise Reduction

Don't let third-party trackers, analytics, and CDNs clutter your HTTP history.

### Step-by-Step Advanced Scope Setup:
1. Navigate to **Target** → **Scope settings**.
2. Check **Use advanced scope control**.
3. Under **Include in scope**, click **Add**:
   - Protocol: `Any`
   - Host or IP range: `.*\.targetdomain\.com$`
   - File: `^/.*`
4. Under **Exclude from scope**, add regex rules for noise:
   - `.*(google-analytics\.com|googletagmanager\.com|facebook\.com|sentry\.io|datadoghq\.com).*`
   - `.*\.(png|jpg|jpeg|gif|css|woff|woff2|svg|ico)$`
5. In **Proxy** → **HTTP history**, click the **Filter bar** at the top:
   - Check **Show only in-scope items**.
   - Check **Hide items without responses**.

---

## 2. Modern Browser Integration & TLS Handling

Avoid system-wide proxy conflicts and browser certificate pinning issues by using Burp's embedded Chromium browser.

### Best Practices:
- **Embedded Browser**: Go to **Proxy** → **Intercept** → click **Open Browser**. This launches Chromium pre-configured with the Burp proxy and CA certificate installed.
- **Passkeys / WebAuthn**: Modern Chromium handles WebAuthn virtual authenticators under **DevTools** (`F12`) → **More tools** → **WebAuthn** to test hardware token flows through the proxy.
- **Upstream Proxy / Tor Routing**: If routing through corporate proxies or an egress SOCKS5 gateway:
  - Go to **Settings** → **Network** → **Connections** → **SOCKS Proxy** (`127.0.0.1:9050`).

---

## 3. Essential BApp Store Extensions Matrix

Install these extensions via **Extensions** → **BApp Store** to supercharge manual and semi-automated testing:

| Extension | Category | Primary Use Case & How to Implement |
| :--- | :--- | :--- |
| **Autorize** | Broken Access Control / IDOR | Automatically replays every request in low-privilege or unauthenticated session cookies to spot unauthorized access in real time. |
| **Turbo Intruder** | Race Conditions / High-Speed Fuzzing | Employs an asynchronous HTTP stack to fire thousands of requests per second and perform single-packet race condition attacks. |
| **JWT Editor** | Authentication / Token Security | Seamlessly decodes, tampers, resigns, and injects JWKs or tests `alg: none` / signature stripping directly in Repeater. |
| **Param Miner** | Cache Poisoning / Hidden Parameters | Guesses unkeyed query parameters, headers (`X-Forwarded-*`), and hidden cookies that trigger alternative code paths. |
| **Logger++** | Advanced Logging & Analysis | Provides deep multi-threaded filtering, regex searching, and export capabilities beyond the default Proxy History. |
| **Active Scan++** | Active Scanning Enhancement | Extends the active scanner to identify edge-case vulnerabilities, blind SSRF, and advanced template injections. |

---

## 4. Modern HTTP/2 & HTTP/3 Protocol Testing

Burp Suite Pro supports native HTTP/2 with full request smuggling and frame inspection capabilities.

### Key Workflows:
- **HTTP/2 Request Smuggling (H2.CL / H2.TE / H2 Downgrading)**:
  - In Repeater, observe the protocol toggle in the inspector panel (**HTTP/1.1** vs **HTTP/2**).
  - Open the **Inspector** → **Request Attributes** and inspect pseudo-headers (`:method`, `:path`, `:authority`, `:scheme`).
  - Test CRLF injection inside pseudo-headers (e.g. `:path: /test\r\nTransfer-Encoding: chunked`).
- **HTTP/2 Downgrade Detection**: When reverse proxies translate frontend HTTP/2 into backend HTTP/1.1, test for hidden header smuggling or header name case-sensitivity discrepancies.

---

## 5. Single-Packet Attack & Race Conditions (Turbo Intruder)

Traditional multi-threaded race condition tests suffer from network jitter. The **Single-Packet Attack** synchronizes requests inside the same TCP window or TLS frame.

### How to Run:
1. In Burp **Repeater** or **Proxy**, right-click the target request → **Extensions** → **Turbo Intruder** → **Send to turbo intruder**.
2. Select the template **`race-single-packet-attack.py`** from the dropdown.
3. Configure the concurrent connections and gate synchronization:
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           requestsPerConnection=100,
                           pipeline=False)

    # Queue 20 identical requests behind a synchronization gate
    for i in range(20):
        engine.queue(target.req, gate='race1')

    # Open the gate to release all requests simultaneously in a single packet/burst
    engine.openGate('race1')

def handleResponse(req, interesting):
    table.add(req)
```
4. Click **Start attack** and look for HTTP 200/201 responses showing double coupon redemptions, duplicate withdrawals, or state-machine overrides.

---

## 6. API & Single Page App (SPA) Testing

Modern Single Page Applications communicate over JSON APIs, GraphQL, and WebSockets.

### 1) OpenAPI / Swagger Ingestion:
- If a target exposes `/v3/api-docs`, `/swagger.json`, or `/openapi.yaml`:
  - Use the **OpenAPI Parser** extension or import directly via **Target** → **Site map** → right-click → **Import OpenAPI**.
  - All discovered endpoints and parameter schemas populate directly into your target tree for Repeater testing.

### 2) GraphQL Introspection & Mutation Testing:
- In Repeater, paste an introspection query:
  ```graphql
  {"query":"query IntrospectionQuery{__schema{queryType{name}mutationType{name}types{name kind fields{name}}}}"}
  ```
- If introspection is disabled, test field suggestion bypasses (e.g. sending invalid fields to observe verbose `Did you mean "fieldName"?` errors).

### 3) WebSocket Inspection:
- Open **Proxy** → **WebSockets history**.
- Inspect live bi-directional frames. Send frames to **WebSockets Repeater** to alter message payloads and test authorization on real-time event updates.

---

## 7. Automated Macro / Session Handling Rules

When testing endpoints that require fresh anti-CSRF tokens or short-lived bearer tokens that expire quickly:

### Setup Session Handling Rule:
1. Go to **Settings** → **Sessions** → **Session handling rules** → Click **Add**.
2. Under **Rule actions**, click **Add** → **Run a macro**.
3. Under **Select macro**, click **Add** and record the sequence of requests needed to fetch a fresh token (e.g. `GET /form` or `POST /auth/refresh`).
4. In the **Configure macro item**, select the response parameter containing the token (e.g. `csrf_token`) and define the parameter to replace in target requests.
5. In the **Scope** tab:
   - Select **Tools Scope**: Repeater, Intruder, Scanner.
   - Select **URL Scope**: Include in-scope items only.

---

## 8. Match and Replace Quality-of-Life Rules

Configure these in **Proxy** → **Proxy settings** → **Match and replace** to streamline everyday testing:

| Type | Match | Replace | Purpose |
| :--- | :--- | :--- | :--- |
| **Request header** | `^User-Agent:.*` | `User-Agent: Mozilla/5.0 Security-Audit-Team` | Identifies your traffic in client server logs. |
| **Request header** | *(empty)* | `X-Forwarded-For: 127.0.0.1` | Checks for naive internal IP trust or bypasses. |
| **Response body** | `disabled="disabled"` | *(empty)* | Re-enables disabled HTML submit buttons or form inputs. |
| **Response body** | `type="password"` | `type="text"` | Unmasks password fields in browser for debugging. |
| **Request header** | `Accept-Encoding: gzip, deflate, br` | `Accept-Encoding: identity` | Prevents server compression for easier inspection in raw logs. |

---

## 9. AI & Modern Assistant Integration

Modern security testing workflows in 2026 leverage AI assistants to accelerate triage and script creation:

1. **Automated Request-to-Code Conversion**:
   - Right-click any HTTP request in Burp → **Copy as curl command** or **Copy as Python script**.
   - Prompt your AI assistant: *"Analyze this HTTP request and write an asynchronous defensive regression test in Python using `httpx` to verify that `403 Forbidden` is returned when authorization cookies are stripped."*
2. **Dynamic Header & Payload Mutation Scripts**:
   - Generate custom Turbo Intruder scripts or custom Burp Montoya API extensions in minutes using AI to fuzz proprietary serialization formats.
3. **Response Analysis & Triage**:
   - Export verbose Burp Scanner XML/JSON outputs or Comparer diffs to synthesize vulnerability summaries and CVSS v4.0 vectors for professional reporting.

---

*Part of the [Web Application VAPT PlayBook](https://github.com/stephanas-code/WebApp-VAPT-PlayBook) by Stephanas Odogu.*
