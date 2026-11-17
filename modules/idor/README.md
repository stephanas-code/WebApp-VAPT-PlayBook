# IDOR (Insecure Direct Object Reference) Vulnerability Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Understanding IDOR](#understanding-idor)
3. [BOLA vs IDOR](#bola-vs-idor)
4. [Authorization Flaws](#authorization-flaws)
5. [Attack Types](#attack-types)
6. [Practical Examples](#practical-examples)
7. [Business Impact](#business-impact)
8. [Detection Techniques](#detection-techniques)
9. [Risk Assessment](#risk-assessment)
10. [Remediation](#remediation)

## Introduction

Insecure Direct Object Reference (IDOR) is a critical vulnerability that occurs when an application provides direct access to objects based on user-supplied input. This vulnerability allows attackers to bypass authorization and access resources belonging to other users or the system.

## Understanding IDOR

IDOR vulnerabilities arise when:
- Applications expose internal implementation objects (files, directories, database keys)
- Insufficient access controls are implemented
- User input is used to directly reference objects without proper validation
- Authorization checks are missing or inadequate

### Common Scenarios
- Direct database record access via ID parameters
- File system access through path manipulation
- API endpoints exposing internal object references
- Session tokens or user IDs in URLs

## BOLA vs IDOR

### BOLA (Broken Object Level Authorization)
- **Scope**: Broader category focusing on authorization failures
- **Context**: Modern API-centric applications
- **Focus**: Object-level access control mechanisms
- **Classification**: OWASP API Security Top 10 #1

### IDOR (Insecure Direct Object Reference)
- **Scope**: Specific vulnerability pattern
- **Context**: Traditional web applications
- **Focus**: Direct object reference without authorization
- **Classification**: Historical OWASP Top 10 category

**Relationship**: IDOR is essentially a subset of BOLA vulnerabilities in modern terminology.

## Authorization Flaws

### Types of Authorization Failures

1. **Missing Authorization**
   - No access control checks implemented
   - Direct object access without validation

2. **Inadequate Authorization**
   - Weak or bypassable access controls
   - Client-side only authorization

3. **Broken Authorization Logic**
   - Flawed authorization implementation
   - Race conditions in authorization checks

4. **Privilege Escalation**
   - Horizontal privilege escalation (accessing peer resources)
   - Vertical privilege escalation (accessing admin resources)

## Attack Types

### 1. Parameter Manipulation
```
# Original request
GET /user/profile?id=123

# Malicious request
GET /user/profile?id=124
```

### 2. Path Traversal
```
# Original request
GET /files/user123/document.pdf

# Malicious request
GET /files/user124/document.pdf
```

### 3. Body Parameter Manipulation
```json
// Original request
{
  "userId": 123,
  "action": "view_profile"
}

// Malicious request
{
  "userId": 124,
  "action": "view_profile"
}
```

### 4. Header Manipulation
```
# Original request
X-User-ID: 123

# Malicious request
X-User-ID: 124
```

### 5. Cookie Manipulation
```
# Original cookie
user_id=123; session=abc123

# Malicious cookie
user_id=124; session=abc123
```

## Practical Examples

### Example 1: Banking Application
```
# Legitimate request
GET /api/accounts/12345/balance

# IDOR Attack
GET /api/accounts/12346/balance
```
**Impact**: Access to other users' account balances

### Example 2: Social Media Platform
```
# Legitimate request
POST /api/posts/delete
Content-Type: application/json
{"post_id": "user123_post456"}

# IDOR Attack
POST /api/posts/delete
Content-Type: application/json
{"post_id": "user124_post789"}
```
**Impact**: Ability to delete other users' posts

### Example 3: Healthcare System
```
# Legitimate request
GET /patient/records?patient_id=P001

# IDOR Attack
GET /patient/records?patient_id=P002
```
**Impact**: Unauthorized access to medical records

### Example 4: E-commerce Platform
```
# Legitimate request
GET /order/details/ORD-123-456

# IDOR Attack
GET /order/details/ORD-123-457
```
**Impact**: Access to other customers' order information

## Business Impact

### Financial Impact
- **Data breach costs**: Average $4.45 million per breach (IBM 2023)
- **Regulatory fines**: GDPR fines up to €20 million or 4% of annual revenue
- **Legal costs**: Class-action lawsuits and settlements
- **Business disruption**: System downtime and remediation costs

### Reputational Impact
- Loss of customer trust
- Brand damage and negative publicity
- Competitive disadvantage
- Market share loss

### Operational Impact
- System downtime for remediation
- Increased security infrastructure costs
- Enhanced monitoring and compliance requirements
- Customer support overhead

### Compliance Impact
- GDPR violations (Privacy regulation)
- HIPAA violations (Healthcare data)
- PCI DSS violations (Payment card data)
- SOX violations (Financial data)

## Detection Techniques

### Manual Testing
1. **Parameter Analysis**
   - Identify direct object references in URLs, forms, and APIs
   - Map user roles and accessible objects
   - Test parameter manipulation

2. **Session Analysis**
   - Test with multiple user accounts
   - Analyze authorization mechanisms
   - Check for session-based access controls

### Automated Testing
1. **Security Scanners**
   - OWASP ZAP
   - Burp Suite Professional
   - Netsparker
   - Acunetix

2. **API Testing Tools**
   - Postman with automated testing
   - REST Assured
   - Insomnia

### Code Review
1. **Static Analysis**
   - Review authorization logic
   - Check for direct object access patterns
   - Analyze access control implementations

2. **Dynamic Analysis**
   - Runtime authorization testing
   - Access control bypass attempts
   - Privilege escalation testing

## Risk Assessment

### Risk Factors

| Factor | High Risk | Medium Risk | Low Risk |
|--------|-----------|-------------|----------|
| **Data Sensitivity** | PII, Financial, Medical | Business data, Internal docs | Public information |
| **User Base** | Public-facing | Partner/B2B | Internal only |
| **Access Method** | Direct DB access | API endpoints | Filtered access |
| **Authentication** | Weak/None | Basic | Multi-factor |
| **Monitoring** | None | Basic logging | Comprehensive |

### CVSS Scoring
- **Base Score**: 6.5-8.5 (Medium to High)
- **Temporal Score**: Varies based on exploit availability
- **Environmental Score**: Depends on data sensitivity and business impact

### Risk Matrix

| Likelihood | Impact | Risk Level | Priority |
|------------|---------|------------|----------|
| High | High | Critical | P0 |
| High | Medium | High | P1 |
| Medium | High | High | P1 |
| Medium | Medium | Medium | P2 |
| Low | High | Medium | P2 |

## Remediation

### Immediate Actions
1. **Access Control Implementation**
   ```python
   # Before (Vulnerable)
   def get_user_profile(user_id):
       return database.get_user(user_id)
   
   # After (Secure)
   def get_user_profile(user_id, current_user):
       if current_user.id != user_id and not current_user.is_admin():
           raise UnauthorizedError("Access denied")
       return database.get_user(user_id)
   ```

2. **Input Validation**
   ```python
   # Validate user ownership
   def validate_resource_access(resource_id, user_id):
       resource = get_resource(resource_id)
       if resource.owner_id != user_id:
           raise UnauthorizedError("Resource access denied")
       return resource
   ```

### Long-term Solutions
1. **Implement Proper Authorization Framework**
   - Role-Based Access Control (RBAC)
   - Attribute-Based Access Control (ABAC)
   - Policy-Based Access Control

2. **Use Indirect References**
   ```python
   # Instead of direct IDs
   /api/users/12345/profile
   
   # Use session-based or mapped references
   /api/users/me/profile
   /api/users/current/profile
   ```

3. **Implement Access Control Lists (ACLs)**
   ```python
   class ResourceACL:
       def __init__(self, resource_id, user_id):
           self.resource_id = resource_id
           self.user_id = user_id
       
       def can_read(self):
           return self._check_permission('read')
       
       def can_write(self):
           return self._check_permission('write')
   ```

### Security Controls
1. **Authentication Controls**
   - Strong authentication mechanisms
   - Multi-factor authentication
   - Session management

2. **Authorization Controls**
   - Principle of least privilege
   - Regular access reviews
   - Automated access control testing

3. **Monitoring Controls**
   - Access logging and monitoring
   - Anomaly detection
   - Real-time alerting

## Additional Resources

### OWASP References
- [OWASP Top 10 - A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP API Security Top 10 - API1 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP Testing Guide - Authorization Testing](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/)

### Tools and Frameworks
- [Burp Suite Professional](https://portswigger.net/burp/pro)
- [OWASP ZAP](https://owasp.org/www-project-zap/)
- [Authz (Authorization Framework)](https://github.com/stalniy/casl)
- [Casbin (Authorization Library)](https://casbin.org/)

---

**Next Steps**: 
- Review [Payloads.md](./Payloads.md) for testing payloads
- Check [Remediation.md](./Remediation.md) for detailed fix implementations
- Follow [Reproduce.md](./Reproduce.md) for hands-on practice
- Use [BurpSuite-Guide.md](./BurpSuite-Guide.md) for tool-specific testing
