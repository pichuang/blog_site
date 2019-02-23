---
layout: post
title: Install i40e driver into VMWare ESXi
date: 2017-03-31 16:06:07
updated: 2017-03-31 16:06:07
tags:
- infra
- vmware
- intel
- driver
---

## Objective
## Environment
- ESXi-6.0.0-20160302001
- i40e
  - [i40e-1.4.28-1331820-4075203][1]
- Intel XL710
  - Including vmnic{2-5}

<!--more-->

## Step

### Check Status
```bash
[root@esxi-1:~] esxcli network nic list
Name    PCI Device    Driver  Admin Status  Link Status  Speed  Duplex  MAC Address         MTU  Description
------  ------------  ------  ------------  -----------  -----  ------  -----------------  ----  -------------------------------------------------
vmnic0  0000:08:00.0  igb     Up            Up             100  Full    8c:ea:1b:30:da:2b  1500  Intel Corporation I210 Gigabit Network Connection
vmnic1  0000:09:00.0  igb     Up            Down             0  Half    8c:ea:1b:30:da:2c  1500  Intel Corporation I210 Gigabit Network Connection
```
- Please enable SSH daemon on the server

### Download i40e driver and unzip
- Please download i40e driver from [here][1] first
- Unzip and transfer to ESXi 
``` bash
[pichuang@nu11:~] unzip i40e-1.4.28-1331820-4075203.zip
Archive:  i40e-1.4.28-1331820-4075203.zip
 extracting: i40e-1.4.28-1331820-offline_bundle-4075203.zip
 extracting: net-i40e-1.4.28-1OEM.550.0.0.1331820.x86_64.vib
 extracting: doc/README.txt
 extracting: source/driver_source_net-i40e_1.4.28-1OEM.550.0.0.1331820.tgz
 extracting: doc/open_source_licenses_net-i40e_1.4.28-1OEM.550.0.0.1331820.txt
 extracting: doc/release_note_net-i40e_1.4.28-1OEM.550.0.0.1331820.txt

[pichuang@nu11:~] scp i40e-1.4.28-1331820-offline_bundle-4075203.zip root@esxi-1.roadshow.ec:/tmp/
i40e-1.4.28-1331820-offline_bundle-4075203.zip                                  100%  141KB   9.9MB/s
```

### Install VIB 
```
[root@esxi-1:~] esxcli software vib install -d /tmp/i40e-1.4.28-1331820-offline_bundle-4075203.zip
Installation Result
   Message: The update completed successfully, but the system needs to be rebooted for the changes to be effective.
   Reboot Required: true
   VIBs Installed: Intel_bootbank_net-i40e_1.4.28-1OEM.550.0.0.1331820
   VIBs Removed:
   VIBs Skipped:
[root@esxi-1:~] reboot
```

### Check NIC Status
```bash
[root@esxi-1:~] esxcli network nic list
Name    PCI Device    Driver  Admin Status  Link Status  Speed  Duplex  MAC Address         MTU  Description
------  ------------  ------  ------------  -----------  -----  ------  -----------------  ----  ---------------------------------------------------------
vmnic0  0000:08:00.0  igb     Up            Up             100  Full    8c:ea:1b:30:da:2b  1500  Intel Corporation I210 Gigabit Network Connection
vmnic1  0000:09:00.0  igb     Up            Down             0  Half    8c:ea:1b:30:da:2c  1500  Intel Corporation I210 Gigabit Network Connection
vmnic2  0000:01:00.0  i40e    Up            Down             0  Half    cc:37:ab:dd:f1:e5  1500  Intel Corporation Ethernet Controller X710 for 10GbE SFP+
vmnic3  0000:01:00.1  i40e    Up            Down             0  Half    cc:37:ab:dd:f1:e6  1500  Intel Corporation Ethernet Controller X710 for 10GbE SFP+
vmnic4  0000:01:00.2  i40e    Up            Down             0  Half    cc:37:ab:dd:f1:e7  1500  Intel Corporation Ethernet Controller X710 for 10GbE SFP+
vmnic5  0000:01:00.3  i40e    Up            Down             0  Half    cc:37:ab:dd:f1:e8  1500  Intel Corporation Ethernet Controller X710 for 10GbE SFP+
```

## Reference
- [VMware ESXi 5.5 i40e 1.4.28 NIC Driver for Intel(R) Ethernet Controllers X710 and XL710 family][1]
- [VMware Compatibility Guide - Intel(R) Ethernet Controller XL710 for 10GbE QSFP+][2]
- [Intel Server Adapter XL710 Card (NIC) Driver for Vmware ESXi 5.5 & 6 - ThinkServer Systems][3]


[1]: https://my.vmware.com/web/vmware/details?downloadGroup=DT-ESXI55-INTEL-I40E-1428&productId=353
[2]: http://www.vmware.com/resources/compatibility/detail.php?deviceCategory=io&productid=37993
[3]: http://support.lenovo.com/tw/zh/downloads/ds112423