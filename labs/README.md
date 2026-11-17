# 🧪 Practice Labs - Setup Guide

## Overview

This guide will help you set up safe, legal environments for practicing web application security testing. All labs listed here are intentionally vulnerable applications designed for educational purposes.

## ⚠️ Safety Warnings

### NEVER:
- Deploy these applications on public networks
- Use default credentials in production
- Leave vulnerable applications running when not in use
- Test techniques on unauthorized systems

### ALWAYS:
- Run labs in isolated environments (Docker, VMs)
- Use localhost or private networks only
- Shut down containers/VMs after practice
- Follow responsible disclosure for any bugs found in lab platforms themselves

## 📋 Prerequisites

### Required Software

1. **Docker Desktop** (Recommended)
   - Download: https://www.docker.com/products/docker-desktop
   - Simplest way to run vulnerable applications
   - Easy cleanup and isolation

2. **Burp Suite Community Edition**
   - Download: https://portswigger.net/burp/communitydownload
   - Essential for intercepting HTTP traffic
   - Configure browser proxy: 127.0.0.1:8080

3. **Web Browser**
   - Firefox or Chrome recommended
   - Install FoxyProxy extension for easy proxy switching

### Optional Tools
- **VirtualBox/VMware**: For VM-based labs
- **Kali Linux**: Pre-installed security tools
- **OWASP ZAP**: Alternative to Burp Suite

## 🎯 Lab Platforms

### 1. DVWA (Damn Vulnerable Web Application)

**Difficulty**: Beginner-friendly
**Focus**: OWASP Top 10 basics

```bash
# Quick start with Docker
docker run -d -p 80:80 vulnerables/web-dvwa

# Access at: http://localhost
# Default credentials: admin / password
```

**Features**:
- Multiple security levels (Low, Medium, High, Impossible)
- Clear difficulty progression
- Covers XSS, SQLi, CSRF, File Upload, etc.

📖 **[DVWA Practice Guide →](dvwa.md)**

### 2. OWASP Juice Shop

**Difficulty**: Beginner to Advanced
**Focus**: Modern web application vulnerabilities

```bash
# Docker installation
docker pull bkimminich/juice-shop
docker run -d -p 3000:3000 bkimminich/juice-shop

# Access at: http://localhost:3000
```

**Features**:
- 100+ challenges
- Real-world modern application
- Scoreboard and hints system
- API, authentication, business logic flaws

📖 **[Juice Shop Challenge Guide →](juice-shop.md)**

### 3. PortSwigger Web Security Academy

**Difficulty**: All levels
**Focus**: Professional-grade vulnerabilities

**Access**: https://portswigger.net/web-security

**Features**:
- Cloud-hosted labs (no installation needed)
- Covers advanced topics
- Video explanations
- Official Burp Suite integration

📖 **[PortSwigger Lab Solutions →](portswigger.md)**

### 4. WebGoat

**Difficulty**: Beginner to Intermediate
**Focus**: Interactive security lessons

```bash
# Docker installation
docker run -d -p 8080:8080 -p 9090:9090 webgoat/webgoat

# Access at: http://localhost:8080/WebGoat
```

**Features**:
- Guided lessons with hints
- Step-by-step challenges
- Covers OWASP Top 10

📖 **[WebGoat Lesson Guide →](webgoat.md)**

## 🔧 Setup Instructions

### Docker-Based Setup (Recommended)

#### Step 1: Install Docker
```bash
# Verify installation
docker --version
docker ps
```

#### Step 2: Pull and Run Labs
```bash
# DVWA
docker run -d --name dvwa -p 80:80 vulnerables/web-dvwa

# Juice Shop
docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop

# WebGoat
docker run -d --name webgoat -p 8080:8080 webgoat/webgoat
```

#### Step 3: Access Applications
- DVWA: http://localhost
- Juice Shop: http://localhost:3000
- WebGoat: http://localhost:8080/WebGoat

#### Step 4: Stop Labs When Done
```bash
# Stop all containers
docker stop dvwa juice-shop webgoat

# Remove containers (data will be lost)
docker rm dvwa juice-shop webgoat
```

### VM-Based Setup

If you prefer virtual machines:

1. Download VirtualBox/VMware
2. Download vulnerable VM images:
   - Metasploitable 2/3
   - OWASP Broken Web Apps
   - bWAPP
3. Import OVA files
4. Configure host-only or NAT networking
5. Access applications via VM IP

## 🎓 Learning Path

### Week 1: Foundations
1. Set up DVWA (Low security)
2. Complete basic XSS challenges
3. Try SQL injection exercises
4. Practice with CSRF

### Week 2: Intermediate
1. DVWA Medium security
2. Start Juice Shop challenges
3. Begin PortSwigger Academy
4. Document findings in reports

### Week 3: Advanced
1. DVWA High security
2. Advanced Juice Shop challenges
3. PortSwigger advanced labs
4. Practice complete assessment workflow

## 📝 Best Practices

### During Testing
- Take detailed notes
- Screenshot your findings
- Document payloads that work
- Try multiple approaches
- Understand WHY exploits work

### After Testing
- Write up findings using templates
- Practice report writing
- Share knowledge with community
- Try remediation challenges

## 🔗 Additional Resources

### More Practice Platforms
- **HackTheBox**: https://www.hackthebox.com
- **TryHackMe**: https://tryhackme.com
- **PentesterLab**: https://pentesterlab.com
- **HackerOne CTF**: https://ctf.hacker101.com

### Bug Bounty Platforms (for experienced testers)
- **HackerOne**: https://www.hackerone.com
- **Bugcrowd**: https://www.bugcrowd.com
- **Intigriti**: https://www.intigriti.com
- **YesWeHack**: https://www.yeswehack.com

## 🆘 Troubleshooting

### Docker Issues
```bash
# Container won't start
docker logs <container_name>

# Port already in use
docker ps  # Find conflicting container
docker stop <conflicting_container>

# Reset everything
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

### Burp Suite Issues
- **Can't intercept traffic**: Check browser proxy settings
- **Certificate errors**: Install Burp CA certificate
- **Slow performance**: Disable unnecessary extensions

### Application Issues
- **Can't login**: Check default credentials documentation
- **Features not working**: Try different security level
- **Database errors**: Reset application (restart container)

## 📧 Support

If you encounter issues with this guide:
- Open an issue: https://github.com/stephanas-code/WebApp-VAPT-PlayBook/issues
- Check existing documentation for each platform
- Join security communities for help

## ⚖️ Legal Reminder

These labs are legal and safe for practice. However:
- Only test systems you own or have permission to test
- Never use these techniques on production systems
- Respect bug bounty program scopes
- Always obtain written authorization for penetration tests

---

**Ready to start?** Choose a lab and begin your hands-on journey:
- 🎯 [DVWA Exercises](dvwa.md) - Start here if you're new
- 🧃 [Juice Shop Tasks](juice-shop.md) - Modern application challenges  
- 🎓 [PortSwigger Labs](portswigger.md) - Professional-grade practice
- 🐐 [WebGoat Lessons](webgoat.md) - Interactive guided learning

Happy (ethical) hacking! 🔐
