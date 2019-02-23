---
layout: post
title: "SDN Lab 7$ L2 Switching and Queue using OFDPA Switch and ovs-ofctl"
date: 2017-01-09 11:19:45 +0800
updated: 2017-01-09 00:00:00 +0800
description: ""
tags:
- sdn
- ofdpa
- ovs-ofctl
- queue
- openflow
---

(Working...)

## Environment
- Switch
  - IP: 192.168.11.2
  - Hardware
    - Model
      - [Edgecore AS5712-54X](http://www.edge-core.com/productsInfo.php?cls=1&cls2=8&cls3=44&id=15)
  - Software
    - ONL version
      - [ONL-2.0.0-ONL-OS-DEB8-2016-12-22.1828-604af0c-AMD64-INSTALLED-INSTALLER](http://opennetlinux.org/binaries/2016.12.22.18.28.604af0c9b3dc9504870c30273ab22f2fb62746c3/ONL-2.0.0-ONL-OS-DEB8-2016-12-22.1828-604af0c-AMD64-INSTALLED-INSTALLER)
    - OF-DPA version
      - [ofdpa_3.0 EA3](https://github.com/onfsdn/atrium-docs/blob/master/16A/ONOS/builds/ofdpa_3.0.3.1%2Baccton1.4~1-1_amd64.deb)
    - `You need launch ofappagent first`. If you dont know how to start it. Please refer [OF-DPA with ONL Cheat Sheet](http://blog.pichuang.com.tw/ofdpa-with-onl-cheat-sheet)
- Contorl VM
  - IP: 192.168.11.1
  - OpenvSwitch
    - 2.6.9

## Topology

```diatt
 +--------+   Port+-----------------------+Port   +--------+
 |        |      1|    AS5712-54X         |2      |        |
 | Host A +-------+    ONL: 2.0.0 Deb8    +-------+ Host B |
 |        |       |    OFDPA: 3.0.3 EA2   |       |        |
 +--------+       +-----------------------+       +--------+
```

### L2 Switching
```
ovs-ofctl
```

## Flow tables ID

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

## Setting
```

```

## References
- [OF-DPA OpenFlow Hybrid Switch Functionality Guide](https://github.com/open-switch/ops/blob/master/docs/ofdpa_hybrid_functionality_guide.md#overview)
- [OpenvSwitch ovs-ofctl and OF-DPA](http://blog.pichuang.com.tw/ovs-ofctl-and-ofdpa)
