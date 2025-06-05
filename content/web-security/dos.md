---
title: Denial of Service (DoS) Attacks
draft: false
date: 2025-01-01
tags:
  - owasp
  - dos
  - ddos
  - availability
  - rate_limiting
---

A Denial of Service (DoS) attack is a malicious attempt to disrupt the normal traffic of a targeted server, service, or network by overwhelming the target or its surrounding infrastructure with a flood of internet traffic. DoS attacks accomplish this disruption by using multiple compromised computer systems to generate attack traffic, making legitimate requests unable to be processed.

**Attack Categories:**

**Volume-Based Attacks:** Attempt to consume bandwidth between the target and the larger internet
- **UDP Floods:** Send large numbers of UDP packets to random ports
- **ICMP Floods:** Overwhelm target with ICMP Echo Request (ping) packets
- **Amplification Attacks:** Exploit UDP protocols (DNS, NTP, SSDP) to amplify attack traffic

**Protocol Attacks:** Exploit weaknesses in Layer 3 and Layer 4 protocols
- **SYN Floods:** Consume server connection state tables
- **Ping of Death:** Send malformed packets larger than maximum allowed size
- **Smurf Attacks:** Use ICMP broadcasts with spoofed source addresses

**Application Layer Attacks:** Target web applications with seemingly legitimate requests
- **HTTP Floods:** Overwhelm web servers with HTTP GET/POST requests
- **Slowloris:** Keep connections open by sending partial HTTP requests
- **RUDY (R-U-Dead-Yet):** Send slow POST requests to exhaust server resources

**Common Attack Vectors:**

**Traffic Flooding:**
```bash
# Example of volumetric attack using hping3
hping3 -S --flood -V -p 80 target.com
```

**Resource Exhaustion:**
```http
GET /search?query=complex_regex_pattern HTTP/1.1
Host: target.com
```
Attackers target computationally expensive operations like:
- Complex database queries with no LIMIT clauses
- Image/video processing endpoints
- Cryptographic operations (password hashing)
- Regular expression processing (ReDoS)

**Connection Exhaustion (Slowloris):**
```http
GET / HTTP/1.1
Host: target.com
User-Agent: Mozilla/4.0
Content-Length: 42
X-a: b
```
The attack sends partial HTTP headers, keeping connections open indefinitely.

**Application Logic Abuse:**
```http
POST /expensive-operation HTTP/1.1
Content-Type: application/json

{"data": "large_payload_requiring_heavy_processing"}
```

## Comprehensive Defense Strategies

### 1. **Rate Limiting and Traffic Shaping**
```nginx
# Nginx rate limiting configuration
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;

    server {
        location /api/ {
            limit_req zone=api burst=20 nodelay;
        }
        location /login {
            limit_req zone=login burst=5;
        }
    }
}
```

### 2. **Content Delivery Network (CDN) Protection**
```yaml
# Cloudflare configuration example
security_level: "high"
ddos_protection: "enabled"
rate_limiting:
  requests_per_minute: 1000
  burst_size: 50
bot_fight_mode: "enabled"
```

### 3. **Resource Management and Circuit Breakers**
```python
# Python circuit breaker pattern
import time
from functools import wraps

def circuit_breaker(failure_threshold=5, recovery_timeout=60):
    def decorator(func):
        func.failure_count = 0
        func.last_failure_time = None

        @wraps(func)
        def wrapper(*args, **kwargs):
            if func.failure_count >= failure_threshold:
                if time.time() - func.last_failure_time < recovery_timeout:
                    raise Exception("Circuit breaker is open")
                else:
                    func.failure_count = 0

            try:
                result = func(*args, **kwargs)
                func.failure_count = 0
                return result
            except Exception as e:
                func.failure_count += 1
                func.last_failure_time = time.time()
                raise e

        return wrapper
    return decorator
```

### 4. **Web Application Firewall (WAF) Rules**
```yaml
# ModSecurity rules for DoS protection
SecRule REQUEST_HEADERS:User-Agent "@pmFromFile suspicious-agents.txt" \
    "id:1001,phase:1,block,msg:'Suspicious User Agent'"

SecRule IP:REQUEST_COUNT "@gt 100" \
    "id:1002,phase:1,block,expirevar:IP.request_count=60"

SecRule REQUEST_URI "@detectSQLi" \
    "id:1003,phase:2,block,msg:'SQL Injection Attack'"
```

### 5. **Infrastructure Scaling and Load Balancing**
```yaml
# Kubernetes HPA configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 100
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 6. **Advanced Detection and Mitigation**
```python
# Anomaly detection for traffic patterns
def detect_dos_attack(request_logs):
    import numpy as np
    from collections import defaultdict

    ip_requests = defaultdict(list)
    current_time = time.time()

    for log in request_logs:
        if current_time - log.timestamp < 300:  # 5-minute window
            ip_requests[log.ip].append(log.timestamp)

    for ip, timestamps in ip_requests.items():
        if len(timestamps) > 1000:  # More than 1000 requests in 5 minutes
            block_ip(ip)
            alert_security_team(f"Potential DoS from {ip}")
```

*Reference: OWASP Top 10 Security Risks*
