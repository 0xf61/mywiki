---
title: Reflected Cross-Site Scripting (XSS)
draft: false
date: 2025-01-01
tags:
  - owasp
  - xss
  - reflected_xss
  - injection
---

Reflected Cross-Site Scripting (XSS) happens when an attacker puts bad code into a spot that's not protected. This code runs right away in the response without being saved on the server. Unlike stored XSS, where the code stays in the app's files, reflected XSS is temporary. The attacker usually makes a special link or form that the victim has to click or visit.

Because the bad script runs in the victim's browser, it can steal login info, take over accounts, or do things as if it were the victim. Reflected XSS relies on tricking users into clicking a bad link or giving bad data.

An application includes user-submitted input directly into the response. For instance, a search form:

`https://example.com/search?q=someinput`

If the server-side code incorporates `someinput` into the HTML page without proper escaping, an attacker can craft a URL with a malicious script:

`https://example.com/search?q=<script>alert('Reflected-XSS')</script>`

When a victim clicks this link, the browser executes the script in the page context.

If a web form takes user data from a POST request and displays it on the page (e.g., an error message or confirmation) without sanitization, an attacker can submit a malicious payload:

```javascript
<script>alert('Reflected XSS');</script>
```

The response then reflects this script, causing the browser to run it whenever the victim views the result page.

1. **Validate and Sanitize User Input**
- Filter out or neutralize dangerous characters or HTML tags.
- Use well-maintained libraries or frameworks that handle HTML sanitization and escaping for your language of choice.
1. **Encode Output Correctly**
- Escape all dynamic content when rendering in HTML, JavaScript, or other contexts.
- For instance, use HTML encoding for data placed in HTML text nodes, and JavaScript encoding for data placed in scripts.
1. **Implement a Content Security Policy (CSP)**
- Configure `script-src`, `object-src`, and other directives to restrict script execution.
- This adds a strong layer of defense if an XSS vector is discovered.
1. **Use Server-Side Security Libraries and Frameworks**
- If your framework supports auto-escaping or context-sensitive encoding, enable it by default.
- Avoid crafting raw HTML strings by concatenating user input; instead, use templating systems that are XSS-aware.
