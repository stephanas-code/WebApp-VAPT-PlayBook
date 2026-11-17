# 🚀 Getting Started with Web Application Security Testing

Welcome! This guide will help you set up your lab environment and start your journey into web application security testing.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Burp Suite Configuration](#burp-suite-configuration)
- [Practice Lab Installation](#practice-lab-installation)
- [First Steps](#first-steps)
- [Troubleshooting](#troubleshooting)

## 📦 Prerequisites

### Required Knowledge

Before diving in, you should have:

- ✅ Basic understanding of HTTP/HTTPS protocols
- ✅ Familiarity with web browsers and how websites work
- ✅ Basic command-line skills (Terminal/CMD)
- ✅ Understanding of HTML, JavaScript basics (helpful but not required)

### Required Software

- **Operating System**: Windows, macOS, or Linux
- **RAM**: Minimum 4GB (8GB+ recommended)
- **Disk Space**: At least 20GB free
- **Internet Connection**: For downloading tools and updates

## 🛠️ Environment Setup

### Step 1: Install Java

Burp Suite requires Java to run.

#### Windows
```bash
# Download Java JDK from Oracle or use OpenJDK
winget install -e --id Oracle.JDK.18

# Verify installation
java -version
```

#### macOS
```bash
# Using Homebrew
brew install openjdk@17

# Verify installation
java -version
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install default-jdk

# Verify installation
java -version
```

### Step 2: Install Docker (Optional but Recommended)

Docker makes it easy to run vulnerable applications safely.

#### Windows/macOS
1. Download [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Install and start Docker Desktop
3. Verify installation:
   ```bash
   docker --version
   docker ps
   ```

#### Linux
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Verify installation
docker --version
```

## 🔥 Burp Suite Configuration

### Download and Install

1. **Download Burp Suite Community Edition** (free):
   - Visit [PortSwigger Downloads](https://portswigger.net/burp/communitydownload)
   - Choose your operating system
   - Download the installer

2. **Install**:
   - **Windows**: Run the .exe installer
   - **macOS**: Open the .dmg and drag to Applications
   - **Linux**: Make the .sh file executable and run it
     ```bash
     chmod +x burpsuite_community_linux_*.sh
     ./burpsuite_community_linux_*.sh
     ```

3. **First Launch**:
   - Start Burp Suite
   - Choose "Temporary project" for now
   - Use "Burp defaults" configuration

### Browser Configuration

#### Option 1: Firefox (Recommended for Testing)

1. **Install Firefox** (if not already installed)
2. **Configure Proxy**:
   - Open Firefox Settings
   - Search for "proxy"
   - Click "Settings" under "Network Settings"
   - Select "Manual proxy configuration"
   - HTTP Proxy: `127.0.0.1`, Port: `8080`
   - Check "Also use this proxy for HTTPS"
   - Click "OK"

3. **Import Burp CA Certificate**:
   - With Burp running and Firefox configured, visit: `http://burpsuite`
   - Click "CA Certificate" in top-right
   - Save the certificate file
   - In Firefox: Settings → Privacy & Security → Certificates → View Certificates
   - Go to "Authorities" tab → "Import"
   - Select the downloaded certificate
   - Check "Trust this CA to identify websites"
   - Click "OK"

#### Option 2: Chrome/Edge with FoxyProxy Extension

1. **Install FoxyProxy** extension
2. **Add Burp Proxy**:
   - Host: `127.0.0.1`
   - Port: `8080`
   - Protocol: HTTP/HTTPS
3. **Import CA Certificate** (similar to Firefox)

### Verify Burp is Working

1. Start Burp Suite
2. Go to "Proxy" tab → "Intercept"
3. Ensure "Intercept is on"
4. In your configured browser, visit any HTTP website
5. You should see the request appear in Burp
6. Click "Forward" to send the request

✅ **Success!** If you see HTTP traffic in Burp, you're ready to proceed.

## 🧪 Practice Lab Installation

### Option 1: OWASP Juice Shop (Recommended for Beginners)

**Easiest modern vulnerable web application**

```bash
# Pull the Docker image
docker pull bkimminich/juice-shop

# Run Juice Shop
docker run -p 3000:3000 bkimminich/juice-shop

# Access in browser
http://localhost:3000
```

**Features:**
- Modern web application
- 100+ challenges
- Built-in hints and tutorials
- Score tracking

### Option 2: DVWA (Damn Vulnerable Web Application)

**Classic vulnerable application, great for learning**

```bash
# Pull and run DVWA
docker run -d -p 80:80 vulnerables/web-dvwa

# Access in browser
http://localhost

# Default credentials:
# Username: admin
# Password: password
```

**After login:**
1. Click "Create / Reset Database"
2. Log in again
3. Set security level to "Low" (under "DVWA Security")

### Option 3: WebGoat

**Interactive security lessons**

```bash
# Pull and run WebGoat
docker run -p 8080:8080 -p 9090:9090 webgoat/webgoat

# Access in browser
http://localhost:8080/WebGoat

# Register a new account
```

### Option 4: All-in-One Lab Environment

```bash
# Run all three labs simultaneously
docker run -p 3000:3000 bkimminich/juice-shop &
docker run -p 80:80 vulnerables/web-dvwa &
docker run -p 8080:8080 webgoat/webgoat &

# Access:
# Juice Shop: http://localhost:3000
# DVWA: http://localhost
# WebGoat: http://localhost:8080/WebGoat
```

## 🎯 First Steps

### 1. Explore the Lab Application

- Browse through Juice Shop or DVWA
- Try to use the application normally
- Look at different pages and features
- Think like a regular user first

### 2. Observe Traffic in Burp

- Keep Burp running with "Intercept off"
- Browse the lab application
- Watch HTTP requests appear in Burp's "HTTP history"
- Click on requests to see details
- Look at the "Request" and "Response" tabs

### 3. Try Your First Test

**Simple XSS Test:**

1. Find a search box or input field
2. Turn on Burp "Intercept"
3. Enter a basic XSS payload: `<script>alert('XSS')</script>`
4. Submit the form
5. In Burp, forward the request
6. See if an alert box appears

### 4. Follow a Module

Now you're ready to follow structured modules:

1. Start with [XSS Module](modules/xss/README.md)
2. Read the vulnerability explanation
3. Try the lab exercises
4. Use the Burp examples provided
5. Practice the reproduction steps

## 🐛 Troubleshooting

### Burp Not Intercepting Traffic

**Problem**: No requests appearing in Burp

**Solutions**:
- ✅ Verify proxy settings in browser (127.0.0.1:8080)
- ✅ Check Burp is listening on correct port (Proxy → Options)
- ✅ Ensure "Intercept" is on if you want to see requests
- ✅ Try visiting an HTTP site first (not HTTPS)
- ✅ Disable other proxy extensions or VPNs

### Certificate Errors

**Problem**: "Your connection is not secure" errors

**Solutions**:
- ✅ Import Burp's CA certificate (see Browser Configuration)
- ✅ Trust the certificate for identifying websites
- ✅ Restart browser after importing certificate
- ✅ Try a different browser to isolate the issue

### Docker Containers Won't Start

**Problem**: `docker run` command fails

**Solutions**:
- ✅ Ensure Docker Desktop is running
- ✅ Check if port is already in use:
  ```bash
  # Windows
  netstat -ano | findstr :3000
  
  # macOS/Linux
  lsof -i :3000
  ```
- ✅ Try a different port:
  ```bash
  docker run -p 3001:3000 bkimminich/juice-shop
  ```
- ✅ Check Docker logs:
  ```bash
  docker logs <container_name>
  ```

### Juice Shop / DVWA Not Loading

**Problem**: Browser shows "Can't reach this page"

**Solutions**:
- ✅ Wait a minute - containers take time to start
- ✅ Check container is running:
  ```bash
  docker ps
  ```
- ✅ Try `http://localhost:3000` not `https://`
- ✅ Disable Burp proxy temporarily to test
- ✅ Check firewall isn't blocking localhost

### Burp Performance Issues

**Problem**: Burp is slow or unresponsive

**Solutions**:
- ✅ Increase Java heap size:
  ```bash
  # Edit Burp launcher to add:
  java -Xmx2g -jar burpsuite.jar
  ```
- ✅ Clear HTTP history regularly
- ✅ Disable unnecessary extensions
- ✅ Close other resource-intensive applications

## 📚 Additional Resources

### Documentation
- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Docker Documentation](https://docs.docker.com/)

### Video Tutorials
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) - Free video lessons
- [PentesterLab](https://pentesterlab.com/) - Hands-on exercises
- [HackerSploit](https://www.youtube.com/c/HackerSploit) - YouTube channel

### Community Support
- [Burp Suite Support](https://portswigger.net/support)
- [OWASP Slack](https://owasp.org/slack/invite)
- [Reddit r/netsec](https://www.reddit.com/r/netsec/)
- [This Repository's Discussions](https://github.com/stephanas-code/WebApp-VAPT-PlayBook/discussions)

## 🎯 Next Steps

Once your environment is set up:

1. 📚 Read the [Learning Path](learning-path.md) for structured progression
2. 🎓 Start with [XSS Module](modules/xss/README.md)
3. 📝 Practice vulnerability reporting with [Report Templates](reports/vuln-report-template.md)
4. 🧪 Complete lab exercises in [Labs Section](labs/README.md)
5. 🔍 Use [Cheatsheets](cheatsheets/burp-suite.md) as quick reference

---

## ⚠️ Remember

- 🔒 **Only test systems you own** or have explicit permission to test
- 📋 **Take notes** as you learn
- 👥 **Ask questions** in our Discussions
- 🏆 **Practice regularly** - consistency is key
- ⚖️ **Stay ethical** - always follow the law

Happy hacking! 🚀

---

**Need help?** [Open an issue](https://github.com/stephanas-code/WebApp-VAPT-PlayBook/issues) or [start a discussion](https://github.com/stephanas-code/WebApp-VAPT-PlayBook/discussions).
