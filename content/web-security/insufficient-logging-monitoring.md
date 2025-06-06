---
title: Insufficient Logging and Monitoring
draft: false
date: 2025-01-01
tags:
  - owasp
  - logging
  - monitoring
  - incident_response
  - audit_trails
  - siem
---

Insufficient Logging and Monitoring happens when an application doesn't properly record, store, or check security events. This makes it hard to find and stop attacks, fraud, data leaks, or bad actions. Without good logging, attackers can stay hidden for a long time, possibly stealing important data or getting more power without anyone noticing.

If monitoring isn't good enough, alerts for things like brute-force attacks, getting unauthorized access, or misusing APIs might be late or missing. Even if logs are recorded, they're not helpful for solving problems if they're not protected from being changed, stored safely, and checked regularly.

An application that doesn't log when someone tries to log in, whether they succeed or fail, allows attackers to keep trying different passwords without being caught.

```http
POST /login

username=admin&password=wrongpassword
```

No log entry is created, so it's impossible to detect repeated failed login attempts.

If an application doesn't log what users with special permissions do, an attacker or someone on the inside might change account roles, settings, or delete data without being noticed.

Example: An admin creates a new user with superuser permissions, but this event isn't logged.

APIs that handle money transactions, password changes, or login tokens should log what happens. Without this, an attacker can move money, change passwords, or change requests without being detected.

```http
POST /update-balance

{ "user": "attacker", "balance": "9999999" }
```

If the API doesn't log this request, systems that detect fraud can't flag it.

Even if logs are created, not actively checking them allows attacks to go unnoticed in real-time. Without automatic alerts, security teams have to manually look through logs, which is often too late.

1. **Implement Comprehensive Logging**
- Log all login events (successful logins, failed attempts, password resets).
- Record actions by users with special permissions (admin access, permission changes, money transactions).
- Include logs of API activity for important actions.
1. **Use Secure and Tamper-Proof Log Storage**
- Store logs in a way that prevents attackers from deleting their activity.
- Use methods like cryptographic signing to prevent log tampering.
1. **Enable Real-Time Monitoring and Alerts**
- Use systems like Splunk, ELK Stack, or Wazuh to monitor logs.
- Set up alerts for suspicious activity (e.g., repeated failed logins, getting unauthorized access, unusual API requests).
1. **Mask or Encrypt Sensitive Data in Logs**
- Avoid logging passwords, API keys, or personal data in plain text.

- Example of secure logging:

```text
 [INFO] User login attempt: user=admin, IP=192.168.1.10, status=FAILED
```

- Example of insecure logging:

```text
 [DEBUG] User login: username=admin, password=admin123
```

1. **Regularly Review and Audit Logs**
- Check logs regularly to find unusual activity.
- Use machine learning to find patterns that indicate a problem.
1. **Ensure Log Retention Policies**
- Keep logs for 6-12 months to help with investigations.
- Use log rotation and archiving to manage storage space.
