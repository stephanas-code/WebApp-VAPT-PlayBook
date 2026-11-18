# CSRF Testing with Burp Suite

## Overview
This guide covers how to use Burp Suite tools to identify and test Cross-Site Request Forgery (CSRF) vulnerabilities.

## Table of Contents
- [Configuration](#configuration)
- [Detection Methods](#detection-methods)
- [Testing Workflow](#testing-workflow)
- [Tools & Extensions](#tools--extensions)
- [Advanced Techniques](#advanced-techniques)

## Configuration

### Proxy Settings
1. Configure browser to use Burp Proxy (default: 127.0.0.1:8080)
2. Install Burp CA certificate in browser
3. Enable intercept in Proxy tab

### Essential Settings
```
Proxy > Options > Intercept Client Requests
- Enable "Intercept requests based on the following rules"
- Add rule for POST requests
```

## Detection Methods

### 1. Manual Token Analysis
**Steps:**
1. Navigate to target functionality (profile update, password change, etc.)
2. In Proxy > HTTP History, find the request
3. Right-click > Send to Repeater
4. Analyze request for CSRF tokens

**Look for:**
- Anti-CSRF tokens in parameters
- Tokens in custom headers
- SameSite cookie attributes
- Referer/Origin header validation

### 2. Using Burp Scanner
**Steps:**
1. Right-click on request > Do active scan
2. Check for CSRF issues in Target > Site map
3. Review Issue details > Request/Response

**Scanner will check for:**
- Missing anti-CSRF tokens
- Predictable token values
- Token validation flaws

### 3. Repeater Testing
**Test scenarios:**
```
1. Remove CSRF token parameter entirely
2. Use empty token value
3. Reuse old/expired tokens
4. Use token from different user session
5. Replace with invalid/random token
6. Test with different HTTP methods (POST to GET)
```

## Testing Workflow

### Step 1: Identify State-Changing Actions
Focus on requests that:
- Modify user data
- Change account settings
- Perform financial transactions
- Delete resources
- Create/update content

### Step 2: Capture Legitimate Request
```
1. Perform the action normally
2. Find request in HTTP History
3. Right-click > Send to Repeater
4. Note all parameters and headers
```

### Step 3: Test Token Validation

#### Test 1: Remove Token
```
1. In Repeater, delete the CSRF token parameter
2. Send request
3. Check if action still succeeds
```

#### Test 2: Modify Token
```
1. Change token value to random string
2. Send request
3. Observe response
```

#### Test 3: Reuse Token
```
1. Send same request multiple times
2. Check if token can be reused
```

#### Test 4: Cross-Session Token
```
1. Obtain token from different session
2. Use in current session request
3. Check if accepted
```

### Step 4: Method Manipulation
```
1. In Repeater, right-click > Change request method
2. Convert POST to GET
3. Check if request still processes
```

### Step 5: Create PoC
Use Burp's built-in CSRF PoC generator:
```
1. Right-click on vulnerable request
2. Engagement tools > Generate CSRF PoC
3. Test in browser
4. Modify as needed for bypass techniques
```

## Tools & Extensions

### Built-in Tools

#### 1. Repeater
- Manual token manipulation
- Quick testing of variations
- Session handling rules testing

#### 2. Intruder
Use for:
- Token brute forcing
- Testing multiple token variations
- Automated bypass attempts

**Attack Type:** Sniper
**Payload Position:** CSRF token value
```
POST /profile/update HTTP/1.1
Host: target.com
csrf_token=§random_value§
```

#### 3. Sequencer
Analyze token randomness:
```
1. Send to Sequencer
2. Configure token location
3. Start live capture
4. Analyze results for entropy
```

### Recommended Extensions

#### 1. CSRF Scanner
```
Extender > BApp Store > CSRF Scanner
```
- Automated CSRF detection
- Token analysis
- PoC generation

#### 2. AuthMatrix
```
Extender > BApp Store > AuthMatrix
```
- Test cross-user CSRF
- Session management testing
- Role-based access control

#### 3. Token Extractor
```
Extender > BApp Store > Custom Parameter Handler
```
- Extract tokens automatically
- Handle dynamic tokens in automated scans

## Advanced Techniques

### 1. Content-Type Manipulation

**Test changing Content-Type:**
```http
# Original
POST /api/profile HTTP/1.1
Content-Type: application/json
{"email": "new@email.com", "csrf": "token123"}

# Modified
POST /api/profile HTTP/1.1
Content-Type: text/plain
{"email": "new@email.com", "csrf": "token123"}
```

**Steps in Burp:**
1. Send to Repeater
2. Modify Content-Type header
3. Test various types (application/x-www-form-urlencoded, text/plain, multipart/form-data)

### 2. Subdomain Bypass

**Test with match rules:**
```
Proxy > Options > Match and Replace
Add rule to modify Origin/Referer headers
```

**Example:**
```
Type: Request header
Match: Origin: https://app.target.com
Replace: Origin: https://evil.target.com
```

### 3. Token Leakage

**Check for token exposure:**
```
Target > Site map > Filter > Show only parameterized requests
```

Look for tokens in:
- URL parameters
- Referer headers
- Server responses
- JavaScript files

### 4. Session Management Testing

**Macro configuration:**
```
Project options > Sessions > Macros > Add
1. Record token extraction from page
2. Configure token handling rules
3. Test if CSRF tied to session
```

### 5. Double Submit Cookie Bypass

**Testing approach:**
```
1. Identify double-submit cookie implementation
2. Test if cookie can be set via injection
3. Test subdomain cookie setting
```

**In Repeater:**
```http
POST /profile/update HTTP/1.1
Host: target.com
Cookie: PHPSESSID=abc123; csrf_token=token123
csrf_token=token123

# Test with modified cookie
Cookie: PHPSESSID=abc123; csrf_token=attack456
csrf_token=attack456
```

## Testing Checklist

### Basic Tests
- [ ] Remove CSRF token parameter
- [ ] Send empty token value
- [ ] Use random/invalid token
- [ ] Reuse old token
- [ ] Use token from another user
- [ ] Change request method (POST to GET)
- [ ] Remove Referer header
- [ ] Modify Origin header

### Advanced Tests
- [ ] Test token entropy with Sequencer
- [ ] Check for token leakage
- [ ] Test Content-Type changes
- [ ] Test subdomain cookie setting
- [ ] Verify SameSite cookie attributes
- [ ] Test CORS configuration
- [ ] Check for clickjacking combination
- [ ] Test with different HTTP methods

## Common Issues and Solutions

### Issue: Token Updates on Each Request
**Solution:**
Use Session Handling Rules:
```
Project options > Sessions > Session Handling Rules > Add
1. Add "Run a macro" action
2. Configure macro to extract new token
3. Set scope to target requests
```

### Issue: JavaScript-Generated Tokens
**Solution:**
1. Analyze JavaScript files for token generation
2. Use Burp's JavaScript analysis tools
3. Extract token from DOM before request

### Issue: Custom Headers & JSON Endpoints
**Solution:**
```
Intercept request and check for:
- X-CSRF-Token / X-XSRF-TOKEN / X-Requested-With
- Content-Type: application/json -> Test if the backend accepts Content-Type: text/plain
  or application/x-www-form-urlencoded to permit simple cross-site requests.
```

## 2026 Modern & AI-Assisted CSRF Workflows
- **Fetch Metadata Header Tampering (`Sec-Fetch-*`)**:
  - In Repeater, test how the server responds when modifying `Sec-Fetch-Site: cross-site` to `same-origin`, or when stripping `Sec-Fetch-*` headers entirely.
  - Verify if backend endpoints rely solely on standard headers or correctly enforce W3C Resource Isolation Policies.
- **Next.js Server Actions & Remix Route Auditing**:
  - Inspect POST requests containing `Next-Action` headers. Test replaying these actions in Repeater with origin headers stripped or pointing to an external domain.
- **AI-Assisted PoC Generation**:
  - In Burp Pro, right-click the target request → **Engagement tools** → **Generate CSRF PoC**.
  - For complex JSON/REST endpoints, copy the request as cURL and prompt an AI assistant: *"Generate an automated HTML/JS proof-of-concept testing cross-site JSON invocation via navigator.sendBeacon and fetch with mode: 'no-cors' for this request."*

## Best Practices

1. **Test systematically** - Follow workflow for each state-changing action
2. **Document findings** - Note which protections exist and which are missing
3. **Generate PoCs** - Always create working proof of concept
4. **Test all methods** - Don't just test token removal
5. **Check mobile APIs** - Often have weaker CSRF protection
6. **Consider context** - Severity depends on impacted functionality
7. **Test combinations** - CSRF + XSS, CSRF + Clickjacking

## References

- [PortSwigger Web Security Academy - CSRF](https://portswigger.net/web-security/csrf)
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
- [OWASP CSRF Guide](https://owasp.org/www-community/attacks/csrf)

## Related Files
- [README.md](README.md) - CSRF overview and theory
- [Payloads.md](Payloads.md) - CSRF payload examples
- [Reproduce.md](Reproduce.md) - Step-by-step reproduction guides
- [Remediation.md](Remediation.md) - How to fix CSRF vulnerabilities
