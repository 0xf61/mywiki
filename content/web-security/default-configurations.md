---
title: Default Configurations - Security Risk
draft: false
date: 2025-01-01
tags:
  - owasp
  - default_configurations
  - security_misconfiguration
  - hardening
  - default_credentials
---

Default configurations refer to the standard settings, usernames, or functions provided by software or systems when first installed. These standard settings often focus on easy setup but might not be secure enough for real-world use. Attackers take advantage of well-known default usernames, passwords, settings, or open connections to gain unauthorized access or to perform other attacks.

Developers and system managers often forget to change these defaults when setting up systems, leaving important services exposed with easy-to-guess or weak security settings. By using publicly available guides or scanning tools, attackers can quickly find systems running default configurations and easily break into them.

Some content management systems (CMS), routers, or database servers come with usernames and passwords like `admin/admin` or `root/root`. If managers don't quickly change these, attackers can easily log in and take control of the system.

Common services or software might run on their standard connections without needing a password (for example, open database connections or debugging tools). Attackers can scan the network to find these services and attack them if there are no extra security measures in place.

In some web programs, sample pages or tools are enabled by default for demonstration. These sample tools can show debugging information, version details, or even allow privileged actions. If they are still active when the system is used for real work, attackers can explore them for weaknesses.

1. **Change Default Usernames and Passwords Immediately**
- When installing, change all administrator and service accounts to have strong, unique passwords.
- Turn off or remove any standard or guest accounts that are not being used.
1. **Make Configuration Settings More Secure**
- Check and set up each service's security options – turn on password requirements, limit permissions, and use secure ways of communicating.
- Turn off or remove standard "example" programs, sample tools, or test data that are not needed for real work.
1. **Limit Network Access**
- Limit access to important connections by using firewalls, security groups, or network separation.
- Close or change standard connections where possible to hide common attack paths.
1. **Follow Vendor and Community Best Practices**
- Read official guides or trusted community advice on how to secure the specific software or service.
- Stay informed about known standard settings or weaknesses and use recommended fixes or updates.
