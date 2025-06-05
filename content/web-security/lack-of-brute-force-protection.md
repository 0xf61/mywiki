---
title: Lack of Brute Force Protection
draft: false
date: 2025-01-01
tags:
  - owasp
  - brute_force
  - protection
  - authentication
  - rate_limiting
  - account_lockout
---

Lack of brute-force protection happens when a program doesn't have ways to stop or find repeated, automatic login tries. This lets attackers guess passwords, PINs, codes, or tokens using tools.

Without things like account lockouts, limits, CAPTCHA, or extra security steps, an attacker can try many password combinations quickly. This makes it easier to get into accounts without permission.

This problem is worse if passwords are weak or stolen passwords are used again, making accounts easier to break into.

An attacker can send many login requests without being stopped:

```

POST /login
username=admin&password=guess123

```

Tools can try many common passwords without being noticed.

If an account isn't locked after many wrong tries, an attacker can keep trying until they get it right.

For systems with short number codes, like 4-digit PINs, not having a delay or limit lets an attacker try all combinations in seconds.

Bots can fill out login forms automatically without problems, helping attacks happen on a large scale.

Attackers use lists of stolen passwords against the login page. If there's no check, they can easily find working username/password pairs.

1. **Set Limits**
- Limit login tries per IP address or account to 3–5 per minute
- Make delays longer after each wrong try
2. **Lock Accounts**
- Lock the account for a while after too many wrong tries (like 5–10)
- Tell users when their account is locked
3. **Use CAPTCHA**
- Add CAPTCHA to login pages after many wrong tries or strange activity
4. **Use Multi-Factor Authentication (MFA)**
- Use MFA to lower the risk of account takeover even if passwords are stolen
5. **Watch for Strange Logins**
- Find login tries from unusual IP addresses or lots of traffic
- Use IP reputation to block bad sources
6. **Find Stolen Passwords**
- Find login tries using known stolen passwords and block them
- Use services to check for reused passwords
7. **Check Login Events**
- Keep track of all login tries, failures, and lockouts
- Look at logs for brute-force patterns

*Reference: OWASP Top 10 Security Risks*
