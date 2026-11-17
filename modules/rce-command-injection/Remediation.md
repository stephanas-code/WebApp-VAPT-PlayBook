# Remediation Guide: OS Command Injection

## Overview
This document outlines defensive controls to prevent OS command injection (RCE) across languages and platforms. Apply defense-in-depth: strict input handling, safe APIs, least privilege, output encoding, and monitoring.

## Risk and Business Impact
- Risk rating: Critical (CVSS typically 9.0–10.0) due to potential full system compromise.
- Impact: Data theft/destruction, supply-chain compromise, lateral movement, ransomware, regulatory violations, and reputational damage.

## Secure Design Principles
- Avoid shell invocation entirely; prefer safe library calls for file/network/process tasks.
- Validate input with allowlists (positive validation) tied to business rules.
- Separate code and data; never concatenate user input into commands.
- Run services with least privilege, using sandboxing (chroot/containers/AppArmor/SELinux) and filesystem isolation.
- Enforce secure defaults and fail closed; deny ambiguous input.

## Language/Framework-Specific Guidance

### General (All Stacks)
- Do not pass user input to: sh -c, bash -c, cmd.exe, powershell.exe, system(), exec(), popen(), passthru(), Runtime.exec(String), ProcessBuilder with shell invocation, etc.
- If external process is absolutely required, use array-based APIs that bypass the shell and supply validated arguments only.
- Disable or restrict dangerous binaries via OS hardening (noexec mount, AppArmor profiles, Windows SRP/AppLocker/WDAC).
- Sanitize environment: clear PATH, IFS, LD_PRELOAD, LD_LIBRARY_PATH, TMPDIR, COMSPEC, SHELL.
- Quote arguments and avoid shell metacharacters if unavoidable. Enforce length limits and character class allowlists.

### Java
- Prefer native libraries; avoid Runtime.getRuntime().exec(String).
- If needed: use ProcessBuilder(List<String>) with fixed executable and validated args. Example:
  - Good: new ProcessBuilder("/usr/bin/convert", "-resize", validatedSize, inputFile, outputFile)
  - Bad: new ProcessBuilder("/bin/sh", "-c", userInput)
- Use Spring Validation / Jakarta Bean Validation for strong input constraints.

### .NET/C#
- Prefer library functions (System.IO, System.Net).
- If starting processes, use ProcessStartInfo with UseShellExecute=false and fixed FileName; treat Arguments as validated tokens, not free text.
- Enable Code Access Security equivalents and Windows Defender Application Control where applicable.

### Python
- Use subprocess.run([...], shell=False, check=True). Never shell=True with user input.
- For file/network ops use built-ins (os, pathlib, shutil, requests) rather than shelling out.
- Validate with pydantic/schemathesis or explicit regex/enum checks.

### Node.js
- Avoid child_process.exec/execFile with user input; prefer spawn/execFile with array args only and no shell.
- Disable shell option; validate using joi/zod/owasp validator.

### PHP
- Avoid system, exec, shell_exec, passthru, popen, backticks.
- Prefer language APIs; if needed use proc_open with descriptor arrays, no shell, and validated arguments.
- Configure disable_functions and open_basedir; use escapeshellarg only as a last resort with allowlists.

### Ruby
- Use Open3.capture3 or system with array form: system("/usr/bin/ls", "-la").
- Avoid backticks and system("cmd string").

## Input Validation Controls
- Use positive validation: regex/enum/range checks (e.g., ^[a-zA-Z0-9_.-]{1,32}$ for filenames).
- Map user selections to internal constants (e.g., {"small": "128x128", "medium": "512x512"}).
- Reject or normalize disallowed bytes: newlines, null bytes, metacharacters ;|&`$><(){}[]*?!\n\r\t.
- Normalize Unicode and enforce ASCII where appropriate to prevent homoglyph bypasses.

## Output Encoding and Context Handling
- If returning command output to users (even safe commands), HTML-encode before display to prevent XSS.
- Avoid reflecting raw error messages; log internally. Implement generic user-facing errors.

## Infrastructure Hardening
- Principle of least privilege: dedicated OS user, minimal file/dir permissions, read-only FS where possible.
- Network egress restriction: proxy-only, DNS allowlist. Block direct outbound traffic from app hosts.
- Secrets management: no secrets via env or command-line; use vaults/managed identities.
- Logging/Monitoring: centralize process execution logs, command lines, and odd timeouts; alert on suspicious binaries (sh, bash, nc, curl, powershell).
- Container hardening: non-root, drop capabilities (NET_RAW, SYS_ADMIN), read-only root FS, seccomp/apparmor profiles.

## Configuration Examples

### NGINX/Reverse Proxy
- Strip/deny suspicious characters in parameters using WAF or NGINX map and return 403.
- Enforce size limits on headers, query strings, and bodies.

### Apache
- Use ModSecurity CRS; enable rules for command injection and file upload anomalies.
- Disable CGI if not required; prefer FastCGI with carefully controlled environment.

### IIS/Windows
- Constrain PowerShell execution policy, WDAC/AppLocker to allowlist signed binaries.
- Remove legacy interpreters from PATH, disable cmd.exe access for app pool identity where feasible.

## Secure Coding Snippets

### Python (safe subprocess)
```python
import shlex, subprocess
allowed = {"list": ["-la"], "id": []}
action = user_input_action  # validated key
args = allowed.get(action)
if args is None:
    raise ValueError("Invalid action")
subprocess.run(["/bin/ls", *args], check=True)
```

### Java (ProcessBuilder)
```java
List<String> cmd = List.of("/usr/bin/convert", "-resize", size, inFile, outFile);
validateSize(size); validatePath(inFile); validatePath(outFile);
new ProcessBuilder(cmd).inheritIO().start();
```

### Node.js (spawn without shell)
```js
const { spawn } = require('child_process');
const cmd = spawn('/usr/bin/convert', ['-resize', size, inFile, outFile], { shell: false });
```

## Verification Checklist (QA/Security)
- No code paths concatenate user input into command strings.
- All external process calls use array args with shell disabled.
- Inputs are allowlisted, length-limited, and type-validated.
- App/service runs under least-privileged identity; containers hardened.
- Outbound network restricted; suspicious process executions alerting in place.
- Automated tests cover negative cases and injection attempts.

## 2026 Modern & AI-Assisted Defensive Updates
- **Distroless Images & Zero-Shell Runtime Environments**: Deploy production workloads in distroless container images (e.g., Google Distroless, Chainguard Images) that omit `/bin/sh`, `/bin/bash`, `curl`, `wget`, and package managers entirely, neutralizing shell-out vectors even if an unsafe API is invoked.
- **Argument-Injection Mitigation (`--` End-of-Options)**: Even when `shell=False` / `execve` array APIs are used, user-controlled arguments starting with `-` or `--` can trigger dangerous flags in target binaries (CWE-88 Argument Injection). Always validate that arguments do not start with `-` and insert the POSIX `--` end-of-options delimiter before positional arguments:
  ```python
  if user_filename.startswith("-"):
      raise ValueError("Leading hyphens are not permitted")
  subprocess.run(["/usr/bin/git", "diff", "--", user_filename], shell=False, check=True)
  ```
- **eBPF Runtime Enforcement & AI/Agentic Sandboxing**:
  - Enforce kernel-level process execution allowlists using eBPF LSM tools (Tetragon, Falco, Tracee) to block unexpected child processes spawned by web workers.
  - For AI-assisted or LLM-integrated features that execute code or system tools, never run commands in the host container; isolate execution inside ephemeral microVMs (Firecracker, gVisor, WebAssembly WASI sandboxes) with zero network egress and strict CPU/memory quotas.
- **AI-Assisted Taint Tracking in CI/CD**: Use AI-augmented SAST (CodeQL, Semgrep) to trace user-controlled sources into both shell sinks (`CWE-78`) and argument-vector sinks (`CWE-88`), auto-generating remediation PRs that replace shell calls with native language SDKs.

## References
- OWASP: Command Injection Cheat Sheet, Proactive Controls
- PortSwigger: OS Command Injection
- CWE-78 (OS Command Injection) & CWE-88 (Argument Injection)
- NIST 800-53/800-190 container guidance
