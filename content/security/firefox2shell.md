---
title: "From Browser to Shell: Exploiting a Windows Server"
draft: false
date: 2025-04-21
tags:
  - privesc
  - security
---

# From App Interface to System Shell: How It Can Happen

Gaining shell access to a Windows server running VMware Horizon VDI, with only a restricted browser as the entry point. This blog post summarizes the clever techniques used to escalate from a browser to a full system shell, as detailed in an insightful article on System Weakness.

## The Scenario

The target was a Windows server where users logged into a Virtual Desktop Infrastructure (VDI) using Active Directory credentials. The only tool available was a locked-down browser, typically used for tasks like checking calendars or viewing task progress. For a red teamer, however, the goal was to break out of this restricted environment and access the underlying operating system.

### Step 1: Reading Server Files

The first breakthrough came by exploiting the browser’s URL handling. The researcher could access local server files by manipulating the protocol part of a URL (e.g., using the file:// protocol), effectively achieving a Local File Inclusion (LFI) vulnerability. This allowed reading sensitive files, but the ultimate goal was a shell.

### Step 2: Creating a File Upload Opportunity

To interact with the operating system, the researcher needed to execute commands. A key insight was leveraging the file picker dialog, which is an OS-level application triggered when uploading files. Instead of searching for an existing upload feature, the researcher dynamically created one using JavaScript:
`document.write('<input type="file">')`

Alternatively, if developer tools were disabled, the same result could be achieved by entering `javascript:document.write('<input type="file">')` directly into the browser’s URL bar on Chromium-based browsers. This opened a file picker window, granting access to OS-level file creation and execution capabilities, provided the directory had write and execute permissions.

### Step 3: Executing a Shell

Using the file picker, the researcher created a .bat file containing the command cmd.exe and executed it, spawning a command shell. However, a potential hurdle was the Windows setting that hides file name extensions, preventing the creation of executable .bat files. This was bypassed by navigating the file explorer to enable the “Show file name extensions” option or by using an alternative method: creating a Microsoft Word document, opening its VBA editor with ALT+F11, and embedding reverse shell code there.

### Key Takeaways

This exploit chain highlights. A restricted browser, often seen as a secure endpoint, can become a gateway to full system compromise if not properly locked down. Key lessons include:

URL Manipulation: Protocols like file:// can expose server files if not restricted.
JavaScript Flexibility: Simple scripts, like file pickers, can create new attack surfaces.
OS Integration: Browser-triggered OS dialogs can bridge the gap to system-level access.

For defenders, this underscores the need to disable unnecessary protocols, restrict JavaScript execution, and enforce strict permissions on VDI environments. For pentesters, it’s a reminder that even the most constrained environments can be exploited with ingenuity.
