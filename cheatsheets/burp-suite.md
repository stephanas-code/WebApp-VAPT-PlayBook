# 🛠️ Modern Burp Suite Pro & Community PlayBook (2026 Edition)

A step-by-step, hands-on beginner's guide to opening Burp Suite, setting up your environment for testing a target domain (e.g. `target.com`), capturing your first request, and applying modern workflows.

---

## 📑 Table of Contents
1. [Day 0: Launching Burp Suite for the First Time](#1-day-0-launching-burp-suite-for-the-first-time)
2. [Setting Up Your Target Scope (`target.com`)](#2-setting-up-your-target-scope-targetcom)
3. [Connecting Your Browser & Capturing Your First Request](#3-connecting-your-browser--capturing-your-first-request)
4. [Mastering the Core Tabs (Proxy, Repeater, Intruder, Comparer)](#4-mastering-the-core-tabs-proxy-repeater-intruder-comparer)
5. [Essential BApp Store Extensions Matrix](#5-essential-bapp-store-extensions-matrix)
6. [Modern HTTP/2 & HTTP/3 Protocol Testing](#6-modern-http2--http3-protocol-testing)
7. [Single-Packet Attack & Race Conditions (Turbo Intruder)](#7-single-packet-attack--race-conditions-turbo-intruder)
8. [API & Single Page App (SPA) Testing](#8-api--single-page-app-spa-testing)
9. [Automated Macro / Session Handling Rules](#9-automated-macro--session-handling-rules)
10. [Match and Replace Quality-of-Life Rules](#10-match-and-replace-quality-of-life-rules)
11. [AI & Modern Assistant Integration](#11-ai--modern-assistant-integration)

---

## 1. Day 0: Launching Burp Suite for the First Time

Whether you are on Windows, macOS, or Kali Linux:

1. **Launch Burp Suite** (Community or Professional).
2. **Startup Wizard**:
   - **Project file**: 
     - *Community Edition*: Select **Temporary project in memory** → click **Next**.
     - *Professional Edition*: Select **New project on disk**, name it (e.g. `audit-target.com.burp`), and choose a folder.
   - **Configuration**: Select **Use Burp defaults** → click **Start Burp**.
3. You will land on the **Dashboard** tab showing the event log and task panel.

---

## 2. Setting Up Your Target Scope (`target.com`)

> 💡 **Why Scope Matters**: Setting scope ensures you **only intercept and test your authorized target**, ignoring background traffic, Windows telemetry, analytics scripts, and third-party CDNs.

### Step-by-Step Beginner Scope Setup:

1. Go to the **Target** tab at the top → select **Scope settings**.
2. Check the box for **Use advanced scope control**.
3. Under **Include in scope**, click **Add**:
   - **Prefix**: *(leave blank)*
   - **Protocol**: `Any`
   - **Host or IP range**: `.*\.target\.com$` *(or `target\.com`)*
   - **Port**: *(leave blank)*
   - **File**: `^/.*`
   - Click **OK**.
4. Under **Exclude from scope**, add common noise domains so they don't fill your history:
   - Click **Add** → Host: `.*(google-analytics\.com|googletagmanager\.com|facebook\.com|sentry\.io|datadoghq\.com).*` → **OK**.
   - Click **Add** → File: `.*\.(png|jpg|jpeg|gif|css|woff|woff2|svg|ico)$` → **OK**.
5. **Clean Your HTTP History View**:
   - Click the **Proxy** tab → **HTTP history**.
   - Click the **Filter bar** (the gray bar near the top: *Filter: Showing all items*).
   - Check **Show only in-scope items**.
   - Check **Hide items without responses**.
   - Click anywhere outside the popup to apply the filter. Now, only `target.com` requests will be visible!

---

## 3. Connecting Your Browser & Capturing Your First Request

The easiest, error-free method in modern Burp Suite is the **Embedded Chromium Browser** (no manual proxy settings or certificate imports required).

### Option A: Using Burp's Embedded Browser (Recommended for Beginners)
1. Go to the **Proxy** tab → select **Intercept**.
2. Verify whether the button says **Intercept is on** or **Intercept is off**:
   - Toggle it to **Intercept is off** (this lets traffic flow through smoothly while logging to HTTP History).
3. Click the **Open Browser** button.
4. A clean Chromium browser window opens. It is pre-configured with Burp's proxy listener (`127.0.0.1:8080`) and trusted CA certificate.
5. In the address bar, type `https://target.com` and hit Enter.
6. Return to Burp Suite → click **Proxy** → **HTTP history**. You will see all HTTP requests and responses made to `target.com`!

### Option B: Using Your Own Browser (Firefox / Chrome)
If you prefer your daily driver:
1. In your browser settings, set HTTP/HTTPS proxy to `127.0.0.1` port `8080`.
2. Browse to `http://burp` in the address bar → click the top-right button **CA Certificate** to download `cacert.der`.
3. In browser settings, search for **Certificates** → **View Certificates** → **Authorities** tab → click **Import**.
4. Select `cacert.der` and check **Trust this CA to identify websites** → click **OK**.

---

## 4. Mastering the Core Tabs (Proxy, Repeater, Intruder, Comparer)

### 1) Proxy (Inspect & Tamper Live Traffic)
- **Intercept On**: Pauses requests before they leave your computer. You can edit parameters (e.g. `role=user` to `role=admin`) and click **Forward**, or click **Drop** to discard.
- **HTTP History**: A complete log of every request sent and received. Click any request to view its raw headers, parameters, and response body.

### 2) Repeater (Modify & Resend)
- In **HTTP history**, right-click any interesting request (e.g. `POST /api/login` or `GET /user/profile`) → click **Send to Repeater** (or press `Ctrl+R`).
- Switch to the **Repeater** tab:
  - Modify headers, parameters, or payloads in the left pane.
  - Click **Send**.
  - Review the immediate server response in the right pane without refreshing your browser.

### 3) Intruder (Targeted Automation & Fuzzing)
- In Repeater or Proxy, right-click a request → **Send to Intruder** (or press `Ctrl+I`).
- Under the **Positions** tab:
  - Click **Clear §** to remove default markers.
  - Highlight the specific parameter value you want to test (e.g. `id=§1§`).
  - Click **Add §** around the parameter.
- Under the **Payloads** tab:
  - Select **Payload type** (e.g. *Numbers*, *Simple list*, or *Brute forcer*).
  - Load your test list.
- Click **Start attack** in the top-right corner to view response statuses and lengths.

### 4) Comparer (Visual Diffing)
- Right-click two different requests or responses in Proxy or Repeater → select **Send to Comparer**.
- Switch to the **Comparer** tab → click **Words** or **Bytes** to highlight exact textual differences (useful for finding blind injection or authorization differences).

---

## 5. Essential BApp Store Extensions Matrix

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

## 6. Modern HTTP/2 & HTTP/3 Protocol Testing

Burp Suite supports native HTTP/2 with full request smuggling and frame inspection capabilities.

### Key Workflows:
- **HTTP/2 Request Smuggling (H2.CL / H2.TE / H2 Downgrading)**:
  - In Repeater, observe the protocol toggle in the inspector panel (**HTTP/1.1** vs **HTTP/2**).
  - Open the **Inspector** → **Request Attributes** and inspect pseudo-headers (`:method`, `:path`, `:authority`, `:scheme`).
  - Test CRLF injection inside pseudo-headers (e.g. `:path: /test\r\nTransfer-Encoding: chunked`).
- **HTTP/2 Downgrade Detection**: When reverse proxies translate frontend HTTP/2 into backend HTTP/1.1, test for hidden header smuggling or header name case-sensitivity discrepancies.

---

## 7. Single-Packet Attack & Race Conditions (Turbo Intruder)

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

## 8. API & Single Page App (SPA) Testing

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

## 9. Automated Macro / Session Handling Rules

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

## 10. Match and Replace Quality-of-Life Rules

Configure these in **Proxy** → **Proxy settings** → **Match and replace** to streamline everyday testing:

| Type | Match | Replace | Purpose |
| :--- | :--- | :--- | :--- |
| **Request header** | `^User-Agent:.*` | `User-Agent: Mozilla/5.0 Security-Audit-Team` | Identifies your traffic in client server logs. |
| **Request header** | *(empty)* | `X-Forwarded-For: 127.0.0.1` | Checks for naive internal IP trust or bypasses. |
| **Response body** | `disabled="disabled"` | *(empty)* | Re-enables disabled HTML submit buttons or form inputs. |
| **Response body** | `type="password"` | `type="text"` | Unmasks password fields in browser for debugging. |
| **Request header** | `Accept-Encoding: gzip, deflate, br` | `Accept-Encoding: identity` | Prevents server compression for easier inspection in raw logs. |

---

## 11. AI & Modern Assistant Integration

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
