---
title: Stack Traces - Information Disclosure
draft: false
date: 2025-01-01
tags:
  - owasp
  - stack_traces
  - error_handling
  - information_disclosure
  - verbose_errors
  - security_misconfiguration
---

Stack traces are like a detailed record of what a program was doing when it had a problem or error. When making a program, this record is super helpful to find and fix the problem. It shows which parts of the program were running, where the error happened, and sometimes what tools or helpers the program was using.

But, if a program shows these records to everyone, like when it's running online, it can be risky. Attackers can see important details about how the program and the server it's on are set up. They might find out where files are stored, how the database is organized, or what helpers the program uses. This information can help them plan attacks, find weak spots, or sneak into the system.

Showing stack traces often happens because of mistakes in how the program handles errors, or because special debugging settings are left on when they shouldn't be. It's important to hide these records from regular users, but still, keep them safe for the people who fix the program.

Imagine a program made with Java has a problem called "NullPointerException." This problem isn't handled correctly, so the program shows a default error page that looks like this:

```java
java.lang.NullPointerException
    at com.example.app.UserService.getUserById(UserService.java:45)
    at com.example.app.UserController.handleRequest(UserController.java:67)
    ...
```

This shows the names of the program's parts, like "UserService" and "UserController," and where they are located. Attackers can learn about the program's structure and find parts that might have known weaknesses.

Or, imagine a program made with Flask shows a detailed record of a problem, including information about the environment it's running in:

```python
Traceback (most recent call last):
  File "/path/to/flask/app.py", line 200, in create_user
    user = User(name=request.form['username'])
KeyError: 'username'
```

Besides showing where the problem happened, this record might also show what versions of Python, Flask, or other helpers the program is using. This helps attackers find unpatched weaknesses in those helpers.

Sometimes, stack traces even include secret information like environment variables or connection strings. For example, a problem with connecting to the database might show the full address, username, or part of the password.

Here are some ways to prevent showing too much information:

1. **Use Proper Error Handling**

- Turn off special debugging features when the program is running for real. Most frameworks have settings to hide stack traces from users when the program is live.

1. **Create Custom Error Pages**

- Catch all problems in the program and handle them.
- Show only simple error messages to users, like "An error occurred" or "Something went wrong."

1. **Keep Logs Private**

- Store detailed records of problems in log files or special logging systems.
- Make sure only authorized people can access these logs.
