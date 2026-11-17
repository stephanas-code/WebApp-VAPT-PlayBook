# CVSS (Common Vulnerability Scoring System) Guide

## Overview

The Common Vulnerability Scoring System (CVSS) provides a standardized method for rating the severity of security vulnerabilities. This guide covers CVSS version 3.1, which is the current standard used in vulnerability assessment and penetration testing.

## What is CVSS?

CVSS is an open framework for communicating the characteristics and severity of software vulnerabilities. It consists of three metric groups:

1. **Base Metrics** - Intrinsic characteristics of a vulnerability
2. **Temporal Metrics** - Characteristics that evolve over time
3. **Environmental Metrics** - Characteristics relevant to a user's environment

## CVSS 3.1 Base Metrics

### Attack Vector (AV)

Describes how the vulnerability can be exploited.

| Value | Description | Score |
|-------|-------------|-------|
| **Network (N)** | Exploitable remotely over a network | 0.85 |
| **Adjacent (A)** | Exploitable only from adjacent network access | 0.62 |
| **Local (L)** | Requires local access or social engineering | 0.55 |
| **Physical (P)** | Requires physical access to the device | 0.20 |

**Examples:**
- Network: SQL injection in a web application
- Adjacent: Bluetooth vulnerability
- Local: Privilege escalation exploit
- Physical: USB-based attack

### Attack Complexity (AC)

Describes the conditions beyond the attacker's control that must exist to exploit the vulnerability.

| Value | Description | Score |
|-------|-------------|-------|
| **Low (L)** | No special attack conditions exist | 0.77 |
| **High (H)** | Successful attack requires special conditions | 0.44 |

**Examples:**
- Low: Basic SQL injection
- High: Race condition that requires precise timing

### Privileges Required (PR)

Describes the level of privileges an attacker must possess before exploitation.

| Value | Description | Score (Unchanged Scope) | Score (Changed Scope) |
|-------|-------------|------------------------|----------------------|
| **None (N)** | No privileges required | 0.85 | 0.85 |
| **Low (L)** | Basic user privileges required | 0.62 | 0.68 |
| **High (H)** | Admin privileges required | 0.27 | 0.50 |

**Examples:**
- None: Anonymous access vulnerability
- Low: Authenticated user privilege escalation
- High: Admin-to-root escalation

### User Interaction (UI)

Describes whether vulnerability exploitation requires user interaction.

| Value | Description | Score |
|-------|-------------|-------|
| **None (N)** | No user interaction required | 0.85 |
| **Required (R)** | User interaction is required | 0.62 |

**Examples:**
- None: Automated exploit possible
- Required: Requires clicking malicious link

### Scope (S)

Determines whether a successful exploit can affect components beyond the vulnerable component.

| Value | Description |
|-------|-------------|
| **Unchanged (U)** | Exploit impacts only the vulnerable component |
| **Changed (C)** | Exploit can impact other components |

**Examples:**
- Unchanged: File disclosure in web application
- Changed: Container escape affecting host system

### Confidentiality Impact (C)

Measures impact on confidentiality of information managed by the component.

| Value | Description | Score |
|-------|-------------|-------|
| **High (H)** | Total information disclosure | 0.56 |
| **Low (L)** | Some information disclosed | 0.22 |
| **None (N)** | No impact on confidentiality | 0.00 |

### Integrity Impact (I)

Measures impact on integrity of the exploited system.

| Value | Description | Score |
|-------|-------------|-------|
| **High (H)** | Total compromise of system integrity | 0.56 |
| **Low (L)** | Modification of some information | 0.22 |
| **None (N)** | No impact on integrity | 0.00 |

### Availability Impact (A)

Measures impact on availability of the impacted component.

| Value | Description | Score |
|-------|-------------|-------|
| **High (H)** | Total shutdown or severe performance degradation | 0.56 |
| **Low (L)** | Reduced performance or intermittent outages | 0.22 |
| **None (N)** | No impact on availability | 0.00 |

## CVSS Score Calculation

### Base Score Formula

**Step 1: Calculate Impact Score**

```
Impact = 1 - [(1 - Confidentiality) × (1 - Integrity) × (1 - Availability)]

If Scope is Unchanged:
  Impact Score = 6.42 × Impact

If Scope is Changed:
  Impact Score = 7.52 × (Impact - 0.029) - 3.25 × (Impact - 0.02)^15
```

**Step 2: Calculate Exploitability Score**

```
Exploitability = 8.22 × Attack Vector × Attack Complexity × Privileges Required × User Interaction
```

**Step 3: Calculate Base Score**

```
If Impact Score ≤ 0:
  Base Score = 0
Else:
  If Scope is Unchanged:
    Base Score = min(10, Impact Score + Exploitability)
  If Scope is Changed:
    Base Score = min(10, 1.08 × (Impact Score + Exploitability))
```

## Severity Ratings

| CVSS Score | Severity Level | Action Required |
|------------|---------------|----------------|
| 0.0 | None | No action required |
| 0.1 - 3.9 | Low | Monitor and plan remediation |
| 4.0 - 6.9 | Medium | Remediate within planned cycles |
| 7.0 - 8.9 | High | Remediate as soon as possible |
| 9.0 - 10.0 | Critical | Immediate remediation required |

## Practical Examples

### Example 1: SQL Injection

**Scenario:** Unauthenticated SQL injection in web application login form

- **AV:N** - Network accessible
- **AC:L** - Simple to exploit
- **PR:N** - No authentication required
- **UI:N** - No user interaction needed
- **S:U** - Affects only the database
- **C:H** - Full data access
- **I:H** - Can modify all data
- **A:H** - Can delete data/crash system

**CVSS Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`
**Base Score:** **9.8 (Critical)**

### Example 2: Cross-Site Scripting (XSS)

**Scenario:** Reflected XSS requiring user to click malicious link

- **AV:N** - Network accessible
- **AC:L** - Simple to exploit
- **PR:N** - No authentication required
- **UI:R** - User must click link
- **S:C** - Can affect other users
- **C:L** - Limited data access
- **I:L** - Limited data modification
- **A:N** - No availability impact

**CVSS Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N`
**Base Score:** **6.1 (Medium)**

### Example 3: Local Privilege Escalation

**Scenario:** Local user can gain root privileges

- **AV:L** - Local access required
- **AC:L** - Simple to exploit
- **PR:L** - Low privileges required
- **UI:N** - No user interaction
- **S:U** - Affects only local system
- **C:H** - Full system access
- **I:H** - Full system control
- **A:H** - Can crash system

**CVSS Vector:** `CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`
**Base Score:** **7.8 (High)**

## Best Practices for CVSS Usage

### 1. Consistency
- Use the same CVSS version across all assessments
- Document scoring rationale for complex cases
- Review scores with team members for consistency

### 2. Context Consideration
- Consider environmental factors beyond base score
- Adjust priority based on business impact
- Account for compensating controls

### 3. Regular Updates
- Re-evaluate scores as new information becomes available
- Update temporal metrics as exploits/patches become available
- Consider environmental changes affecting impact

### 4. Communication
- Always include CVSS vector string
- Explain scoring rationale in reports
- Provide clear remediation priorities

## Common Scoring Mistakes

### 1. Overestimating Complexity
- Don't set AC:H just because exploitation requires tools
- AC:H is for conditions beyond attacker control

### 2. Confusing Access Requirements
- PR refers to privileges on the target system
- Don't confuse with attack vector requirements

### 3. Impact Assessment Errors
- Base impact on potential, not demonstrated impact
- Consider worst-case scenario for each metric

### 4. Scope Confusion
- Scope changes when vulnerability affects other components
- Consider privilege boundaries and trust zones

## CVSS Tools and Resources

### Official Resources
- [CVSS v3.1 Specification](https://www.first.org/cvss/v3.1/specification-document)
- [CVSS v3.1 Calculator](https://www.first.org/cvss/calculator/3.1)
- [CVSS Examples](https://www.first.org/cvss/examples)

### Useful Tools
- **NIST NVD Calculator** - https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator
- **CVSS Score Calculator** - Command line and API tools
- **Vulnerability Scanners** - Most include CVSS scoring

## Integration with Penetration Testing

### During Testing
1. Score vulnerabilities as they are discovered
2. Use scores to prioritize further testing
3. Focus detailed analysis on high/critical findings

### In Reports
1. Include CVSS vector for each finding
2. Sort findings by severity (CVSS score)
3. Provide risk matrix combining CVSS and business context

### For Remediation
1. Use CVSS scores for initial prioritization
2. Adjust based on environmental factors
3. Consider exploit availability and patch status

## Advanced Topics

### Temporal Metrics

**Exploit Code Maturity (E):**
- Unproven (U): 0.91
- Proof-of-Concept (P): 0.94
- Functional (F): 0.97
- High (H): 1.0

**Remediation Level (RL):**
- Official Fix (O): 0.95
- Temporary Fix (T): 0.96
- Workaround (W): 0.97
- Unavailable (U): 1.0

**Report Confidence (RC):**
- Unknown (U): 0.92
- Reasonable (R): 0.96
- Confirmed (C): 1.0

### Environmental Metrics

**Modified Base Metrics:** Adjust base metrics for local environment
**Requirements:** Adjust importance of C/I/A for specific environment

### CVSS 4.0 Considerations

While CVSS 3.1 is still the standard, CVSS 4.0 introduces:
- Additional base metrics
- Refined impact calculations
- Better handling of complex scenarios

## Conclusion

CVSS provides a standardized framework for vulnerability severity assessment, but should be used in conjunction with environmental context and business impact analysis. Consistent application of CVSS methodology improves communication between security teams and helps prioritize remediation efforts effectively.

---

**Reference:** This guide is based on CVSS v3.1 specification by FIRST (Forum of Incident Response and Security Teams)
**Last Updated:** 2024-01-15
