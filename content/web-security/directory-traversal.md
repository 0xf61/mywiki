---
title: Directory Traversal (Path Traversal)
draft: false
date: 2025-01-01
tags:
  - owasp
  - directory_traversal
  - path_traversal
  - file_inclusion
  - injection
---

Directory Traversal (also called Path Traversal) is a security problem that lets attackers get to files or folders outside where they should be able to in a website's file system. This usually happens when the website doesn't check or clean user input that says which file path to use. Attackers use special characters (like `../`) to go up the folder tree and find important system files or website data.

Directory Traversal is often seen when websites let users download or look at files by using a file name or path as a setting. If the website just adds the user's input to a main folder without checking it, attackers can change the path to get out of the folder structure. This can let them read server files they shouldn't, find passwords, or cause more problems on the computer.

Directory Traversal lets attackers get to any files by going outside the folders they should be in (like `/etc/passwd`). Local File Inclusion (LFI) lets them add local files to websites, which can let them run code. Both can show sensitive data, but LFI can be more dangerous if used to run code.

A website might let users pick a file name using a URL setting:

`https://example.com/getFile?name=report.pdf`

If the server code adds `name` to a folder path, like `"/var/www/files/" + name`, and doesn't clean the input, an attacker could send:

`https://example.com/getFile?name=../../etc/passwd`

This might show the content of `/etc/passwd` (if it's allowed), giving sensitive information about user accounts on the server.

On Windows servers, directory traversal often uses backslashes (`..\`) instead of forward slashes. For example:

`https://example.com/getFile?name=..\\..\\Windows\\System32\\config\\SAM`

This could show important system data under certain conditions.

Directory Traversal problems can sometimes be used with other attacks:

- **Local File Inclusion (LFI):** An attacker can use path traversal in LFI to add sensitive files to the website's output or run scripts.
- **Log File Poisoning:** If a website lets attackers change file paths and logs, they can put bad content in logs and then get or run that content using directory traversal.

1. **Check and Clean Input Carefully**
   - Remove or change any directory traversal sequences (like `../` or `..\`) from user inputs.
   - Only allow letters, numbers, and approved file types in file names.
2. **Use Safe File Handling Methods**
   - Use server-side code that uses a set file directory or keeps allowed file references in a safe place.
   - Don't use raw user input directly in file system calls. Instead, match user-requested file names to safe internal paths.
3. **Only Give Necessary Permissions**
   - Run the website with only the necessary permissions.
   - Set up your web server and file system so the website can only get to the folders it needs. For example, use things like chroot jails, SELinux policies, or Docker containers to limit the website's file system access.
4. **Use Built-In Security Features**
   - If your programming language or framework has file handling functions with path normalization or sandboxing, use them.
   - For example, in Java, `java.nio.file.Files` and `java.nio.file.Paths` can help normalize paths and lower the risk of directory traversal.
