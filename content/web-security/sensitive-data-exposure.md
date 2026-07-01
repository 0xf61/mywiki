---
title: Sensitive Data Exposure
draft: false
date: 2025-01-01
tags:
  - owasp
  - sensitive_data_exposure
  - encryption
  - cryptographic_failures
  - data_protection
  - tls
  - ssl
---

Sensitive Data Exposure happens when a program accidentally shows secret or personal information, like passwords, credit card numbers, health records, or private business information. This can happen if data isn't properly scrambled (or isn't scrambled at all), if data is stored in unsafe places, or if not enough rules are in place to control who can see the data. Attackers can use these weak spots to get into data while it's being sent (like through unsafe internet connections) or while it's stored (like in unprotected computer files).

If secret data is shown, it can lead to identity theft, money being stolen, fines from the government, and damage to a company's reputation. Common reasons for this include not using secure internet connections, keeping passwords as plain text, or using weak ways to scramble data.

If a website sends login information over a regular internet connection instead of a secure one, someone can secretly watch the data being sent and see the username and password.

Some programs keep user passwords directly in a computer file without scrambling them. If someone gets into that file, they can see everyone's password. This also puts users at risk if they use the same passwords on different websites.

Sometimes, programs include secret codes or keys in website addresses. These codes can be seen in computer logs, website history, or in other places, allowing unintended people to see them.

Even if data is scrambled, using old or broken methods can make it easy for attackers to unscramble the data.

1. **Use Strong Encryption (Transport Layer Security)**
   - Always use secure internet connections for important pages (like login or account settings).
   - Use the latest secure connection methods to protect data from being watched or changed while it's being sent.
2. **Encrypt Sensitive Data at Rest**
   - Store passwords using special scrambling methods that can't be easily reversed (like bcrypt, Argon2, or scrypt).
   - For other secret data (like financial or health records), use strong scrambling methods (like AES-256) with safe ways to manage the secret keys.
3. **Avoid Storing Tokens in Logs or URLs**
   - Don't put secret codes or keys in website addresses. Instead, put them in secure parts of the internet message.
   - Make sure secret data is hidden or left out of computer logs, especially if other people might see them.
4. **Regularly Update Cryptographic Measures**
   - Stop using weak or outdated scrambling methods.
   - Stay informed about new security problems and quickly fix or update your systems.
5. **Implement Strict Access Controls**
   - Only allow authorized people and programs to access computer files.
   - Only give people the minimum access they need to do their jobs.
