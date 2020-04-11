---
layout: post
title:  "OF-DPA with ONL Cheat Sheet"
date:   2017-01-04 10:42:10 +0800
updated: 2017-01-04 00:00:00 +0800
tags:
  - cicada
  - onl
  - ofdpa
  - sdn
categories:
  - infra
---

## Environment
- Hardware
  - Model
    - [Edgecore AS5712-54X](http://www.edge-core.com/productsInfo.php?cls=1&cls2=8&cls3=44&id=15)
- Software
  - ONL version
    - [ONL-2.0.0-ONL-OS-DEB8-2017-07-18.1642-40fc82b](https://opennetlinux.org/binaries/2017.07.18.1642.40fc82b48cabf8b14aa5d16d9dfa47c50a8c95a6)
  - OF-DPA version 
    - [ofdpa_3.0 EA5](https://github.com/onfsdn/atrium-docs/blob/master/16A/ONOS/builds/ofdpa_3.0.5.5+accton1.7-1_amd64.deb?raw=true)

<!--more-->

### Set Base configuration
1. vi /etc/network/interfaces
```bash
auto ma1
iface ma1 inet static
address 192.168.11.2
netmask 255.255.255.0
gateway 192.168.11.254
dns-nameservers 192.168.100.1
```

### Rate Limit
- Setting Rate Limit per queue 
  - `client_queue_config <queueId> <intIfNum> <minRate> <maxRate>` 
    - queyeID := [ 0 - 7 ] 
  - ex: `client_queue_config 0 2 1 1000` 
    - Queue 0, Port 2, 1 means minimal 0.1%, 1000 means Maximum 100%
    - Min and Max queue rates must range between 1-1000 
- Show Rate Limit per queue 
  - `client_queue_config <queueId> <intIfNum>` 
  - ex: `client_queue_config 1 1` 
    - Show Queue 1, Port 1 min/Max rate 

### OF Agent App
- Enable ofagentapp service
  - via CLI 
    - `launcher ofagentapp --controller=<CONTROLLER_IP>:6653 --listen=<SWITCH_MA1_IP>:6653`
  - via INIT
    - `service ofagentd restart`
    - The ofagent.conf at `/etc/ofagent/ofagent.conf`
  - More detailes [How to enable debugging mode on OF-DPA][2]
- Stop ofagentapp
  - `pkill ofagentapp`

### Port Speed
- Change Port Speed per Port
  - vi /etc/accton/ofdpa.conf
```bash
...
# port_speed_<port>= Speed
# Speed := { 1000 | 10000 | 40000 }
...
# Default value in AS5712-54X is port_speed=10000
port_speed_3=1000   # front port 3
port_speed_4=10000   # front port 4
...
```

- Change Port Mode per Port
  - vi /etc/accton/ofdpa.conf
```bash
...
# port_mode_<port> = Speed_Type [Interface Type]
# Speed_Type:= { 1x40g | 4x10g }
# Interface Type := { CR | CR4 | SR | SR4 | LR | LR4 | KR | KR4 | SFI | XFI | ... }
...
port_mode_1=1x40g
port_mode_2=1x40g if=SR4
port_mode_3=4x10g #Breakout cable
...
```

### Dump BRCM Informations
- Dump Port Table
  - `client_port_table_dump`
- Dump Meter Table
  - `client_meter_dump`
- Dump Flow Table
  - `client_flowtable_dump`
- Dump Group Table
  - `client_grouptable_dump`
- Dump Tunnel
  - `client_tunnel_dump`
- Dump OAM
  - `client_oam_dump`
- Dump Class Color Table
  - `client_classcolortable_dump` 
- Show physical port mapping
  - `client_drivport -a`
- Enable sFlow
  - `client_sflow`
- Purge all configuration
  - `client_cfg_purge`

### BRCM Diag Shell with ONL
- NOTE: Please launch `ofagentapp` first in backaground
- Show processes status
  - `client_drivshell ps`
- Show port config
  - `client_drivshell port <port_number>`
- Show usage command
  - `client_drivshell help` 
- Redirect log to other devices (ex: /dev/console)
  - `client_drivshell log f=/dev/console`
    - You can use command `tty` to find your teletype device name currently
    - Tips: `client_drivshell log f=$(tty)`

### NOS Information
- Show Platform Information with ONL
  - `onl-platform-show`
- Show sysconfig with ONL
  - `onl-sysconfig`

### Mount Disk
- Change w/r permission on disk
  1. `vi /etc/mtab.yml`
  2. `onl-mount mount all`
- Check mount status
  - `mount`
- Check partition status
  - `df -h`

### ONL Platform Information
- Show ONL information for human read 
  - `onlpdump -r`
- Show Hardware information
  - `onlpdump -s`
- Show SFP information
  - `onlpdump -S`

### Only launch OF-DPA without OpenFlow
- Start OF-DPA without OpenFlow agent
```bash
KERNEL_MODS=/lib/modules/`uname -r`/ofdpa
insmod $KERNEL_MODS/linux-kernel-bde.ko dmasize=32M maxpayload=128
insmod $KERNEL_MODS/linux-user-bde.ko
```

- Check Status
```bash
lsmod
client_drivshell ps
```

## Flow tables ID (OF-DPA 3.0.3)

|Table Name | Table ID|
|-----------|---------|
|Ingress Port | 0 |
|Port DSCP Trust | 5|
|Port PCP Trust | 6|
|Tunnel DSCP Trust | 7|
|Tunnel PCP Trust | 8|
|Injected OAM| 9|
|VLAN | 10|
|VLAN 1 | 11|
|Ingress Maintenance Point | 12|
|MPLS L2 Port | 13|
|MPLS DSCP Trust | 15|
|MPLS PCP Trust | 16|
|L2 Policer | 18|
|L2 Policer Actions | 19|
|Termination MAC | 20|
|L3 Type| 21|
|MPLS 0 | 23|
|MPLS 1 | 24|
|MPLS 2 | 25|
|MPLS-TP Maintenance Point | 26|
|MPLS L3 Type| 27|
|MPLS Label Trust | 28|
|MPLS Type | 29|
|Unicast Routing | 30|
|Multicast Routing | 40|
|Bridging | 50|
|Policy ACL | 60|
|Color Based Actions | 65|
|Egress VLAN | 210|
|Egress VLAN 1 | 211|
|Egress Maintenance Point| 226|
|Egress DSCP PCP Remark | 230|
|Egress TPID | 235|

## Reference
- [OF-DPA文档读后感][1]
- [How to enable debugging mode on OF-DPA][2]

[1]: http://www.lai18.com/content/9682257.html
[2]: https://edge-core.github.io/How-to-enable-debugging-mode-on-OF-DPA/