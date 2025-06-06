---
title: Local File Inclusion (LFI)
draft: false
date: 2025-01-01
tags:
  - owasp
  - lfi
  - injection
  - file_inclusion
  - path_traversal
---

Local File Inclusion (LFI) is a type of security problem that happens when a website includes files from the server without checking if the user input is safe. Usually, the website gets a file path from the user (like `?page=` in a URL) and uses this path to show content. If the website doesn't properly check this path, attackers can change it to access important files on the server.

The main problem is that user input is used in functions that handle files (like `include`, `require` in PHP, or file reading in other languages) without being checked. Attackers can use special sequences like `../` to access files they shouldn't, such as system logs, configuration files with passwords, or even the website's code.

LFI can become more serious if attackers can include and use files that contain harmful code or user-submitted content. In some cases, LFI can lead to Remote Code Execution (RCE), which means attackers can run code on the server. Even if it's just reading files, it can expose important information, help attackers plan more attacks, and compromise privacy.

```php
// Vulnerable code snippet
<?php
    $page = $_GET['page'];  // For example, ?page=index
    include($page);         // No input validation
?>
```

An attacker could do this by entering:

`?page=../../../../etc/passwd`

to try to read the server's `/etc/passwd` file (if they have permission).

Some websites save user input in server logs. If an attacker can write PHP code into a log (for example, by changing the User-Agent header) and then include that log file using the vulnerable parameter, the PHP code can be run.

Example request:

`GET /vulnerable.php?page=../../../var/log/apache/access.log`

where the log file might have harmful code that the server runs.

- /etc/passwd or /etc/shadow on UNIX systems.
- config.php or wp-config.php in website directories (revealing database passwords).
- Error logs or access logs that may contain other information that can be used or even injected harmful code.

These examples show how an attacker can use unchecked file inclusion to read system files or make the problem worse by injecting files.

1. **Input Validation and Whitelisting**
   - Never trust paths provided by users.
   - Keep a list of allowed file names or paths if you need to include files dynamically. For example, match user-friendly input values ( `?page=help`) to internal, verified file names ( `/path/to/help.php`).
2. **Parameterized Routing / Avoid Direct include**
   - Instead of directly accepting file paths, use a controlled routing system. For example, keep all valid include files in one directory and use a lookup table.
   - If a file must be included, make sure its path is strictly checked (for example, using `realpath` checks or directory checks).
3. **Least Privileges and Hardened Server Configuration**
   - Limit file system permissions so that the website user only has the necessary access. This reduces the impact if a problem is exploited.
   - Disable risky functions (like `allow_url_include` or even `allow_include` in some configurations) in the PHP settings when they are not needed.
   - Consider using `open_basedir` restrictions in PHP to limit file operations to specific, safe directories.
4. **Filtering and Encoding**
   - Remove or encode special characters from user input (like `../`) that allow path traversal.
   - In some cases, using strong filtering can reduce the risk of LFI attacks, but whitelisting is usually more secure than blacklisting.
