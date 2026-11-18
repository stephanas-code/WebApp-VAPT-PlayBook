# 🔐 Web Application VAPT PlayBook
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Last Commit](https://img.shields.io/github/last-commit/stephanas-code/WebApp-VAPT-PlayBook)
![Issues](https://img.shields.io/github/issues/stephanas-code/WebApp-VAPT-PlayBook)
![GitHub Stars](https://img.shields.io/github/stars/stephanas-code/WebApp-VAPT-PlayBook?style=social)

> **Authored by Stephanas Odogu** — A comprehensive, production-ready playbook for web application vulnerability assessment and penetration testing (ethical use only)

## 📖 About This Project

Welcome to the **Web Application VAPT PlayBook**! This repository is a comprehensive, production-ready resource authored by **Stephanas Odogu**, designed for security enthusiasts, ethical hackers, and professionals looking to master web application security testing.

Whether you're a beginner taking your first steps into cybersecurity, a professional honing your skills, or someone making a career change into infosec, this guide provides structured, hands-on learning paths with real-world examples.

**See the modules/, labs/, and reports/ directories for documentation and reporting templates.**

### 🎯 What Makes This Different?

- **OWASP-Focused**: All modules mapped to OWASP Top 10 and beyond
- **Practical Examples**: Real Burp Suite requests, responses, and payloads
- **Beginner-Friendly**: Clear explanations without jargon overload
- **Report Templates**: Professional vulnerability report examples ready to use
- **Practice Labs**: Guided exercises with DVWA, Juice Shop, and more
- **Ethical Framework**: Strong emphasis on legal and responsible testing

### 👥 Who Is This For?

- **Beginners**: New to web security? Start with our learning path
- **Students**: Preparing for certifications (CEH, OSCP, eWPT)
- **Professionals**: Brushing up on specific vulnerability classes
- **Career Changers**: Transitioning into cybersecurity from other fields
- **Bug Bounty Hunters**: Quick reference for common vulnerability patterns

## 📚 Table of Contents

### Core Documentation

- [Getting Started](getting-started.md) - Setup your lab environment
- [Burp Suite Modern PlayBook](cheatsheets/burp-suite.md) - 2026 hands-on testing guide, extensions matrix & advanced workflows
- [Contributing Guidelines](CONTRIBUTING.md) - How to contribute
- [Code of Conduct](CODE_OF_CONDUCT.md) - Community standards
- [Security Policy](SECURITY.md) - Responsible disclosure
- [Disclaimer](DISCLAIMER.md) - Legal and ethical guidance

### 🎓 Vulnerability Modules & Threat Mapping

Each module provides end-to-end vulnerability analysis, Burp Suite request/response artifacts, and 2026 AI-assisted remediation patterns mapped against industry standards:

| Module | OWASP Top 10 | Primary CWEs | Typical CVSS v3.1 / v4.0 | Benchmark CVE Examples to Investigate |
| :--- | :--- | :--- | :--- | :--- |
| [**Authentication Bypass**](modules/auth-bypass/) | A07:2021 - Identification & Authentication Failures | CWE-287, CWE-306, CWE-384, CWE-798 | **7.5 – 9.8** (High / Critical) | CVE-2024-4577, CVE-2024-21887, CVE-2023-46805 |
| [**Business Logic Flaws**](modules/business-logic-flaws/) | A04:2021 - Insecure Design (API8 / API6) | CWE-840, CWE-362, CWE-799, CWE-837 | **5.3 – 8.8** (Medium / High) | CVE-2023-38035, CVE-2022-26134 |
| [**CSRF (Cross-Site Request Forgery)**](modules/csrf/) | A01:2021 - Broken Access Control | CWE-352, CWE-359 | **4.3 – 8.8** (Medium / High) | CVE-2023-26360, CVE-2021-34473 |
| [**File Upload Vulnerabilities**](modules/file-upload/) | A04:2021 - Insecure Design / A03: Injection | CWE-434, CWE-22, CWE-400 | **7.5 – 9.8** (High / Critical) | CVE-2024-27198, CVE-2023-22515, CVE-2021-41773 |
| [**IDOR / BOLA**](modules/idor/) | A01:2021 - Broken Access Control (API1:2023) | CWE-639, CWE-284, CWE-862 | **6.5 – 9.1** (Medium / Critical) | CVE-2023-34362, CVE-2021-22941 |
| [**Insecure Deserialization**](modules/insecure-deserialization/) | A08:2021 - Software & Data Integrity Failures | CWE-502, CWE-1321 | **8.1 – 9.8** (High / Critical) | CVE-2024-21626, CVE-2023-38646, CVE-2022-22965 |
| [**Command Injection / RCE**](modules/rce-command-injection/) | A03:2021 - Injection | CWE-78, CWE-88, CWE-77 | **8.8 – 10.0** (Critical) | CVE-2024-6387, CVE-2024-3400, CVE-2021-44228 |
| [**Security Misconfiguration**](modules/security-misconfiguration/) | A05:2021 - Security Misconfiguration | CWE-16, CWE-200, CWE-1188, CWE-611 | **5.3 – 9.8** (Medium / Critical) | CVE-2024-23897, CVE-2023-22518, CVE-2023-20198 |
| [**Sensitive Data Exposure**](modules/sensitive-data-exposure/) | A02:2021 - Cryptographic Failures | CWE-312, CWE-319, CWE-327, CWE-330 | **5.3 – 8.2** (Medium / High) | CVE-2024-24919, CVE-2023-4966 |
| [**SQL Injection (SQLi)**](modules/sqli/) | A03:2021 - Injection | CWE-89, CWE-564 | **8.5 – 9.8** (High / Critical) | CVE-2023-34362, CVE-2023-38646, CVE-2022-24816 |
| [**SSRF (Server-Side Request Forgery)**](modules/ssrf/) | A10:2021 - Server-Side Request Forgery | CWE-918, CWE-610 | **7.5 – 9.8** (High / Critical) | CVE-2024-21887, CVE-2023-46805, CVE-2021-26855 |
| [**XSS (Cross-Site Scripting)**](modules/xss/) | A03:2021 - Injection | CWE-79, CWE-80, CWE-116 | **4.7 – 8.8** (Medium / High) | CVE-2024-21683, CVE-2023-27350, CVE-2023-22527 |

---

## 🔍 Key Security Standards & Taxonomies to Research
- **OWASP Top 10 (2021 & API Security Top 10 2023)**: Standard awareness framework outlining the most critical security risks to web applications and modern APIs.
- **Common Weakness Enumeration (CWE)**: Universal catalog of software weaknesses, flaws, and fault patterns (maintained by MITRE).
- **Common Vulnerability Scoring System (CVSS v3.1 / CVSS v4.0)**: Standardized framework for assessing the severity and characteristics of vulnerabilities based on attack vector, complexity, privileges required, and CIA impact.
- **Common Vulnerabilities and Exposures (CVE)**: Dictionary of publicly disclosed cybersecurity vulnerabilities (NVD/MITRE) serving as real-world case studies for root cause analysis and defensive validation.
