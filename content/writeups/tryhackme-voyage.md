---
title: "TryHackMe: Voyage"
date: 2025-09-01
description: "Voyage started with exploiting a vulnerability in Joomla CMS to leak its configuration and obtain a set of credentials for SSH access."
categories: ["TryHackMe"]
tags: ["web", "joomla", "vulnerability", "ssh", "privilege-escalation"]
image: "/images/writeups/voyage-thumb.jpg"
difficulty: "Medium"
platform: "TryHackMe"
author: "Adithya Kallaje"
---

# TryHackMe: Voyage Writeup

![Voyage Banner](/images/writeups/voyage-banner.jpg)

## Overview

Voyage is a medium-difficulty room on TryHackMe that focuses on exploiting a Joomla CMS vulnerability to gain initial access and then escalating privileges on a Linux system.

## Reconnaissance

### Nmap Scan
```bash
nmap -sC -sV -oN initial.nmap 10.10.x.x
```

The scan revealed:
- Port 22 (SSH)
- Port 80 (HTTP - Joomla CMS)

### Web Enumeration
```bash
gobuster dir -u http://10.10.x.x -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

## Initial Access

### Joomla Vulnerability Exploitation

The target was running an outdated version of Joomla with a known configuration disclosure vulnerability.

```bash
# Exploit the vulnerability
curl "http://10.10.x.x/administrator/components/com_admin/sql_scripts/joomla.sql"
```

This revealed database credentials that were reused for SSH access.

## Privilege Escalation

After gaining initial access as a low-privileged user, I discovered:

1. **SUID Binary Analysis**
2. **Sudo Privileges Check**
3. **Kernel Version Enumeration**

### Exploitation Steps

```bash
# Check for SUID binaries
find / -perm -4000 2>/dev/null

# Check sudo privileges
sudo -l
```

## Lessons Learned

- Always check for configuration disclosure vulnerabilities in CMS platforms
- Credential reuse is a common security issue
- Proper enumeration is key to finding privilege escalation vectors

## Tools Used

- Nmap
- Gobuster
- Burp Suite
- LinPEAS

---

**Difficulty**: Medium  
**Platform**: TryHackMe  
**Completion Date**: September 1, 2025
