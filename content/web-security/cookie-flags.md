---
title: Cookie Flags - Security Configuration
draft: false
date: 2025-01-01
tags:
  - owasp
  - cookies
  - httponly
  - secure
  - samesite
  - security_misconfiguration
---

Cookie Flags are like labels on cookies that tell your computer how to use them safely. If these labels aren't set right, bad people might be able to trick your computer or steal your information.

Cookies are often used to remember who you are when you log in to a website or to remember your preferences. Making sure these labels are correct helps keep your information safe.

If the `HttpOnly` label isn't set, it's like leaving the cookie jar open. Someone could use a trick to grab the cookie using a special code:

```javascript
<script>
  alert(document.cookie);
</script>
```

If someone steals your cookie, they could pretend to be you on the website.

If a cookie doesn't have the `Secure` label, it's like sending it through the mail without an envelope. Anyone watching could see the cookie.

Here's an example of a cookie without the `Secure` label:

```html
Set-Cookie: sessionid=abcd1234; Path=/; HttpOnly;
```

Without `Secure`, the cookie can be seen by others if you're not using a safe connection.

The `SameSite` label helps prevent tricks where someone makes you do something without you knowing. If this label is missing or set wrong, it's like letting anyone use your cookie, even if they shouldn't.

Here's an example of a cookie missing the `SameSite` label:

```html
Set-Cookie: sessionid=abcd1234; Path=/; Secure; HttpOnly;
```

If the `SameSite` label is missing, someone could trick your computer into using the cookie when it shouldn't.

Here's how to set the labels correctly:

1. **Set HttpOnly to Prevent Stealing**

- This makes sure that only the website can use the cookie, not sneaky code.
- Example:

```http
Set-Cookie: sessionid=abcd1234; Path=/; HttpOnly;
```

2. **Use Secure to Keep Cookies Safe**

- This makes sure the cookie is only sent through a safe, locked connection.
- Example:

```http
Set-Cookie: sessionid=abcd1234; Path=/; Secure; HttpOnly;
```

3. **Enforce SameSite for Protection**

- Use `SameSite=Lax` or `SameSite=Strict` to prevent others from using your cookie on different websites without your permission.
- Example:

```http
Set-Cookie: sessionid=abcd1234; Path=/; Secure; HttpOnly; SameSite=Lax;
```

4. **Set Rules for Where Cookies Can Be Used**

- Limit cookies to certain parts of a website to make sure they can't be used where they shouldn't be.
- Example:

```http
Set-Cookie: sessionid=abcd1234; Path=/account; Secure; HttpOnly; SameSite=Strict;
```
