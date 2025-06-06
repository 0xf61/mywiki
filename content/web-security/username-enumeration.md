---
title: Username Enumeration
draft: false
date: 2025-01-01
tags:
  - owasp
  - username_enumeration
  - authentication
  - reconnaissance
  - information_disclosure
---

Username Enumeration happens when someone tries to find out if a username is real on an application by looking at how the system responds.

Applications often show they have this problem through login forms, password reset pages, registration checks, and API responses. If an application gives different messages or takes different amounts of time to respond depending on whether a username is real, someone can use this to find out real usernames before attacking.

A login form that has this problem might give different messages depending on whether the username is real:

```http
POST /login

username=admin&password=wrongpassword
```

**Response:**

```json
"Invalid password."
```

(This means "admin" is a real username)

```http
POST /login

username=notrealuser&password=wrongpassword
```

**Response:**

```json
"User does not exist."
```

(This means "notrealuser" is not a real username)

People can use this to make a list of real usernames.

If the password reset feature gives away username information, someone can try email addresses or usernames:

```http
POST /reset-password

email=user@example.com
```

**Responses:**

- "Password reset link sent to your email" → (This means the email is real)
- "No account found with this email" → (This means the email is not real)

Even if the error messages are the same, how long the server takes to respond can show if a username is real. For example:

- Real username: Response time 250ms
- Not real username: Response time 50ms

People can measure these times and guess which usernames are real.

1. **Use the Same Error Messages**
- Make sure that when you log in or reset your password, the messages don't show whether a username is real or not.
- Use the same message for everything:
    - "Invalid login information."
    - "If the account is real, you will get a password reset email."
1. **Make Response Times the Same**
- Make sure that logging in and account-related requests take the same amount of time, no matter if the username is real or not. This stops people from timing the responses.
1. **Limit Attempts and Watch**
- Only allow a certain number of login and reset attempts per IP address or session (like 5 attempts per minute).
- Use Web Application Firewalls (WAF) to find and stop people from trying to guess usernames automatically.
1. **Use CAPTCHA on Important Pages**
- Put CAPTCHAs on login, registration, and password reset pages to stop people from guessing usernames automatically.
