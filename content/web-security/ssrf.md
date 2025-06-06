---
title: Server-Side Request Forgery (SSRF)
draft: false
date: 2025-01-01
tags:
  - owasp
  - ssrf
  - server_side_request_forgery
  - injection
---

Server-Side Request Forgery (SSRF) happens when someone tricks a server into making requests it shouldn't. This can let them steal information, scan the internal network, find cloud information, and mess with services. SSRF is very dangerous when apps let users control URLs or don't stop requests from going out.

Common Problems:

- Getting URLs from the outside without checking them (like allowing any URL in request settings)
- Getting into internal services (like databases, admin areas, cloud info areas)
- Stealing AWS credentials through SSRF using the Instance Metadata Service (IMDS)
- Getting around network rules to mess with internal systems
- Working with Cloud Services (like Kubernetes, Docker APIs) to get unauthorized access
- Making the app do bad things to other services

To make things safer, apps should check user-given URLs, only allow certain outgoing requests, block access to internal IP addresses (like 127.0.0.1, 169.254.169.254), and use metadata service version 2 (IMDSv2) in AWS. Also, keeping track of outgoing requests can help find and stop SSRF attempts.
