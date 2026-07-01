---
title: Code Injection
draft: false
date: 2025-01-01
tags:
  - owasp
  - code_injection
  - injection
  - remote_code_execution
---

Code Injection is a significant security vulnerability that occurs when an attacker can send malicious code or input to an application, and the application executes it as part of its own code. This typically happens when user input is processed without proper validation or sanitization. If successful, an attacker could gain control over the application or the underlying system.

Unlike SQL Injection (which targets databases) or Command Injection (which targets operating system commands), Code Injection specifically targets the application's source code language, such as Python or PHP. If the application executes this injected code, attackers can perform unauthorized actions, including accessing sensitive data.

One common way this vulnerability occurs is through the misuse of functions that evaluate or execute code from strings, such as `eval()`:

```php
<?php
    // Insecure PHP snippet
    $userInput = $_GET['data'];
    eval("\$variable = $userInput;");
?>
```

If an attacker provides input like this:

`?data=system('cat /etc/passwd');`

the `eval()` function will attempt to execute `system('cat /etc/passwd');` as PHP code. If the server configuration allows it, this could grant the attacker the ability to execute arbitrary commands and access sensitive files.

Certain programming languages (like Java, PHP, and Python) support serialization/deserialization, which is a way to convert complex data structures into a format that can be easily stored or transmitted. If an application deserializes untrusted user-provided data without proper validation, an attacker could inject malicious code. For example, in PHP:

```php
<?php
    // Insecure example of unserializing user data
    $serializedData = $_POST['serialized'];
    $object = unserialize($serializedData);
    // Potentially triggers malicious constructors or methods
?>
```

If the serialized data contains malicious code, deserializing it can trigger the execution of that code within the application's context.

Web applications often use templating engines to generate dynamic HTML content. If user input is directly embedded into a template without proper escaping or sanitization, attackers can inject template syntax that executes code on the server. For example, a vulnerable Python Flask application using Jinja2:

```php
# Vulnerable Python with Jinja2
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route('/')
def index():
    user_input = request.args.get('data')
    template = f"Hello {user_input}!"
    return render_template_string(template)
```

If the application doesn't validate user input, an attacker could provide input that includes template expressions, such as:

`/?data={{7*7}}` or `/?data={{\'\'.__class__.__mro__[1].__subclasses__()%}...`

This could cause the Jinja2 template engine to execute arbitrary Python code on the server.

Here's how to prevent Code Injection:

1.  **Avoid Evaluating User-Supplied Code**
    - Avoid using functions like `eval()`, `exec()`, or `unserialize()` with user-provided input.
    - If absolutely necessary, perform strict input validation and use secure sandboxing techniques.
2.  **Handle Serialization Securely**
    - Never deserialize data from untrusted sources.
    - If deserialization is required, use secure formats (like JSON) and validate the data structure rigorously.
    - Utilize libraries designed for secure deserialization.
3.  **Use Secure Templating Engines**
    - Employ templating engines that provide automatic context-aware escaping.
    - Ensure templates are configured securely and do not expose dangerous functions or variables to user input.
4.  **Implement Strict Input Validation and Sanitization**
    - Treat all user input as potentially malicious.
    - Validate input against expected formats and data types.
    - Sanitize or encode input appropriately for the context where it will be used (e.g., HTML, JavaScript, template).
5.  **Apply Principle of Least Privilege**
    - Run applications with minimal necessary permissions.
    - Limit the capabilities of the user or process running the application to reduce the potential impact of a successful injection.
