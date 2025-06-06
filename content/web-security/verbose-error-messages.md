---
title: Verbose Error Messages
draft: false
date: 2025-01-01
tags:
  - owasp
  - verbose_error_messages
  - information_disclosure
  - error_handling
  - security_misconfiguration
---

Verbose Error Messages happen when a program shows too much information about what's going on inside when something goes wrong. This can include details about how it's set up, how it works, or even the way the database is organized. While it's helpful to have detailed error messages when you're creating a program, it's not a good idea to leave them on when the program is being used by real people. This is because these messages can show attackers important details like file paths, specific commands, or setup information. Attackers can use this information to find weaknesses in the program or figure out how to cause more problems.

Too much detail in error messages can be caused by the way the program is set up by default, errors that aren't handled properly, or tools that aren't designed for real-world use. To prevent information from leaking, it's important to make sure that the error messages people see are simple and don't give away too much information. At the same time, you should still keep detailed records of errors in a safe place where only authorized people can see them.

For example, a program might show a full stack trace (a list of the steps the program was taking when it crashed) to the user's browser. This can reveal things like class names, line numbers, and library versions. Attackers can use this information to figure out what framework the program is using, how the files are organized, or what part of the program is vulnerable.

If a SQL query fails (SQL is a language used to talk to databases), the program might show a detailed message like:

```sql
SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in
your SQL syntax near 'FROM users WHERE id= ' at line 1
```

This shows the attacker how the query is structured, including table names and SQL commands. This can help them figure out how to inject malicious code into the query.

In some cases, the program might show file system paths or server configuration details, like `/var/www/myapp/config.php`. Attackers can use these paths to look for specific files or learn more about the server's environment.

Here are some ways to prevent verbose error messages from causing problems:

1. **Customize and Restrict Error Messages**
- Show simple, user-friendly error messages that don't give away technical details when the program is being used by real people.
- Only show high-level information, like "An unexpected error has occurred" or "Unable to process your request."
1. **Secure Exception Handling**
- Use special handlers or tools that catch errors and control how they are shown to users.
- Keep detailed records of errors internally, but don't show them to the public.
1. **Use Different Configurations for Development and Production**
- When using frameworks like Django, Rails, or Express, make sure that debug settings are turned off when the program is being used by real people.
- Production mode usually hides detailed error messages and stack traces by default.
