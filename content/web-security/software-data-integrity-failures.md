---
title: Software and Data Integrity Failures
draft: false
date: 2025-01-01
tags:
  - owasp
  - software_integrity
  - data_integrity
  - supply_chain_attacks
  - ci_cd_security
  - digital_signatures
---

Software and Data Integrity Failures happen when applications don't check if software updates, important data, or things they rely on are correct.

Common Problems:

- Not having digital signatures or checking if software updates are correct using codes.
- Using libraries, plugins, or packages from places that are not trusted or have been hacked.
- Messing with configuration files, logs, or important system data.
- Not securing the systems that build and release software (CI/CD pipelines).
- Putting in bad things that the application relies on (attacks on where things come from).
- Not making sure data in databases or caches is correct.

To make these risks smaller, applications should use special signatures to check if software is correct. They should only use things from trusted places. They should secure the systems that build and release software. They should protect important data from being changed without permission by using codes, access controls, and ways to find if something has been changed. Checking regularly and keeping an eye on what the application depends on can also help reduce the risk of software and data problems.
