---
title: Insecure Direct Object References (IDOR)
draft: false
date: 2025-01-01
tags:
  - owasp
  - idor
  - access_control
  - authorization
  - broken_access_control
---

Insecure Direct Object Reference (IDOR) is a type of problem that happens when a program uses information given by a user to get to things inside it, like files or records in a computer's memory. This can be a problem if the program doesn't check carefully to make sure the user is allowed to see or change those things. If it doesn't, someone could trick the program into showing them things they shouldn't see, or letting them change things they shouldn't change.

IDOR often happens because the program doesn't have good rules about who is allowed to see or change things. The program might think that if someone is logged in, they should be able to see or change anything. But that's not safe, because someone could change the information they give the program to see or change things that belong to other people.

For example, imagine a website where you can see your profile by going to a link like this:

`https://example.com/user/profile?id=12345`

This link shows the information for the user with the ID number 12345. If the website doesn't check to make sure you're allowed to see that user's information, someone could change the link to:

`https://example.com/user/profile?id=67890`

And see the information for the user with the ID number 67890.

Or, imagine a website that stores files and lets you see them with links like this:

`https://example.com/documents?file=invoice_12345.pdf`

If the website doesn't check to make sure you're allowed to see that file, someone could change the link to:

`https://example.com/documents?file=invoice_67890.pdf`

And see someone else's file. This could let them see private information.

Sometimes, people can even use IDOR to give themselves special permissions on a website. For example, they could change a number in a request to make themselves an administrator, if the website doesn't check to make sure they're allowed to do that.

Here are some ways to prevent IDOR:

1. **Check Who Is Allowed to See or Change Things**
   - Always make sure the person using the program is allowed to see or change the thing they're trying to see or change.
   - Do this checking on the computer that runs the website, not just on the user's computer.
2. **Use Secret Codes Instead of Direct References**
   - Instead of using simple numbers to identify things, use secret codes that are hard to guess.
   - This makes it harder for people to find things they're not supposed to see.
3. **Check the Information Users Give**
   - If you have to use simple numbers to identify things, check to make sure the person using the program is allowed to see or change the thing they're asking for.
   - Don't trust hidden information or information that comes from the user's computer, because they can change it.
4. **Use Safe Programming Practices**
   - Use tools and programs that have built-in ways to control who is allowed to see or change things.
   - Only give people the minimum permissions they need to do their jobs.

*Reference: OWASP Top 10 Security Risks*
