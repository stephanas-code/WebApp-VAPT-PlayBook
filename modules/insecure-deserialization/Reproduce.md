# Insecure Deserialization - Reproduction Guide

## Goal
Reproduce and verify insecure deserialization across common stacks, demonstrating impact (RCE/privilege escalation) in a safe environment.

## Test Environment Prereqs
- Isolated lab or containerized targets; never test on production.
- Burp Suite (Community or Pro), curl, Python 3, Java 8+, .NET SDK, PHP, Docker.
- Sample vulnerable apps (pick relevant):
  - Java: ysoserial payloads; DVJA or custom servlet that reads Java serialized objects
  - .NET: vulnerable ViewState app or demo BinaryFormatter endpoint
  - PHP: endpoint calling unserialize() on cookie/body; PHAR processing route
  - Python: API using pickle.loads or yaml.load

## Discovery Checklist
- Identify parameters likely to hold serialized data:
  - Cookies: session, cart, user, prefs
  - Headers: X-Serialized-Data, X-Object, X-ViewState
  - Body: hidden fields, JSON fields named data/object/state
- Indicators:
  - Java magic bytes: ac ed 00 05
  - PHP serialized strings start with: O:, a:, s:, i:
  - Pickle opcodes: 80 03 or protocol markers
  - .NET ViewState (__VIEWSTATE hidden input)

## Java Walkthrough
1. Intercept request with serialized object (or endpoint that expects it).
2. Generate gadget payload with ysoserial:
   - java -jar ysoserial.jar CommonsCollections6 "whoami" > payload.ser
   - base64 payload: base64 -w0 payload.ser
3. Replace parameter or header with base64 of payload. Example using Burp:
   - Right-click -> Extensions -> Java Deserialization Scanner (if installed) -> Generate payload
4. Send request and observe:
   - 200 with command output or timing/side effects (DNS callback)
   - Server error/log entry indicating deserialization

## .NET Walkthrough (BinaryFormatter / ViewState)
1. Identify ViewState in pages; note ViewState MAC and EventValidation.
2. If MAC disabled or machine keys known, craft payload with ysoserial.net:
   - ysoserial.exe -f BinaryFormatter -g ObjectDataProvider -o base64 -c "calc.exe"
3. For ViewState, use tools to sign/encrypt with keys; submit via __VIEWSTATE.
4. Validate via process spawn, file drop, or DNS pingback.

## PHP Walkthrough
1. Locate unserialize() usage (source review) or PHP serialized values in cookies/body.
2. Craft exploit class matching available gadget chain (magic methods __wakeup/__destruct).
3. Encode object: php -r 'echo base64_encode("O:4:\"User\":1:{s:8:\"username\";s:5:\"admin\";}");'
4. Replace cookie/body value and resend. Confirm effect via behavior change or file read.
5. For PHAR, upload crafted phar-in-image and hit file operation using phar:// wrapper.

## Python Walkthrough
1. Find endpoints calling pickle.loads or yaml.load.
2. Build pickle payload with a benign command (whoami/id) and base64-encode.
3. Replace parameter and send. Confirm via logs or response differences.

## Safety and Validation
- Prefer out-of-band (OOB) verification: Burp Collaborator or DNS callback to avoid damaging commands.
- Start with benign commands, then escalate impact proofs as approved.
- Capture evidence: request/response pairs, server logs, screenshots.

## Reporting
- Describe affected endpoints/parameters and the data flow to deserialization sink.
- Provide proof-of-concept payloads, exact steps, and observed impact.
- Map to risk: potential RCE/privilege escalation; business impact on confidentiality, integrity, availability.
- Recommend remediations: replace with safe serialization, type allowlists, integrity protection.

---
*Part of the Web Application VAPT Guide - Insecure Deserialization Module*
