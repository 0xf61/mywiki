---
title: DOM-based Cross-Site Scripting (XSS)
draft: false
date: 2025-01-01
tags:
  - owasp
  - dom_based_xss
  - xss
  - injection
  - client_side
---

DOM-based Cross-Site Scripting (XSS) is a type of XSS where the problem happens in the website code inside the browser. This means the bad stuff happens without sending the harmful information to the website's server. In DOM-based XSS, the problem starts when the website's code (like JavaScript) reads or writes information using unsafe methods (such as `document.location`, `document.write`, or `innerHTML`) with information that might be dangerous. Because of this, bad people can change the browser to put in and run bad code directly.

Because the bad code never gets to the server (or the server doesn't see it as dangerous), regular safety checks on the server might not find or stop it. DOM-based XSS can be tricky to find and fix if the people who make the website don't check the website's code carefully.

Imagine some code that takes a piece of information from the website address and shows it on the page:

```

// Example of unsafe code
let userParam = new URLSearchParams(window.location.search).get('text');
document.getElementById('output').innerHTML = userParam;

```

If a bad person makes a website address like this:

`https://example.com/page?text=Saying hello!`

the code will put that message directly onto the page, which could be a problem if the message is harmful.

On some websites, the website saves information in the part of the address that starts with a '#'. If the website's code puts this information directly onto the page, a bad person can put bad code in that part of the address:

```

// Reading window.location.hash and directly showing it
let hashContent = window.location.hash.substring(1); // e.g. '#Saying hello!'
document.getElementById('hashOutput').innerHTML = decodeURIComponent(hashContent);

```

If someone visits a link with a bad message (like `https://example.com/#Saying hello!`), the bad code could run.

1. **Safe Ways to Change the Page**
- Use code that automatically treats what people type as regular text, not as special instructions. For example, use `textContent` instead of `innerHTML`.
- Try not to put HTML code directly onto the page. If you have to, use special tools to clean the code and remove anything dangerous.
1. **Clean and Safe Code**
- When you put information on the page, make sure it's cleaned and made safe for where it's going.
- For example, if you're putting it in an HTML page, clean it to stop bad code from running.
1. **Check What People Type**
- Even though DOM-based XSS avoids the server, checking and limiting what people can type in the website address can still help stop bad things from happening.
- Use tools to find and remove any bad letters or code.
1. **Website Safety Rules (CSP)**
- Setting up clear website safety rules can lower the risk of bad code getting in, even if there are some DOM-based XSS problems.
- For example, don't allow code to be written directly on the page, and only allow code from trusted sources to limit the damage from bad code.

*Reference: OWASP Top 10 Security Risks*
