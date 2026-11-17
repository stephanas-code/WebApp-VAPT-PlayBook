# Vulnerability Report Template

## Executive Summary
[Provide a brief, non-technical overview of the vulnerability and its potential impact on the organization]

## Vulnerability Details

### Title
[Clear, concise vulnerability title]

### Severity
- **CVSS Score**: [X.X]
- **Rating**: [Critical/High/Medium/Low]
- **Vector String**: CVSS:3.1/AV:X/AC:X/PR:X/UI:X/S:X/C:X/I:X/A:X

### Description
[Detailed technical description of the vulnerability, including:
- What the vulnerability is
- Where it was found
- How it works
- Why it's a security concern]

### Affected Systems
- **Application**: [Application name and version]
- **URL/Endpoint**: [Specific location]
- **Parameters**: [Affected parameters/functions]
- **Environment**: [Production/Staging/Development]

## Proof of Concept (PoC)

### Prerequisites
[List any requirements needed to reproduce the vulnerability]

### Steps to Reproduce
1. [Detailed step-by-step instructions]
2. [Include exact commands, requests, or actions]
3. [Be specific enough for others to replicate]

### Expected Result
[What should happen in a secure system]

### Actual Result
[What actually happens, demonstrating the vulnerability]

### Evidence
[Include:
- Screenshots with annotations
- Request/response examples
- Code snippets
- Video demonstrations if applicable]

```http
[Example HTTP request]
```

```http
[Example HTTP response]
```

## Impact Analysis

### Business Impact
[Describe the potential business consequences:
- Data breach possibilities
- Financial implications
- Reputation damage
- Legal/compliance issues]

### Technical Impact
[Describe technical consequences:
- System compromise
- Data exposure
- Service disruption
- Privilege escalation potential]

### Attack Scenario
[Describe a realistic attack scenario that exploits this vulnerability]

## Risk Assessment

### Likelihood
[High/Medium/Low] - [Explain why]

### Exploitability
[How easy is it to exploit? Consider:
- Skill level required
- Tools availability
- Access requirements
- Detection probability]

### Affected Users/Systems
[Scope of impact:
- Number of users affected
- Criticality of affected systems
- Data sensitivity involved]

## Remediation

### Recommended Solution
[Provide detailed, actionable fix recommendations:
- Code changes needed
- Configuration updates
- Architecture modifications]

**Example Fix:**
```[language]
[Code example showing the fix]
```

### Temporary Mitigations
[If immediate fix isn't possible, suggest temporary workarounds:
- WAF rules
- Access restrictions
- Monitoring enhancements]

### Verification Steps
[How to verify the fix is effective]

## References

### Related Vulnerabilities
- [CWE-XXX: Weakness Name](URL)
- [CVE-XXXX-XXXXX](URL)

### Additional Resources
- [OWASP Guide](URL)
- [Vendor Documentation](URL)
- [Security Advisory](URL)

## Timeline

| Date | Event |
|------|-------|
| YYYY-MM-DD | Vulnerability discovered |
| YYYY-MM-DD | Initial report submitted |
| YYYY-MM-DD | Vendor acknowledgment |
| YYYY-MM-DD | Fix implemented |
| YYYY-MM-DD | Verification completed |

## Additional Information

### Testing Environment
- **Tools Used**: [List of tools]
- **Testing Methodology**: [Approach used]
- **Scope**: [Testing boundaries]

### Compliance Considerations
[Note any relevant compliance frameworks:
- PCI-DSS
- GDPR
- HIPAA
- SOC 2
- ISO 27001]

## Appendices

### Appendix A: Full Request/Response Logs
[Include complete technical details]

### Appendix B: Tool Output
[Scanner results, script output, etc.]

### Appendix C: Supporting Documentation
[Any additional evidence or context]

---

**Report Prepared By**: [Your Name]
**Date**: [YYYY-MM-DD]
**Version**: [1.0]
**Classification**: [Confidential/Internal/Public]
