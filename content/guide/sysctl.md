---
title: Linux Kernel Hardening
draft: false
date: 2024-05-12
tags:
  - sysctl
  - configuration
  - security
---

Here is the list of sysctl settings I use to harden the Linux kernel.
You can add them to `/etc/sysctl.conf` or create a new file in `/etc/sysctl.d/` with a `.conf` extension.

After writing the settings, run `sysctl -p` to apply them.

```
# Prevent ptrace from changing the running process
kernel.yama.ptrace_scope=3

# Disable kexec
kernel.kexec_load_disabled=1

# Disable sysrq
kernel.sysrq=0

# Restrict kernel pointer
kernel.kptr_restrict=2

# Remove sensitive data in kernel messages
kernel.dmesg_restrict=1

# Hide dmesg during boot
kernel.printk=3 3 3 3

# Restrict privilege escalation with TTY
dev.tty.ldisc_autoload=0

# Minimize privilege escalation potential
kernel.unprivileged_userns_clone=0

# Restrict performance events
kernel.perf_event_paranoid=3

# Disable eBPF feature
kernel.unprivileged_bpf_disabled=1

# eBPF jit hardening
net.core.bpf_jit_harden=2

# SYNFlood protection (0)
net.ipv4.tcp_syncookies=1

# RST Packet drop
net.ipv4.tcp_rfc1337=1

# Increase entropy to spread critical memory regions randomly
vm.mmap_rnd_bits=32
vm.mmap_rnd_compat_bits=16

# Verify source in received packets
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1

# Disable ICMP redirects
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.conf.all.secure_redirects=0
net.ipv4.conf.default.secure_redirects=0
#net.ipv6.conf.all.accept_redirects=0
#net.ipv6.conf.default.accept_redirects=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.default.send_redirects=0

# Prevent Smurf attack, restrict ICMP
net.ipv4.icmp_echo_ignore_all=1
#net.ipv6.icmp.echo_ignore_all=1

# Restrict data spoofing
fs.protected_fifos=2
fs.protected_regular=2

# Disable redirects
net.ipv4.conf.all.accept_source_route=0
net.ipv4.conf.default.accept_source_route=0
#net.ipv6.conf.all.accept_source_route=0
#net.ipv6.conf.default.accept_source_route=0

# Disable TCP SACK
net.ipv4.tcp_sack=0
net.ipv4.tcp_dsack=0
net.ipv4.tcp_fack=0

# Restrict IPv6 to prevent MitM attacks
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
net.ipv6.conf.default.router_solicitations=0
net.ipv6.conf.default.accept_ra_rtr_pref=0
net.ipv6.conf.default.accept_ra_pinfo=0
net.ipv6.conf.default.accept_ra_defrtr=0
net.ipv6.conf.all.accept_ra=0
net.ipv6.conf.default.accept_ra=0
net.ipv6.conf.default.autoconf=0
net.ipv6.conf.default.dad_transmits=0
net.ipv6.conf.default.max_addresses=1

# IPv6 Addr generated from Mac address. Right?
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
net.ipv6.conf.eth0.use_tempaddr = 2
net.ipv6.conf.wlan0.use_tempaddr = 2

# Restrict symbolic links (only those with read/write permissions can access)
fs.protected_symlinks=1
fs.protected_hardlinks=1

# Prevent leaking system time
net.ipv4.tcp_timestamps=0

# Increase network receive queue
net.core.netdev_max_backlog = 16384

# Increase incoming connection (4096)
net.core.somaxconn = 8192

# Increase memory for network
net.core.rmem_default = 1048576
net.core.rmem_max = 16777216
net.core.wmem_default = 1048576
net.core.wmem_max = 16777216
net.core.optmem_max = 65536
net.ipv4.tcp_rmem = 4096 1048576 2097152
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.udp_rmem_min = 8192
net.ipv4.udp_wmem_min = 8192

# Allow TCP fast open
net.ipv4.tcp_fastopen = 3

# Max queue length
net.ipv4.tcp_max_syn_backlog = 8192

# Max timewait prevent DOS
net.ipv4.tcp_max_tw_buckets = 2000000

# Delete some socket if filled out
net.ipv4.tcp_tw_reuse = 1

# Set timeout for waiting FIN packets
net.ipv4.tcp_fin_timeout = 10

# Remember the old window size not start from 0 to max
net.ipv4.tcp_slow_start_after_idle = 0

# Discover best MTU
net.ipv4.tcp_mtu_probing = 1

# Use swap if %17 ram remains
vm.swappiness=17
```
