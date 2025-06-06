---
title: Authorization Bypass
draft: false
date: 2025-01-01
tags:
  - owasp
  - bypass
  - access_control
---

Authorization Bypass is a security problem where a program doesn't check permissions properly. This lets bad people get to things or do things they shouldn't be able to. It usually happens because the rules for who can do what are weak or not complete. Even if someone doesn't have the right permissions, they can trick the system (by guessing links, changing settings, or messing with how the system remembers them) to get into places or do things they're not allowed to. Sometimes, programmers think it's enough to check things on the user's side of the computer, but they don't protect the important parts on the server.

Authorization Bypass can cause big problems, like letting people see private information, become more powerful in the system than they should be, change important records, or even take control of the whole program.

For example, a program might have special pages just for administrators, like this:

`https://example.com/admin/dashboard`

If the server doesn't check if the person asking to see the `/admin/dashboard` page is really an administrator, anyone (even someone who's not logged in) could just type the address into their browser and see it.

Or, imagine a request includes a setting that says what kind of user someone is:

```http
 POST /updateUser
 Role: user
```

If the program lets someone change that setting to:

```http
 POST /updateUser
 Role: admin
```

without checking if they really have permission to be an administrator, they could become an administrator and do bad things.

Some processes (like buying things online or signing up for an account) have steps that are supposed to be done in order. But if the program only checks these steps on the user's side, a bad person could skip steps by changing the address or settings, and get to the end without doing everything they're supposed to.

Here's how to prevent Authorization Bypass:

- Always check on the server side who is allowed to see each thing, do each function, or go to each address.
- Have clear rules about who can do what, and check these rules every time someone asks to do something, not just when they log in or on the user's side of the computer.
- Don't just rely on hidden information, cookies, or things on the user's side of the computer to decide who has what permissions.
- Check that any information a user sends matches what you expect, and make sure they have permission to do what they're asking to do.
- Don't let people get to addresses directly; instead, connect addresses to roles that are allowed to use them.
- Use a central system for checking permissions (like middleware or filters) so that the rules are the same everywhere and can't be skipped in certain parts of the program.
- Make sure that the codes that remember users are connected to their permissions every time they ask to do something.
- Protect these codes from being stolen or used again by using secure cookies, HTTP-only flags, and encryption if needed.
