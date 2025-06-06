---
title: HTTP Security Headers
draft: false
date: 2025-01-01
tags:
  - owasp
  - http_headers
  - security_headers
  - security_misconfiguration
  - hsts
  - csp
  - xss
---

HTTP Headers are very important for web security. They give extra information about what's sent between computers (clients) and servers. If these headers are not set up correctly, or if they are missing, bad things can happen. For example, hackers can use Cross-Site Scripting (XSS), Clickjacking, or Man-in-the-Middle (MitM) attacks to steal information. Setting up HTTP headers the right way makes websites safer by making sure computers talk securely, stopping bad things browsers might do, and fixing common problems that hackers use.

If security headers aren't set up right, hackers can change what you see, put bad scripts on the website, or use weaknesses in your browser to hurt users and steal important information.

The HTTP Strict Transport Security (HSTS) header makes sure browsers only use HTTPS to connect to a website. This stops hackers from making you use an older, less safe version of the site:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

If this header is missing, a hacker can make you visit the HTTP version of the site and steal or change your information.

If a website lets other sites put it inside a box (`<iframe>`), hackers can trick you into clicking hidden things. This is called Clickjacking.

To stop this, use this header:

```http
X-Frame-Options: DENY
```

Without it, a hacker can put the website inside a bad page and trick you into doing things you don't want to do.

Sometimes, browsers try to guess what type of file it is, even if the website says what type it is. This can be dangerous because hackers can trick the browser into running bad scripts.

To stop this, use this header:

```http
X-Content-Type-Options: nosniff
```

Without it, hackers can trick browsers into running files that aren't supposed to be scripts as if they were JavaScript.

If a website doesn't have a Content Security Policy (CSP), hackers can put bad scripts on the site using Cross-Site Scripting (XSS).

A good CSP header looks like this:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-random123'; object-src 'none'
```

Without it, bad scripts can run in your browser when you visit the site.

1. **Use HTTPS with HSTS**

   - Stops hackers from making you use an older version of the site by making sure everything is over HTTPS.
   - Recommended setting:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

1. **Stop Clickjacking with X-Frame-Options**

   - Stops other sites from putting the website in a box to trick you.
   - Recommended setting:

```http
X-Frame-Options: DENY
```

1. **Stop MIME Sniffing with X-Content-Type-Options**

   - Makes sure the browser uses the right type of file and doesn't run bad scripts.
   - Recommended setting:

```http
X-Content-Type-Options: nosniff
```

1. **Stop XSS with Content-Security-Policy**

   - Limits where scripts, styles, and other things can come from.
   - Example policy:

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-random123'; object-src 'none'
```

1. **Use Referrer-Policy to Protect Privacy**

   - Controls how much information is sent when you go to another website.
   - Recommended setting:

```http
Referrer-Policy: strict-origin-when-cross-origin
```
