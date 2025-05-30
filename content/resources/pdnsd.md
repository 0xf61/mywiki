---
title: Pdnsd
draft: false
date: 2024-01-23
tags:
  - archlinux
  - dns
  - caching
---

[pdnsd](http://members.home.nl/p.a.rombouts/pdnsd/index.html)
is a DNS server that saves DNS information locally
This can make browsing the internet faster if set up correctly
Compared to [BIND](https://wiki.archlinux.org/title/BIND "BIND") or
[dnsmasq](https://wiki.archlinux.org/title/Dnsmasq "Dnsmasq"),
pdnsd can remember its saved information after the computer restarts
The "p" in "pdnsd" stands for "persistent," meaning it lasts a long time
See [Domain name resolution#DNS servers](https://wiki.archlinux.org/title/Domain_name_resolution#DNS_servers "Domain name resolution") to compare it with other DNS servers.

## Configuration

The package has a sample configuration file at `/usr/share/doc/pdnsd/pdnsd.conf`. You should create your own configuration file at `/etc/pdnsd.conf`.

## Tips and tricks

### Performance settings for home broadband users

If your DNS server is slow, using `pdnsd` as a caching server can help. After doing the setup above, these settings in `/etc/pdnsd.conf` will help:

Under global settings:

```
neg_rrs_pol=on;
par_queries=1;
```

Under server settings:

```
proxy_only=on;
purge_cache=off;
```

`neg_rrs_pol=on;` means that pdnsd will save negative responses, even if they are not "authoritative." This is important because many requests for AAAA records (IPv6) and MX records will not return results. Without negative caching, these requests will be sent even after a domain name has been cached.

`par_queries=1;` is useful if you have more than one DNS server. It specifies how many parallel queries will be made at once. If you have four DNS servers, and `par_queries=2;`, the first two servers will be queried at the same time. If they both fail, pdnsd will move on to the next two. The setting above means that one DNS server at a time gets queried.

`proxy_only=on;` prevents `pdnsd` from resolving all the way back to the "authoritative" name server. Instead, it accepts the results of the DNS servers that were specified in the "server" section.

`purge_cache=off;` tells `pdnsd` not to remove cache entries even if they have outlived the DNS record's time-to-live. This is useful when your internet provider's DNS server goes down.

### Additional performance settings

#### TTLs (Time-To-Live)

Each DNS record includes a maximum time-to-live, or TTL. This tells the recipient how long to store the record. Many DNS records have short TTLs, such as 3600 seconds (one hour). To improve performance, you can set a global minimum TTL, causing fewer lookups to be performed. The disadvantage is that a cached record may be out of date. However, most IP addresses do not change often.

Times are in seconds by default. You can add "m", "h", "d", or "w" to specify minutes, hours, days, or weeks.

`min_ttl` in the global settings sets a minimum TTL for cached records.

`neg_ttl` in the global settings sets a minimum TTL for non-existent domains.

#### Timeouts

Shorter timeouts mean that pdnsd will give up on a query more quickly, resulting in faster performance. The disadvantage is that pdnsd might return an error simply because the server was not given enough time to respond.

`timeout` in the global settings determines when pdnsd gives up on a query.

`tcp_qtimeout` in the global settings determines how long a TCP query connection may be left open.

`timeout` in the server settings determines how long pdnsd will wait for a response from each server.

#### Debugging

To see what servers pdnsd is using, how timeouts are working, and what default TTLs are being used, turn debug on in the global settings:

```
debug=on;
```

Restart pdnsd and watch the `pdnsd.service` for changes with the [systemd journal](https://wiki.archlinux.org/title/Systemd_journal "Systemd journal"):

```
# journalctl -f -u pdnsd.service
```

Turn debug off when you are done because it can slow down performance.

#### Cache size

By default, pdnsd will create records for all entries in `/etc/hosts`. If you have a lot of entries, the default maximum cache size may not be large enough.

To increase the cache size, edit the `perm_cache` line in the 'global settings' section of the configuration file (size in kB).

Alternatively, you can prevent pdnsd from reading your hosts file by adding the option `authrec=off` to the 'source' section. If that does not work, create a separate hosts file (e.g., `/etc/hosts-pdnsd`) with only your system information and point your 'source' section to that instead.

```
/etc/hosts-pdnsd
```

```
#<ip-address> <hostname.domain.org> <hostname>
127.0.0.1 localhost.localdomain my_hostname
::1 localhost.localdomain localhost
```

### Shared server for your LAN

If you have several computers on your network, you can make pdnsd the DNS server for them all. This allows your entire network to share a single DNS cache. To do this, set `server_ip` in the `global` section to the name of your network interface (usually `eth0`). If you have a firewall, allow connections to port 53 from any address on your network.

Then, configure the other computers on your network to use the computer running pdns as their primary DNS server.

### Name blocking

pdnsd allows you to block hosts or domains. Create a new `neg` section in `pdnsd.conf`. `neg` sections have two main options. `name` is the name of the host or domain you want to block. `types` can be set to `domain` to block all hosts in the given domain. The default `pdnsd.conf` has an example that blocks ads from doubleclick.net.

Since you can only set one domain per block, it is better to create separate configuration files for lists of blocked domains, e.g., `/etc/pdnsd.d/spam_domains` and add an `include` section in `pdnsd.conf` like this:

```
include {file="/etc/pdnsd.d/spam_domains";}
```

Alternatively, you can add a `source` section that loads a file in `/etc/hosts` format that connects domains to the IP address 0.0.0.0. This will not block subdomains because `/etc/hosts` does not allow wildcards.

### pdnsd-ctl

From [pdnsd-ctl(8) § DESCRIPTION](https://man.archlinux.org/man/pdnsd-ctl.8#DESCRIPTION):

**pdnsd-ctl** controls **pdnsd**, a DNS server that saves information. The status control socket must be enabled before you can use **pdnsd-ctl**.

To do that, include the option

```
status_ctl = on;
```

in the global section of the `/etc/pdnsd.conf`.

If you changed the cache directory in `/etc/pdnsd.conf`, run `pdnsd-ctl` with the `-c` option:

```
# pdnsd-ctl -c path/to/cache
```

Useful commands:

View cache:

```
# pdnsd-ctl dump
```

Clear cache:

```
# pdnsd-ctl empty-cache
```

## Troubleshooting

### result of uptest for 192.168.x.x: failed

If you can ping your internet provider's DNS server, but the log shows:

```
# journalctl -f -u pdnsd.service
```

```
result of uptest for 192.168.x.x: failed
```

Check the interface configured in `/etc/pdnsd.conf`:

```
interface = any;
```

or the one in the server section:

```
interface=enp2s0;
```

Find the correct name by running: `ifconfig`.

## FAQs

Q) It does not seem much faster to me. Why?**A)** The extra speed comes from how long it takes to connect to a server. With slower connections, there will not be as large a difference.

Q) Why is it so much slower now than before?**A)** You probably have the `proxy_only` option turned off in one of the server sections of `pdnsd.conf`. Turn it on if you use the DNS server provided by your internet provider.

Retrieved from " [https://wiki.archlinux.org/index.php?title=Pdnsd&oldid=830222](https://wiki.archlinux.org/index.php?title=Pdnsd&oldid=830222)"

- [Domain Name System](https://wiki.archlinux.org/title/Category:Domain_Name_System "Category:Domain Name System")
