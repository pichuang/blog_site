---
layout: post
title: Linux iptables
date: 2017-02-28 05:42:31
updated: 2017-02-28 05:42:31
category:
- Infra
- Linux
tags:
- linux
- misc
- iptables
---

# Packet Flow in Netfilter

{% img http://inai.de/images/nf-packet-flow.png %}

<!--more-->

{% img http://xkr47.outerspace.dyndns.org/netfilter/packet_flow/packet_flow10.png %}

# Concept
- 4 Tables after kernel version 2.6.x
  - Raw
  - Mangle
  - Nat
  - Filter (Default Table in iptables)

# Scenario
## SNAT
- `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`
  - [Quick-Tip: Linux NAT in Four Steps using iptables](http://www.revsys.com/writings/quicktips/nat.html) 

## Port Forwarding
- `iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 3389 -j DNAT --to-destination <TARGET_IP>:3389`
  - [Destination NAT](https://www.netfilter.org/documentation/HOWTO/NAT-HOWTO-6.html#ss6.2) 

## Drop ICMP Packet
- `iptables -t filter -A FORWARD -p ICMP -j DROP`
