---
title: How to Find Hidden PPPoE Information from Your Modem
draft: false
date: 2025-06-01
tags:
  - pppoe
  - linux
---

It's 2025, and some ISPs still hide these PPPoE login details. I don't really understand their logic. They don't want customers to use other devices, but why? I don't have the answer to this question. If you know, I would appreciate it if you could comment.

Fortunately, there is a way to get this information. You just need to know a little Linux; it is not a very difficult process.

## Requirements

1.  Raspberry Pi/Linux Machine/Virtual Linux Machine, whichever you use.
2.  One meter of CAT6 cable
3.  Modem to get PPPoE information

Let's include the necessary package in our Debian system and start preparing the system.

## How to Set Up a PPPoE Server

This package allows you to connect to PPPoE service providers, but we will not connect; we will be a server.

```
apt install pppoe
```

Save the following in `/etc/ppp/options`. If you don't say `show-password`, it will not be logged properly, so take action accordingly. By default, this config also contains the `hide-password` line, don't be surprised and use it.

```
# /etc/ppp/options
#
# Originally created by Jim Knoble <jmknoble@mercury.interpath.net>
# Modified for Debian by alvar Bray <alvar@meiko.co.uk>
# Modified for PPP Server setup by Christoph Lameter <clameter@debian.org>
#
# To quickly see what options are active in this file, use this command:
#   egrep -v '#|^ *$' /etc/ppp/options

require-pap
login

# LCP settings
lcp-echo-interval 10
lcp-echo-failure 2

# Always Show Password
show-password

# Debug enable
debug

# PPPoE compliant settings
noaccomp
default-asyncmap
mtu 1492

# Log All
logfile /var/log/pppoe-server.log

# ---<End of File>---
```

Save and exit, let's learn our ethernet with the ifconfig command, mine is `ens3`, I noted it in the corner. Since I use Raspberry Pi, my job is a little easier at this point.

```bash
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 61.xxx.xxx.xxx  netmask 255.255.255.0  broadcast 89.xxx.xxx.xxx
        ether de:ad:be:ee:ee:ef  txqueuelen 1000  (Ethernet)
        RX packets 31935318  bytes 4272939405 (3.9 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2361679  bytes 1130613927 (1.0 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

Now we need to run the process on the machine and put it in the background. Normally, if we configured it like a modem, our job would be simple, but we are holding our ears differently.

```bash
pppoe-server -F -I ens3 -O /etc/ppp/options
```

It will go to the next line and wait like that, don't panic, it means it is working, we just need to put the process in the background. Press CTRL + C, stop the process, run it with `screen` or `tmux`, it's up to you.

## Time to Learn PPPoE

Now we have come to the most action-packed part. Take out the Ethernet cable, unplug the device given to you by the ISP, plug it into the Ethernet port of the Raspberry Pi from the WAN port, and make sure it is working. Don't even think about unplugging it.

Now plug in the modem and let it boot, let it request PPPoE from the port. Some devices may request late, so let the devices sit for a while.

Now let's check our log.

```
cat /var/log/pppoe-server.log
```

Yes, as I thought, our information has arrived. If you are doing it on Superonline, you will see it as @fiber, if you are doing it on TTnet, you will see it as @ttnet, and if you are doing it on Turksat, you will see it as @tsat.

```bash
using channel 1
Using interface ppp0
Connect: ppp0
sent [LCP ConfReq id=0x1 <mru 1492> <auth pap> <magic 0x0000c0c0>]
rcvd [LCP ConfAck id=0x1 <mru 1492> <auth pap> <magic 0x0000c0c0>]
rcvd [LCP ConfReq id=0x4 <mru 1492> <magic 0xa1a58299>]
sent [LCP ConfAck id=0x4 <mru 1492> <magic 0xa1a58299>]
sent [LCP EchoReq id=0x0 magic=0x61bc6dcc]
rcvd [LCP EchoReq id=0x0 magic=0xa1a58399]
sent [LCP EchoRep id=0x0 magic=0x61bc6dcc]
rcvd [PAP AuthReq id=0x1 user="101010101010@tsat" password="1234567"]
no PAP secret found for 101010101010@tsat
sent [PAP AuthNak id=0x1 "Login incorrect"]
PAP peer authentication failed for 101010101010@tsat
sent [LCP TermReq id=0x2 "Authentication failed"]
rcvd [LCP EchoRep id=0x0 magic=0xa1a58399]
rcvd [LCP TermReq id=0x5 "Failed to authenticate ourselves to peer"]
sent [LCP TermAck id=0x5]
rcvd [LCP TermAck id=0x2]
Connection terminated.
```

Yes, our information has arrived, now you can go and use it in any router you want. If you think the ISP will be upset about this situation, you can also get the MAC address of the device given to you and give it to the router you use. If your ISP does not support it, do not forget to disable 802.1q.

If you cannot see the log file or it has not been created, do the following and repeat all the above steps.

```bash
touch /var/log/pppoe-server.log
chmod 0774 /var/log/pppoe-server.log
```

You can do all the above operations with a virtual machine, in that case, do not neglect to port mirror.

[Reference](https://mertcangokgoz.com/gizli-olan-pppoe-modemden-nasil-ogrenilir/)
