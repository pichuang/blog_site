---
layout: post
title: "Upgrade Intel Drivers on VMWare ESXi"
date: 2017-01-24 21:51:47 +0800
updated: 2017-01-24 00:00:00 +0800
description: ""
tags:
  - infra
  - vmware
  - intel
  - driver
categories:
  - infra
---

## Objective
Upgrade Intel NIC firmware (5.02->5.05) on VMWare ESXi for make sure the DirectPath I/O can work well

## Environment
- ESXi-6.0.0-20160302001
- Intel XL710
  - Including vmnic{2-5}

<!--more-->

## Step

### Check Current Version
```bash
[root@esxi:~] ethtool -i vmnic2
driver: i40e
version: 1.3.38
firmware-version: 5.02 0x80002248 1.33.0
bus-info: 0000:01:00.0
```
- Please enable SSH daemon on the server

### Download NVM Update file and Upgrade NIC
```bash
[root@esxi:~] cd /tmp
[root@esxi:/tmp] wget http://downloadmirror.intel.com/25796/eng/XL710_NVMUpdatePackage_v5_05_ESX.tar.gz
[root@esxi:/tmp] tar zxvf XL710_NVMUpdatePackage_v5_05_ESX.tar.gz
[root@esxi:/tmp] cd /tmp/XL710/ESXi_x64
[root@esxi:/tmp/XL710/ESXi_x64] chmod +x nvmupdate64e
[root@esxi:/tmp/XL710/ESXi_x64] ./nvmupdate64e

Intel(R) Ethernet NVM Update Tool
NVMUpdate version 1.26.17.11
Copyright (C) 2013 - 2016 Intel Corporation.

WARNING: To avoid damage to your device, do not stop the update or reboot or power off the system during this update.
Inventory in progress. Please wait [**********]

Num Description                            Device-Id B:D   Adapter Status
=== ====================================== ========= ===== ====================
01) Intel(R) I210 Gigabit Network Connecti 8086-1533 09:00 Update not available
02) Intel(R) I210 Gigabit Network Connecti 8086-1533 08:00 Update not available
03) Intel(R) Ethernet Converged Network Ad 8086-1572 01:00 Update available

Options: Adapter Index List (comma-separated), [A]ll, e[X]it
Enter selection:a

Would you like to back up the NVM images? [Y]es/[N]o: y

Update in progress. This operation may take several minutes.
[**********]
Reboot is required to complete the update process.

Tool execution completed with the following status: All operations completed successfully

[root@esxi:/tmp/XL710/ESXi_x64] reboot
```
- [NVM Update Utility for Intel® Ethernet Converged Network Adapter XL710 and X710 Series](https://downloadcenter.intel.com/download/24769#help)

### Check Latest Version
```bash
[root@esxi:~] ethtool -i vmnic2
driver: i40e
version: 1.3.38
firmware-version: 5.05 0x800028ac 1.33.0
bus-info: 0000:01:00.1
```

## Reference
- [update intel drivers on vmware esxi](http://www.it-book.co.uk/2835/update-intel-drivers-on-vmware-esxi)
- [Intel® Ethernet Controller X710/XL710 and Intel® Ethernet Converged Network Adapter X710/XL710 Family: Linux* Performance Tuning Guide](http://www.intel.com.tw/content/www/tw/zh/embedded/products/networking/xl710-x710-performance-tuning-linux-guide.html)
