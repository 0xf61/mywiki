---
title: HTTP Strict Transport Security (HSTS)
draft: false
date: 2025-01-01
tags:
  - owasp
  - hsts
  - https
  - tls
  - ssl
  - cryptographic_failures
---

HTTP Strict Transport Security (HSTS) is a security feature that helps protect websites from attacks that try to weaken the connection or steal cookies. When a website uses HSTS, it tells browsers to only connect to it using a secure HTTPS connection for a certain amount of time. This means that even if someone tries to visit the site using a regular HTTP connection, the browser will automatically switch to HTTPS. This helps prevent people from accidentally using an insecure connection.

HSTS makes the internet safer by encouraging the use of secure connections. It also helps protect against attacks where someone might try to trick the user into using an insecure HTTP connection without them knowing.

Here's a simple example of how the HSTS feature is used:

```plaintext hljs

Strict-Transport-Security: max-age=31536000

```

In this example, the number 31536000 means one year. This tells the browser to remember to only use HTTPS for this website for the next 365 days. If someone tries to visit the site using HTTP, the browser will automatically change the connection to HTTPS.

Some websites also use these extra settings:

```plaintext hljs

Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

```

- `includeSubDomains` means that the HSTS protection also applies to all subdomains of the website, making sure they also use secure connections.
- `preload` is used by browsers that have a list of websites that always use HTTPS. If a website is on this list, the browser will always use HTTPS, even if it's the first time visiting the site. This makes sure there's no chance of using an insecure connection.

Here's how to use HSTS to protect your website:

1. **Always Use HTTPS**
   - Make sure your website has a valid security certificate for HTTPS.
   - Redirect all regular HTTP requests to the secure HTTPS version of your website before you start using HSTS.
2. **Set Up the HSTS Header**
   - Choose a good `max-age` value (usually at least 31536000 seconds, which is one year).
   - Think about using `includeSubDomains` to protect all subdomains.
   - Only use `preload` if you're sure all your subdomains use HTTPS and you want to add your website to the browser's list of preloaded HSTS sites.
3. **Start Slowly**
   - If you're not sure if all your subdomains are ready for HTTPS, start with a smaller `max-age` and don't use `includeSubDomains`.
   - Gradually increase the `max-age` and then add `includeSubDomains` once you're sure everything is secure.

*Reference: OWASP Top 10 Security Risks*
