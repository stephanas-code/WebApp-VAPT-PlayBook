# Reports Directory

This directory contains vulnerability reporting templates, examples, and guides for web application vulnerability assessment and penetration testing (VAPT).

## Available Files

### Templates and Examples

- **[vuln-report-template.md](vuln-report-template.md)** - Standardized template for vulnerability reports
- **[vuln-report-example.md](vuln-report-example.md)** - Complete example of a vulnerability report showing proper documentation format

### Guides and References

- **[cvss-guide.md](cvss-guide.md)** - Comprehensive guide to CVSS (Common Vulnerability Scoring System) version 3.1

## Usage Guidelines

### For Vulnerability Reports

1. **Use the Template**: Start with `vuln-report-template.md` for consistent reporting format
2. **Follow the Example**: Reference `vuln-report-example.md` to understand proper documentation structure
3. **Apply CVSS Scoring**: Use `cvss-guide.md` to properly score vulnerability severity

### Report Structure

All vulnerability reports should include:

- Executive Summary
- Vulnerability Details (with CVSS scoring)
- Technical Analysis
- Proof of Concept
- Impact Assessment
- Remediation Steps
- References and Timeline

### Quality Standards

- Follow industry best practices for vulnerability disclosure
- Include clear, reproducible proof-of-concept steps
- Provide actionable remediation guidance
- Use proper CVSS v3.1 scoring methodology
- Maintain professional and objective tone

## File Descriptions

### vuln-report-template.md
A blank template with all necessary sections and formatting guidelines for creating professional vulnerability reports. Use this as your starting point for new vulnerability documentation.

### vuln-report-example.md
A complete example vulnerability report documenting an SQL injection finding. This demonstrates:
- Proper formatting and structure
- Technical detail requirements
- CVSS scoring application
- Professional remediation guidance
- Industry-standard terminology

### cvss-guide.md
Detailed reference guide for CVSS v3.1 including:
- Complete metric explanations
- Scoring methodology
- Practical examples
- Common mistakes to avoid
- Integration with penetration testing workflows

## Contributing

When adding new reports or updating existing ones:

1. Follow the established template structure
2. Ensure CVSS scores are properly calculated
3. Include all required sections
4. Test any proof-of-concept code
5. Review for accuracy and completeness

## Related Resources

- [Main Project README](../README.md)
- [Getting Started Guide](../getting-started.md)
- [Security Policy](../SECURITY.md)
- [Contributing Guidelines](../CONTRIBUTING.md)

---

**Note**: All vulnerability reports should follow responsible disclosure practices and comply with applicable laws and regulations.
