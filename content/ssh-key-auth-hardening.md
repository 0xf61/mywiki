---
title: "Secure SSH with Key-Based Authentication"
draft: false
date: 2026-04-24
tags:
  - ssh
  - linux
  - system-administration
---

To secure your SSH server, disable password authentication and rely exclusively on SSH keys. This prevents brute-force attacks against user passwords.

1. Generate an SSH key pair on your client machine (if you don't have one):

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

2. Copy the public key to your server:

   ```bash
   ssh-copy-id username@server_ip
   ```

3. Log into your server and edit the SSH daemon configuration file:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

4. Modify the following directives to disable password authentication and enable public key authentication:

   ```text
   PasswordAuthentication no
   PubkeyAuthentication yes
   PermitRootLogin prohibit-password
   ```

5. Restart the SSH service to apply the changes:
   ```bash
   sudo systemctl restart sshd
   ```
