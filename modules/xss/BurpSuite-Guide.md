# Cross-Site Scripting (XSS): Burp Suite Hands-on Guide

A practical, modern testing guide for discovering, testing, and confirming Cross-Site Scripting (Reflected, Stored, and DOM-based) using Burp Suite Professional and Community.

---

## 📑 Table of Contents
1. [Setup and Scope](#setup-and-scope)
2. [Reflected XSS Workflow](#reflected-xss-workflow)
3. [Stored XSS Workflow](#stored-xss-workflow)
4. [DOM XSS & Modern Frameworks (React, Vue, Angular)](#dom-xss--modern-frameworks)
5. [Automated Fuzzing with Intruder](#automated-fuzzing-with-intruder)
6. [Bypassing WAFs and Filters in Repeater](#bypassing-wafs-and-filters-in-repeater)
7. [2026 Modern & AI-Assisted Workflows](#2026-modern--ai-assisted-workflows)
8. [Evidence Collection & Reporting](#evidence-collection--reporting)

---

## Setup and Scope
- Proxy browser traffic through Burp (or use Burp's embedded Chromium browser via **Proxy** → **Open Browser**).
- Enable **Target** → **Scope settings** for your authorized domain to filter noise.
- Recommended Extensions: **DOM Invader** (built into Burp Chromium), **Logger++**, **Param Miner**.

---

## Reflected XSS Workflow
1. Identify all reflection points:
   - Submit canary strings like `sec7357` into search bars, filter inputs, and query parameters.
2. In **Proxy** → **HTTP history**, filter for your canary string in response bodies.
3. Send promising requests to **Repeater**:
   - Determine context: HTML body, attribute, inside `<script>` tag, or URL parameter.
   - Test character reflection: test if `<`, `>`, `"`, `'`, `/`, backticks, and curly braces `{}` are reflected unencoded.
   - Inject context-appropriate benign proof: `<svg onload=alert(1)>` or `"-alert(1)-"`.

---

## Stored XSS Workflow
1. Locate stateful input fields: user bios, profile names, comments, ticket descriptions, or avatar file names.
2. Inject a tagged canary (e.g. `audit<svg onload=confirm(1)>`) and submit the form.
3. Browse to all administrative or secondary viewing pages where the stored data is rendered to verify execution across different privilege contexts.

---

## DOM XSS & Modern Frameworks
Modern Single Page Applications (React, Next.js, Vue, Angular) frequently execute dynamic data via client-side JavaScript.

### Using DOM Invader (Burp Embedded Browser):
1. Launch Burp's embedded Chromium browser.
2. Open DevTools (`F12`) → select the **DOM Invader** tab.
3. Turn on **DOM Invader** and enable **Postmessage interception**.
4. Set a custom canary (e.g. `auditCanary`).
5. As you navigate the site, DOM Invader automatically traces sources (`location.search`, `location.hash`, `document.referrer`, `window.postMessage`) into dangerous sinks (`eval`, `innerHTML`, `document.write`, `setTimeout`).
6. Inspect reported sinks directly in DevTools to view the exact stack trace and unescaped input.

---

## Automated Fuzzing with Intruder
1. In Repeater, right-click the candidate request → **Send to Intruder**.
2. Mark payload positions around the reflected parameter:
   `GET /search?q=§test§ HTTP/1.1`
3. Load a curated XSS payload set or use the **BApp Store** XSS lists.
4. Add **Grep - Match** rules for the exact payload or `<script>` tags to identify reflections that survived server sanitization unescaped.

---

## Bypassing WAFs and Filters in Repeater
- Event handlers without spaces: `<svg/onload=alert(1)>`
- Case variation: `<sCrIpt>alert(1)</sCrIpt>`
- HTML entity encoding inside attributes: `<a href="javascript:&NewLine;alert(1)">Click</a>`
- Template literal evaluation: `${alert(1)}` inside ES6 backticks.

---

## 2026 Modern & AI-Assisted Workflows
- **W3C Trusted Types Verification**:
  - In Repeater, inspect response headers for `Content-Security-Policy: require-trusted-types-for 'script'`.
  - In Chromium console, verify whether the browser throws a `TypeError: This document requires 'TrustedHTML' assignment` when attempting to pass raw strings to DOM sinks.
- **AI-Assisted Context Analysis**:
  - Right-click the reflected request/response in Repeater, copy as cURL, and prompt an AI assistant: *"Analyze the reflection context in this HTML/JS response. Identify whether it sits inside a template literal, attribute, or raw text node, and suggest the exact minimal escape sequence needed to execute proof JavaScript without triggering generic WAF regexes."*

---

## Evidence Collection & Reporting
- Capture request/response pairs in Repeater showing unencoded payload reflection with `Content-Type: text/html`.
- Note if cookies lack `HttpOnly` or if CSP is missing/misconfigured (e.g. `unsafe-inline` present).
- Reference [remediation.md](remediation.md) for context-aware encoding and CSP policies.

---

*Part of the [Web Application VAPT PlayBook](https://github.com/stephanas-code/WebApp-VAPT-PlayBook) by Stephanas Odogu.*
