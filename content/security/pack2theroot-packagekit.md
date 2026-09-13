---
title: "Pack2TheRoot (CVE-2026-41651): PackageKit TOCTOU Local Privilege Escalation"
draft: false
date: 2026-09-13
tags:
  - cve
  - privesc
  - linux
---

A high-severity (CVSS 3.1: 8.8) local privilege escalation vulnerability in PackageKit, nicknamed **Pack2TheRoot**. It was discovered by Deutsche Telekom's Red Team and disclosed on 2026-04-22. Any local unprivileged user can exploit it to gain root on a default installation of many major Linux distributions.

## What is PackageKit?

PackageKit is a D-Bus system service that runs as **root** and provides a cross-distro, cross-architecture API for package management. GUI software centers like GNOME Software use it, and desktop tools like `pkcon` talk to it directly. Authorization (e.g. "does this user may install packages?") is delegated to polkit.

## The Bug: TOCTOU Race on Transaction Flags

When a client wants to install a package, it creates a transaction object over D-Bus and calls a method like `InstallFiles(flags, [path])`.

The `flags` bitfield controls the transaction's behavior. Certain flag values like `SIMULATE` and `ONLY_DOWNLOAD` cause PackageKit to **skip polkit authorization entirely** — because those operations are considered safe and should never modify the system.

The vulnerability is a time-of-check time-of-use (TOCTOU) race condition on `transaction->cached_transaction_flags`. Three bugs in `src/pk-transaction.c` chain together:

1. **Unconditional flag overwrite**: `InstallFiles()` writes caller-supplied flags to the cached transaction flags without checking whether the transaction has already been authorized or started. A second call blindly overwrites the flags even while the transaction is RUNNING.
2. **Silent state-transition rejection**: the state machine guard discards illegal backward transitions (e.g. `RUNNING` → `WAITING_FOR_AUTH`), but the flag overwrite already happened — the transaction keeps running with corrupted flags.
3. **Late flag read**: the scheduler reads the cached flags at dispatch time, not at authorization time.

### The exploit, step by step

The public PoC (on GitHub since 2026-04-23) boils down to **two back-to-back D-Bus calls on the same transaction object**:

```python
# 1. Get a transaction object from the PackageKit daemon
 tid = dbus_call("org.freedesktop.PackageKit", "/org/freedesktop/PackageKit",
                "org.freedesktop.PackageKit", "CreateTransaction")

# 2. Call InstallFiles with a "safe" flag (SIMULATE / ONLY_DOWNLOAD)
#    → no polkit auth required, transaction is accepted as harmless
 #    dummy = benign package
 dbus_call(PK_BUS, tid, TX_IFACE, "InstallFiles", (FLAG_SIMULATE, [dummy]))

# 3. Immediately call InstallFiles AGAIN on the same transaction,
#    this time with flags stripped and the payload package path
 #    → unconditionally overwrites cached_transaction_flags
 dbus_call(PK_BUS, tid, TX_IFACE, "InstallFiles", (FLAG_NONE, [payload]))
```

No sleep, no retry loop — both messages are sent before the daemon replies. That's the whole trick:

- GLib dispatches D-Bus messages at **higher priority than idle callbacks**, and the scheduler runs transactions via idle callbacks. So call 2 is *guaranteed* to be processed before the transaction executes.
- Call 2 overwrites the cached flags (bug 1) and tries to move the state machine backwards to re-authenticate (bug 2). The backward transition is silently discarded — but the flag overwrite already happened.
- When the scheduler's idle callback finally dispatches the transaction, it reads the flags **at dispatch time** (bug 3) — now `FLAG_NONE` — and the backend performs a real install as root. The polkit check that should have gated a real install never runs.

### The payload

The "payload" is just a package the attacker builds locally, containing a maintainer script that drops a SUID root shell:

```bash
# RPM %post scriptlet / deb postinst — executed as root during install
install -m 4755 /bin/bash /var/tmp/.suid_bash
```

After the install, `/var/tmp/.suid_bash -p` is a root shell. One detail the PoC handles: the drop directory must be on a filesystem mounted **without `nosuid`/`noexec`** (it probes `/proc/mounts` for `/var/tmp`, `/dev/shm`, `/tmp`, `$HOME` in order). A side effect of the race leaves the daemon in a corrupted state, so `packagekitd` crashes with an assertion failure after the install — which is also the IOC below.

## Who is affected

- All PackageKit versions **>= 1.0.2 and <= 1.3.4** (1.0.2 is over 12 years old)
- Explicitly confirmed on default installs: Ubuntu Desktop 18.04/24.04.4/26.04, Ubuntu Server 22.04–24.04, Debian Desktop Trixie 13.4, RockyLinux Desktop 10.1, Fedora 43 Desktop & Server (apt and dnf backends)
- Assume any distro shipping an enabled PackageKit is vulnerable. Since PackageKit is an optional dependency of **Cockpit**, many servers (including RHEL) can be affected too.

## Check, fix, mitigate

Check if PackageKit is installed and active (don't just grep `ps` — the daemon is D-Bus activated on demand):

```bash
dpkg -l | grep -i packagekit   # Debian/Ubuntu
rpm -qa | grep -i packagekit   # RPM-based
systemctl status packagekit    # loaded/running = daemon available
```

- **Fix**: PackageKit 1.3.5 or your distro's backport (Ubuntu/Debian/Fedora shipped patched packages on disclosure day).
- **Workaround** (if no patch available): a polkit rule that immediately denies PackageKit install actions for non-root users — `/etc/polkit-1/rules.d/49-workaround-cve-2026-41651.rules` (see the Telekom advisory below for the exact rule). Side effect: GUI software centers can no longer install packages; `sudo dnf/yum install` is unaffected.

## Detection / IOC

Exploitation crashes the PackageKit daemon with an assertion failure (systemd restarts it), which is visible in logs:

```bash
journalctl --no-pager -u packagekit | grep -i emitted_finished
```

```text
packagekitd[2082]: PackageKit:ERROR:../src/pk-transaction.c:514:pk_transaction_finished_emit:
assertion failed: (!transaction->priv->emitted_finished)
```

## References

- [Telekom Security: Pack2TheRoot advisory](https://github.security.telekom.com/2026/04/pack2theroot-linux-local-privilege-escalation.html)
- [GHSA-f55j-vvr9-69xv](https://github.com/PackageKit/PackageKit/security/advisories/GHSA-f55j-vvr9-69xv)
- [NVD entry](https://nvd.nist.gov/vuln/detail/CVE-2026-41651)
- Public PoC: [baph00met/CVE-2026-41651](https://github.com/baph00met/CVE-2026-41651), [shibaaa204/Pack2TheRoot](https://github.com/shibaaa204/Pack2TheRoot)
