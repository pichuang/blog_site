---
layout: post
title: Data Center Bridging (DCB) Research
date: 2017-06-20 12:04:02
updated: 2017-06-20 12:04:02
category: Infra
tags:
- dcb
- pfc
- ets
---

## Overview
Data Center Bridging is a collection of standards-based extensions to classical Ethernet. 

It provides a **lossless data center transport layer** that enables the convergence of LANs and SANs onto a single unified fabric.

In addition to supporting Fibre Channel Over Ethernet (FCoE) and iSCSI Over DCB

It includes the following capabilities:
* Priority-based flow control (PFC; IEEE 802.1Qbb)
* Enhanced Transmission Selection (ETS; IEEE 802.1Qaz)
* Congestion Notification (CN; IEEE 802.1Qau)
* Data Center Bridging eXchange (DCBx)

## Environment
- Ubuntu Server LTS 14.04.5
  - lldpd-dev
- XL710
  - p11p{1-4}



## References
- [Data Center Bridging (DCB) for Intel® Network Connections][1]
- [Dell Networking – RDMA over Converged Ethernet Technical Brief][2]
- [DCB学习之一 (PFC)][3]

[1]: ftp://ftp.supermicro.com/CDR-B9_1.00_for_Intel_B9_platform/Intel/LAN/v16.8.1/APPS/FCOEBOOT/DOCS/dcb.htm
[2]: http://pleiades.ucsc.edu/doc/dell/network/Dell_Networking_RoCE_Configuration.pdf
[3]: http://blog.csdn.net/goodluckwhh/article/details/11539111