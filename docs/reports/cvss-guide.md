# CVSS v3.1 Scoring Guide

## Introduction

The Common Vulnerability Scoring System (CVSS) provides a standardized way to assess and communicate the severity of security vulnerabilities. This guide covers CVSS v3.1, the current standard for vulnerability assessment.

## CVSS v3.1 Overview

CVSS v3.1 scores range from 0.0 to 10.0, with higher scores indicating more severe vulnerabilities. The score is calculated using three metric groups:

1. **Base Metrics** - Intrinsic characteristics of the vulnerability
2. **Temporal Metrics** - Characteristics that change over time
3. **Environmental Metrics** - Characteristics relevant to a specific environment

## Severity Ratings

| CVSS Score | Severity Level | Description |
|------------|----------------|-------------|
| 0.0 | None | No impact |
| 0.1 - 3.9 | Low | Limited impact, minimal risk |
| 4.0 - 6.9 | Medium | Moderate impact, some risk |
| 7.0 - 8.9 | High | Significant impact, serious risk |
| 9.0 - 10.0 | Critical | Severe impact, immediate attention required |

## Base Metrics (Required)

### Attack Vector (AV)
Describes how the vulnerability is exploited:

- **Network (N)** - Exploitable remotely over a network
  - Score: 0.85
  - Example: Web application vulnerabilities, network services

- **Adjacent (A)** - Exploitable from adjacent network
  - Score: 0.62
  - Example: Bluetooth, local network attacks

- **Local (L)** - Requires local access or local account
  - Score: 0.55
  - Example: Local privilege escalation

- **Physical (P)** - Requires physical access
  - Score: 0.20
  - Example: Hardware vulnerabilities, physical device access

### Attack Complexity (AC)
Describes the conditions beyond attacker control:

- **Low (L)** - No special conditions required
  - Score: 0.77
  - Example: Simple buffer overflow, SQL injection

- **High (H)** - Specialized conditions required
  - Score: 0.44
  - Example: Race conditions, complex configuration requirements

### Privileges Required (PR)
Describes the level of privileges required:

- **None (N)** - No privileges required
  - Score: 0.85 (unchanged scope) / 0.85 (changed scope)
  - Example: Anonymous access vulnerabilities

- **Low (L)** - Basic user privileges required
  - Score: 0.62 (unchanged scope) / 0.68 (changed scope)
  - Example: Authenticated user vulnerabilities

- **High (H)** - Administrative privileges required
  - Score: 0.27 (unchanged scope) / 0.50 (changed scope)
  - Example: Admin-only vulnerabilities

### User Interaction (UI)
Describes whether user interaction is required:

- **None (N)** - No user interaction required
  - Score: 0.85
  - Example: Network service exploits

- **Required (R)** - User interaction required
  - Score: 0.62
  - Example: Social engineering, malicious links

### Scope (S)
Describes whether the vulnerability affects other components:

- **Unchanged (U)** - Impacts only the vulnerable component
  - Example: Local file system access

- **Changed (C)** - Impacts resources beyond the vulnerable component
  - Example: Sandbox escape, privilege escalation

### Confidentiality Impact (C)
Measures impact on information confidentiality:

- **High (H)** - Total loss of confidentiality
  - Score: 0.56
  - Example: All system files readable

- **Low (L)** - Some loss of confidentiality
  - Score: 0.22
  - Example: Some restricted information accessible

- **None (N)** - No loss of confidentiality
  - Score: 0.00
  - Example: Availability-only attacks

### Integrity Impact (I)
Measures impact on information integrity:

- **High (H)** - Total loss of integrity
  - Score: 0.56
  - Example: Complete system file modification

- **Low (L)** - Some loss of integrity
  - Score: 0.22
  - Example: Limited file modification

- **None (N)** - No loss of integrity
  - Score: 0.00
  - Example: Read-only vulnerabilities

### Availability Impact (A)
Measures impact on system availability:

- **High (H)** - Total loss of availability
  - Score: 0.56
  - Example: System crash, complete DoS

- **Low (L)** - Reduced performance or availability
  - Score: 0.22
  - Example: Partial DoS, performance degradation

- **None (N)** - No impact on availability
  - Score: 0.00
  - Example: Information disclosure only

## Scoring Examples

### Example 1: SQL Injection (High Severity)

**Scenario**: SQL injection in web application login form

**Metrics**:
- AV:N (Network) - Web application accessible over internet
- AC:L (Low) - Simple SQL injection, no special conditions
- PR:N (None) - No authentication required
- UI:N (None) - Direct exploitation via HTTP request
- S:U (Unchanged) - Affects only the database component
- C:H (High) - Can read entire database
- I:H (High) - Can modify all database records
- A:H (High) - Can delete database, cause DoS

**Vector String**: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`

**Score**: 9.8 (Critical)

### Example 2: Cross-Site Scripting (Medium Severity)

**Scenario**: Reflected XSS in search parameter

**Metrics**:
- AV:N (Network) - Web application accessible over internet
- AC:L (Low) - Simple XSS injection
- PR:N (None) - No authentication required
- UI:R (Required) - Victim must click malicious link
- S:C (Changed) - Can affect other users' sessions
- C:L (Low) - Can steal some user data
- I:L (Low) - Can modify page content
- A:N (None) - No availability impact

**Vector String**: `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N`

**Score**: 6.1 (Medium)

### Example 3: Local Privilege Escalation (High Severity)

**Scenario**: Buffer overflow in setuid binary

**Metrics**:
- AV:L (Local) - Requires local system access
- AC:L (Low) - Straightforward buffer overflow
- PR:L (Low) - Requires local user account
- UI:N (None) - Direct exploitation
- S:U (Unchanged) - Affects only local system
- C:H (High) - Can read all system files
- I:H (High) - Can modify all system files
- A:H (High) - Can crash system

**Vector String**: `CVSS:3.1/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`

**Score**: 7.8 (High)

### Example 4: Information Disclosure (Low Severity)

**Scenario**: Directory listing enabled on web server

**Metrics**:
- AV:N (Network) - Accessible over internet
- AC:L (Low) - Simple HTTP request
- PR:N (None) - No authentication required
- UI:N (None) - Direct access
- S:U (Unchanged) - Affects only web server
- C:L (Low) - Some file names visible
- I:N (None) - No modification possible
- A:N (None) - No availability impact

**Vector String**: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`

**Score**: 5.3 (Medium)

## Temporal Metrics (Optional)

### Exploit Code Maturity (E)
- **Not Defined (X)** - Default value
- **High (H)** - Functional autonomous code exists
- **Functional (F)** - Functional exploit code available
- **Proof-of-Concept (P)** - Proof-of-concept code available
- **Unproven (U)** - No exploit code available

### Remediation Level (RL)
- **Not Defined (X)** - Default value
- **Unavailable (U)** - No solution available
- **Workaround (W)** - Unofficial, non-vendor solution
- **Temporary Fix (T)** - Official temporary fix
- **Official Fix (O)** - Complete vendor solution

### Report Confidence (RC)
- **Not Defined (X)** - Default value
- **Confirmed (C)** - Detailed reports, confirmed by vendor
- **Reasonable (R)** - Significant details, corroborated sources
- **Unknown (U)** - Few or conflicting technical details

## Environmental Metrics (Optional)

### Modified Base Metrics
Environmental metrics allow organizations to customize the base score based on their specific environment. Each base metric has a corresponding modified version:

- Modified Attack Vector (MAV)
- Modified Attack Complexity (MAC)
- Modified Privileges Required (MPR)
- Modified User Interaction (MUI)
- Modified Scope (MS)
- Modified Confidentiality Impact (MC)
- Modified Integrity Impact (MI)
- Modified Availability Impact (MA)

### Requirements
- **Confidentiality Requirement (CR)**
- **Integrity Requirement (IR)**
- **Availability Requirement (AR)**

Values: High (H), Medium (M), Low (L), Not Defined (X)

## Scoring Best Practices

### 1. Focus on Base Metrics
- Always provide base metrics
- Use temporal and environmental metrics when relevant
- Base score is most commonly used for prioritization

### 2. Consider Real-World Impact
- Evaluate actual exploitability in your environment
- Consider existing security controls
- Account for business impact

### 3. Document Your Reasoning
- Explain why each metric value was chosen
- Provide context for scoring decisions
- Include environmental factors

### 4. Regular Review
- Re-evaluate scores as conditions change
- Update when new exploits become available
- Adjust based on remediation progress

## Common Scoring Mistakes

### 1. Overestimating Complexity
- Don't assume high complexity without evidence
- Consider attacker skill level realistically
- Simple vulnerabilities often have low complexity

### 2. Underestimating Network Impact
- Web applications typically have Network attack vector
- Consider indirect network access paths
- Don't default to Local without justification

### 3. Ignoring Scope Changes
- Privilege escalation often changes scope
- Container escapes change scope
- Cross-user impacts typically change scope

### 4. Confusing Prerequisites with Complexity
- Having prerequisites doesn't mean high complexity
- Authentication requirements affect Privileges Required, not Attack Complexity
- User interaction is separate from Attack Complexity

## Integration with Risk Management

### Vulnerability Prioritization
```
Critical (9.0-10.0): Immediate action required
├── Patch within 24-48 hours
├── Consider emergency change procedures
└── Notify senior management

High (7.0-8.9): Urgent action required
├── Patch within 1 week
├── Implement temporary mitigations
└── Escalate to security team

Medium (4.0-6.9): Timely action required
├── Patch within 30 days
├── Schedule during maintenance windows
└── Regular monitoring

Low (0.1-3.9): Lower priority
├── Patch within 90 days
├── Include in regular updates
└── Periodic review
```

### Risk Context Factors
- **Asset Criticality**: Adjust priority based on system importance
- **Threat Landscape**: Consider active exploitation
- **Compensating Controls**: Factor in existing protections
- **Business Impact**: Align with business priorities

## Tools and Resources

### CVSS Calculators
- [FIRST CVSS Calculator](https://www.first.org/cvss/calculator/3.1)
- [NVD CVSS Calculator](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator)
- [NIST SP 800-126](https://csrc.nist.gov/publications/detail/sp/800-126/rev-3/final)

### Validation Tools
- CVSS vector string validators
- Automated scoring tools
- Integration with vulnerability scanners

### Training Resources
- FIRST CVSS training materials
- Vendor-specific CVSS guides
- Industry best practices documentation

## Conclusion

CVSS v3.1 provides a standardized framework for vulnerability assessment, but proper application requires understanding both the technical details and business context. Always consider:

1. **Accuracy**: Choose metrics based on technical analysis
2. **Context**: Consider environmental factors
3. **Consistency**: Apply metrics uniformly across assessments
4. **Communication**: Explain scoring rationale to stakeholders

Remember that CVSS is a tool to aid decision-making, not replace professional judgment. Use it as part of a comprehensive vulnerability management program.

---

**References**:
- [CVSS v3.1 Specification](https://www.first.org/cvss/v3.1/specification-document)
- [NIST SP 800-126 Rev. 3](https://csrc.nist.gov/publications/detail/sp/800-126/rev-3/final)
- [FIRST CVSS Implementation Guidance](https://www.first.org/cvss/implementation-guidance)

**Last Updated**: January 2024
**Version**: 1.0
