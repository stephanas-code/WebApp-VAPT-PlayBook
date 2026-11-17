# Contributing to Web Application VAPT Guide

First off, thank you for considering contributing to this project! 🎉

This guide is built by the security community, for the community. Your contributions help make web security knowledge more accessible to everyone.

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Contribution Guidelines](#contribution-guidelines)
- [Style Guide](#style-guide)
- [Commit Message Guidelines](#commit-message-guidelines)
- [Pull Request Process](#pull-request-process)
- [Responsible Disclosure](#responsible-disclosure)

## 📜 Code of Conduct

This project adheres to the Contributor Covenant [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to the project maintainers.

## 🤝 How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- **Clear title** and detailed description
- **Steps to reproduce** the issue
- **Expected vs actual behavior**
- **Screenshots** if applicable
- **Environment details** (OS, browser, Burp version, etc.)

### Suggesting Enhancements

Enhancement suggestions are welcome! Please:

- Use a clear, descriptive title
- Provide detailed explanation of the suggested enhancement
- Explain why this enhancement would be useful
- Include examples if possible

### Adding New Modules

Want to add a new vulnerability module? Great! Each module should include:

1. **README.md** - Overview and explanation of the vulnerability
2. **burp/** folder - Sample HTTP requests/responses
3. **payloads.md** - Example payloads with explanations
4. **reproduce.md** - Step-by-step reproduction guide
5. **remediation.md** - Mitigation strategies

### Improving Documentation

Documentation improvements are always appreciated:

- Fix typos or grammatical errors
- Clarify confusing explanations
- Add missing information
- Improve formatting
- Add diagrams or screenshots

### Contributing Payloads

When contributing new payloads:

- Explain **why** the payload works
- Provide **context** for when to use it
- Include **variations** for different scenarios
- Test payloads in controlled environments
- Never include real credentials or sensitive data

## 🛠️ Development Setup

### Prerequisites

- Git installed locally
- GitHub account
- Text editor (VS Code, Sublime, etc.)
- Markdown knowledge

### Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/webapp-vapt-guide.git
cd webapp-vapt-guide

# Add upstream remote
git remote add upstream https://github.com/stephanas-code/WebApp-VAPT-PlayBook.git
```

### Keep Your Fork Updated

```bash
git fetch upstream
git checkout main
git merge upstream/main
```

## 📝 Contribution Guidelines

### Content Quality

- **Accuracy**: Ensure all technical information is correct
- **Clarity**: Write for beginners, avoid unnecessary jargon
- **Completeness**: Provide full context and examples
- **Ethics**: Always emphasize legal and responsible use

### File Organization

```
modules/
└── vulnerability-name/
    ├── README.md          # Overview
    ├── burp/
    │   ├── example-1.txt
    │   └── example-2.txt
    ├── payloads.md        # Payload examples
    ├── reproduce.md       # Reproduction steps
    └── remediation.md     # Mitigation guide
```

### Markdown Standards

- Use clear headings hierarchy (H1 → H2 → H3)
- Include code blocks with language specification
- Use bullet points for lists
- Add line breaks for readability
- Include links to external resources

### Burp Suite Examples Format

```markdown
## Example: [Brief Description]

### Request

```http
GET /vulnerable?param=<payload> HTTP/1.1
Host: target.local
User-Agent: Burp/2.1
Cookie: session=abc123
```

### Response

```http
HTTP/1.1 200 OK
Content-Type: text/html

<html><body>Reflected: <payload></body></html>
```

### Explanation

- **Line 1**: The payload is injected in the `param` parameter
- **Response**: Notice the payload is reflected without encoding
- **Impact**: Allows execution of arbitrary JavaScript
```

## ✍️ Style Guide

### Language

- Use clear, concise English
- Write in active voice
- Avoid overly technical jargon without explanation
- Be professional but approachable

### Formatting

- **Bold** for emphasis on key terms
- `Code blocks` for commands, code, and file paths
- > Blockquotes for important warnings or notes
- Emojis sparingly for visual organization (like these headers!)

### Examples

- Always provide working examples
- Test examples before submitting
- Include expected output
- Explain each step clearly

## 💬 Commit Message Guidelines

We follow conventional commit format:

```
type(scope): brief description

[optional body]

[optional footer]
```

### Types

- `feat`: New feature or module
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting, typos
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance tasks

### Examples

```
feat(modules): add DOM-based XSS module

docs(readme): improve quick start instructions

fix(sqli): correct blind SQLi payload syntax

chore(deps): update Burp Suite references to v2024
```

## 🔄 Pull Request Process

### Before Submitting

1. **Test your changes** - Ensure all links work and examples are accurate
2. **Check formatting** - Verify Markdown renders correctly
3. **Update documentation** - If adding features, update relevant docs
4. **Follow style guide** - Maintain consistency with existing content

### Submission Steps

1. **Create a branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes** with clear commits

3. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

4. **Open a Pull Request** on GitHub with:
   - Clear title describing the change
   - Detailed description of what and why
   - Reference any related issues
   - Screenshots if applicable

### Pull Request Template

```markdown
## Description
[Brief description of changes]

## Type of Change
- [ ] Bug fix
- [ ] New feature/module
- [ ] Documentation update
- [ ] Performance improvement

## Testing
[How you tested the changes]

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No breaking changes
```

### Review Process

- Maintainers will review your PR within 3-5 business days
- You may be asked to make changes
- Once approved, your PR will be merged
- You'll be added to the contributors list! 🎉

## 🔒 Responsible Disclosure

### Security Vulnerabilities

If you discover a security vulnerability in this repository:

- **DO NOT** open a public issue
- Report privately through GitHub Security Advisories
- Or email security concerns to [security contact - see SECURITY.md]
- Allow reasonable time for fixes before public disclosure

### Sensitive Information

Never include in contributions:

- ❌ Real credentials (usernames, passwords, API keys)
- ❌ Production system details
- ❌ Personal information
- ❌ Proprietary information
- ❌ Actual vulnerability details from unauthorized testing

### Ethical Guidelines

- Only test systems you own or have permission to test
- Use sanitized, fictional examples
- Don't promote malicious use of techniques
- Include ethical disclaimers in tutorials
- Respect responsible disclosure timelines

## 🏆 Recognition

All contributors will be:

- Listed in the repository's contributors page
- Mentioned in release notes (for significant contributions)
- Part of making web security more accessible

Thank you for helping make the web more secure! 🔐

## 📬 Questions?

Not sure about something? Feel free to:

- Open a discussion on GitHub Discussions
- Ask in an issue with the `question` label
- Check existing issues for similar questions

---

**Remember**: Quality over quantity. One well-documented module is better than ten incomplete ones.

Happy contributing! 🚀
