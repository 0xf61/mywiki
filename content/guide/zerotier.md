---
title: Zerotier endpoint access with bridge on Raspberry Pi
draft: false
date: 2025-05-11
tags:
  - zerotier
  - zerotrust
  - network
---

[Zerotier](https://www.zerotier.com/) is a service that can be used to create a zero-trust network. It acts as a relay service between devices on different networks without a central server. This lets you establish a local connection to devices included in your virtual network using a bridge that has internet access or is on the same local network with internet.

The setup is very simple. After logging in, you get a network ID. Then, you install the Zerotier application for your device's platform. [Link](https://www.zerotier.com/download/).

## Example Scenario

You can use a system like the one below to remotely connect to an embedded Raspberry Pi.

```bash
curl -s https://install.zerotier.com | sudo bash
```

After installation, enable the IP Forwarding kernel parameter on the Raspberry Pi with this command:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

You must join the 16-character network ID to the agents.

```bash
zerotier-cli join $ZEROTIER_NETWORK_ID
```

After the device is authorized on the site, you can access the Raspberry Pi using its IP address. To allow the Raspberry Pi to access devices on the network it's connected to, you need to forward the packets using iptables rules.

Based on the output of the `ip a` command, define the network names you want to bridge as follows:

```bash
PHY_IFACE=eth0; ZT_IFACE=zt7nnig26
```

Use the commands below to bridge the two networks.

```bash
sudo iptables -t nat -A POSTROUTING -o $PHY_IFACE -j MASQUERADE
sudo iptables -A FORWARD -i $PHY_IFACE -o $ZT_IFACE -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i $ZT_IFACE -o $PHY_IFACE -j ACCEPT
```

After this, run the following commands in order to make the connection permanent after a possible power outage.

```bash
sudo apt install iptables-persistent
sudo bash -c iptables-save > /etc/iptables/rules.v4
```
