---
title: Session Fixation
draft: false
date: 2025-01-01
tags:
  - owasp
  - session_fixation
  - web_vulnerability
  - session_management
  - authentication
---

Session Fixation is a security problem where someone can make you use a session ID that they already know. This lets them take over your session after you log in. This can happen if the website doesn't give you a new session ID after you log in. Instead, someone could set a session ID before you log in and then use it again after you log in.

Also, if sessions are still good after you log out, someone who gets your session ID can keep using your account even after you log out. This happens if the website doesn't properly stop the session when you log out, so it's still active.

By using session fixation, attackers can pretend to be you and get into your account to do things they shouldn't or see your personal information.

1. **Attacker makes a session ID:**

```

GET /login
Set-Cookie: JSESSIONID=123456

```

2. **Attacker tricks you into using this session ID**

   - By putting the session ID in a fake link:

```

https://example.com/login;JSESSIONID=123456

```

   - By putting a session ID in a cookie using a trick called Cross-Site Scripting (XSS).
3. **You log in using the attacker's session ID**

   - The session ID stays the same after you log in.
4. **Attacker can now get into your account**

   - Because the session ID is the same before and after you log in, the attacker can use `JSESSIONID=123456` to get into your account.

Some websites don't stop session tokens properly when you log out. If this happens:

1. **You log in and get a session token:**

```

Set-Cookie: sessionid=abcd1234; HttpOnly; Secure

```

2. **Attacker steals the session ID (like by XSS, session fixation, or watching the network).**

3. **You log out, thinking the session is stopped.**

4. **Attacker uses the same session token again after you log out:**

```

GET /dashboard
Cookie: sessionid=abcd1234

```

   - If the server doesn't stop the session properly, the attacker can still get in.

5. **Make a New Session ID After Login**

   - Give a new session ID right after someone logs in to stop session fixation.

   - In PHP:

```php

session_regenerate_id(true);

```

   - In Java (Spring Security):

```java

http.sessionManagement().sessionFixation().newSession();

```

6. **Stop the Session Properly on Logout**

   - Make sure the session is completely stopped when someone logs out:

```php

session_destroy();

```

   - Remove session cookies in HTTP headers:

```http

Set-Cookie: sessionid=deleted; expires=Thu, 01 Jan 1970 00:00:00 GMT; Secure; HttpOnly

```

7. **Set Secure Cookie Attributes**

   - Use HttpOnly, Secure, and SameSite to protect session cookies:

```http

Set-Cookie: JSESSIONID=abcd1234; HttpOnly; Secure; SameSite=Strict

```

8. **Make Sessions Timeout and Expire**

   - Automatically stop sessions that are not being used to prevent hijacking.
   - Stop sessions after a set time (like 30 minutes of not being used).
9. **Stop Sharing Sessions on Different Devices**

   - Use device fingerprinting or IP binding to only let the session be used on the device it started on.

*Reference: OWASP Top 10 Security Risks*
