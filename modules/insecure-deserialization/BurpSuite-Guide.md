# Insecure Deserialization - Burp Suite Guide

## Objectives
Use Burp Suite to detect and exploit insecure deserialization, then validate impact safely.

## Setup
- Proxy your browser through Burp. Import target CA certificate.
- Enable the following Burp features:
  - Logger, Proxy history, Intruder, Repeater
  - Extensions (BApp Store): Java Deserialization Scanner, .NET ViewState Viewer, JSON Web Tokens, Collaborator Everywhere

## Recon in Burp
- Look for suspicious parameters in Proxy history:
  - Cookies: session, cart, profile, user, prefs
  - Headers: X-Serialized-Data, X-Object, __VIEWSTATE
  - Body: hidden fields, base64/binary blobs, fields named data/object/state
- Indicators:
  - Java: aced0005 magic bytes (application/x-java-serialized-object)
  - PHP: O: / a: / s: / i: prefixes, PHAR paths
  - .NET: Large __VIEWSTATE, EventValidation fields
  - Python: base64 blobs recurring in APIs

## Passive Checks
- Add match-and-replace or Burp Search for serialization signatures (aced0005, O:, pickle, phar://).
- Use Collaborator Everywhere to detect OOB interactions during testing.

## Active Testing Workflow
1. Send candidate requests to Repeater.
2. Attempt benign payload injection for the suspected tech:
   - Java: base64 of ysoserial CommonsCollections6 with harmless command (whoami)
   - .NET: ysoserial.net BinaryFormatter payload; for ViewState, use signing/encryption if applicable
   - PHP: serialized object with safe __wakeup side-effect
   - Python: pickle payload executing harmless command
3. Observe responses, timing, and Collaborator DNS/HTTP callbacks.
4. If app crashes or times out, try lower-impact probes (file read, arithmetic) and confirm safely.

## Using Intruder
- Positions: mark the serialized value (cookie/header/body field).
- Payloads:
  - Paste base64-gzip variants, truncated prefixes/suffixes, or gadget permutations
  - Use payload processing: base64 encode, gzip, URL-encode, append/prepend
- Grep - Match:
  - Error strings: InvalidClassException, BadPaddingException, TypeNameHandling, unserialize(): Error, pickle.UnpicklingError
  - OOB interaction confirmation tokens

## Extensions
- Java Deserialization Scanner: passive and active probes, highlights sinks (ObjectInputStream).
- .NET ViewState Viewer: decode/inspect, verify MAC/encryption.
- Additional: Logger++ for better search, Collaborator Client for polling.

## 2026 Modern & AI/ML Deserialization Testing Workflows
- **FastAPI / Pydantic / Node.js Prototype Pollution Deserialization**:
  - In Repeater, inspect JSON endpoints accepting recursive objects.
  - Test prototype pollution injection into JSON parsers (`__proto__`, `constructor.prototype`):
    ```json
    {
      "__proto__": {
        "isAdmin": true,
        "status": "authenticated"
      }
    }
    ```
- **ML Model & Pickle Deserialization in AI Pipelines**:
  - Many modern web apps allow uploading pre-trained models or weights (`.pkl`, `.pt`, `.bin`, `.joblib`).
  - Test if the backend executes `torch.load()` or `pickle.loads()` without `weights_only=True` by firing a harmless Collaborator DNS ping payload during model import.
- **AI-Assisted Gadget & Payload Triage**:
  - Use Burp Collaborator to safely confirm OOB DNS interactions without triggering destructive payloads.
  - Copy candidate error traces (e.g. Jackson polymorphic type handling errors, `System.Text.Json` exceptions) into an AI assistant to verify whether target classes match safe DTO boundaries or indicate vulnerable type deserialization.

## Validation and Evidence
- Favor OOB via Collaborator to prove code execution without dangerous commands.
- Capture request/response pairs and timestamps, Collaborator payload IDs.
- Note affected endpoints, parameters, required headers, and any auth context.

## Reporting Template Snippet
- Issue: Insecure Deserialization leading to [RCE/Privilege Escalation]
- Affected endpoint: [URL]
- Parameter: [name]
- Evidence: [response snippet, server log, OOB interaction]
- Impact: Potential RCE, data tampering, and full account takeover
- Remediation: Replace unsafe deserialization, type allowlists, integrity protection (see Remediation.md)

---
*Part of the Web Application VAPT Guide - Insecure Deserialization Module*
