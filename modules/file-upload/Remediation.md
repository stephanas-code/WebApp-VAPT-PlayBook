# File Upload Vulnerabilities - Remediation

This guide outlines secure coding, validation, and hardening strategies to prevent file upload vulnerabilities.

## Security Objectives
- Only allow intended, safe file types
- Ensure uploaded files are never executed as code
- Store uploads securely with access controls and scanning
- Validate both client- and server-side (server is authoritative)

## Design Principles
- Positive security model (allowlist) for extensions and MIME
- Verify file content (magic bytes), not just names/headers
- Store outside webroot; serve via indirect handlers
- Rename files to safe, random names; strip user-controlled paths
- Enforce strict size limits, quotas, and rate limiting
- Asynchronous malware scanning before exposure
- Defense in depth with WAF, CSP, and segregation of duties

## Server-side Validation Checklist
- Validate filename: remove paths, control chars, unicode RTL overrides
- Enforce allowlist extensions (e.g., .jpg, .png, .pdf) only
- Validate MIME using trusted libraries (magic bytes)
- Re-encode images using safe libraries (e.g., ImageMagick with policy)
- Reject archives unless explicitly required; if required, sandbox extraction
- Validate dimensions, frame count, and complexity for media
- Enforce max size per type and per request
- Sanitize metadata (EXIF) where applicable

## Safe Storage and Delivery
- Store uploads outside the document root (e.g., /data/uploads)
- Serve via application handler that sets Content-Disposition: attachment
- Set strict permissions: owner-only write, no execute (chmod 0640), separate user/group
- Disable script execution in upload directories
  - Apache: .htaccess
    - php_flag engine off
    - RemoveHandler .php .phtml .php3 .php5
    - RemoveType .php .phtml
  - Nginx: location ^~ /uploads/ { default_type application/octet-stream; add_header X-Content-Type-Options nosniff; try_files $uri =404; }
- Use separate domain/subdomain for static uploads (no cookies), apply CSP

## Example Secure Implementation (Node.js/Express)
```js
import multer from 'multer';
import crypto from 'crypto';
import { promises as fs } from 'fs';
import fileType from 'file-type';

const storage = multer.memoryStorage();
const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
});

const ALLOW = {
  'image/jpeg': 'jpg',
  'image/png': 'png',
  'application/pdf': 'pdf',
};

app.post('/upload', upload.single('file'), async (req, res) => {
  if (!req.file) return res.status(400).send('Missing');
  const type = await fileType.fromBuffer(req.file.buffer);
  if (!type || !ALLOW[type.mime]) return res.status(415).send('Unsupported');
  const name = crypto.randomBytes(16).toString('hex') + '.' + ALLOW[type.mime];
  const path = `/data/uploads/${name}`;
  await fs.writeFile(path, req.file.buffer, { mode: 0o640 });
  // queue for AV scan before exposing download URL
  res.json({ id: name });
});
```

## Example Secure Implementation (PHP)
```php
$allow = [ 'image/jpeg' => 'jpg', 'image/png' => 'png', 'application/pdf' => 'pdf' ];
if (!isset($_FILES['file']) || $_FILES['file']['error'] !== UPLOAD_ERR_OK) { http_response_code(400); exit; }
$buf = file_get_contents($_FILES['file']['tmp_name']);
$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime = $finfo->buffer($buf);
if (!isset($allow[$mime])) { http_response_code(415); exit; }
$fname = bin2hex(random_bytes(16)) . '.' . $allow[$mime];
$dest = '/data/uploads/' . $fname;
if (!move_uploaded_file($_FILES['file']['tmp_name'], $dest)) { http_response_code(500); exit; }
chmod($dest, 0640);
```

## Framework and Platform Hardening
- Apache/Nginx/IIS: disable execution in upload paths
- PHP: disable dangerous functions (exec, system) where feasible; open_basedir
- Java/Tomcat: isolate WAR contexts; avoid writing under webapp dirs
- .NET/IIS: Request Filtering allowlists; remove script handlers from upload dirs
- Containers: run as non-root; mount uploads with noexec,nodev,nosuid

## Malware and Content Scanning
- Use ClamAV or commercial scanners via async queue
- Quarantine and review suspicious files
- Verify images by decoding and re-encoding (strip ICC/EXIF) to defeat polyglots

## Business Controls
- Log all upload attempts (user, IP, name, size, MIME)
- Alert on anomalies (excess failures, unusual types)
- Periodic review/cleanup of orphaned uploads
- DLP for sensitive content exfiltration

## Testing and Verification
- Incorporate tests from this module into CI (safe fixtures)
- Negative tests: ensure .php/.jsp rejected, traversal neutralized
- Security headers on delivery: X-Content-Type-Options: nosniff; Content-Disposition: attachment; Content-Security-Policy for preview endpoints

## References
- OWASP File Upload Cheat Sheet
- CWE-434 (Unrestricted Upload)
- PortSwigger research on polyglots and MIME sniffing

## 2026 Modern & AI-Assisted Defensive Updates
- **Direct-to-Object-Storage Presigned Uploads**: Bypass the application server entirely for binary transfers by issuing short-lived, condition-constrained presigned URLs (e.g., S3/GCS POST policies enforcing exact `Content-Type`, `content-length-range`, and SHA-256 checksums) into an isolated quarantine bucket.
- **Sandboxed Content Disarm and Reconstruction (CDR)**: Process untrusted media and documents inside ephemeral WebAssembly (Wasm) sandboxes or rootless microVMs (Firecracker/gVisor) using memory-safe parsers (e.g., Rust `image` crate or `libvips`) to strip metadata, active scripts, embedded macros, and polyglot trailers before copying clean assets to a CDN domain.
- **AI-Assisted Upload Pipeline Verification**:
  - Use AI-assisted code review in CI to detect direct filesystem writes (`fs.writeFile`, `move_uploaded_file`, `shutil.copyfileobj`) that use user-supplied `originalname` or lack magic-byte verification.
  - Integrate automated static/dynamic polyglot and archive-bomb (Zip Slip / decompression ratio) regression fixtures into CI pipelines to ensure parser sandboxes enforce strict memory, CPU, and path-normalization limits.

## Checklist (Go/No-Go)
- [ ] Allowlist and magic-byte validation implemented
- [ ] Upload directory outside webroot and non-executable (or isolated object storage bucket)
- [ ] Files renamed randomly; metadata sanitized via CDR/re-encoding
- [ ] Size, type, decompression ratio, and rate limits enforced
- [ ] AV scanning before exposure
- [ ] Robust logging and alerting in place
