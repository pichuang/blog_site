---
layout: post
title:  "OpenvSwitch ovs-ofctl and OF-DPA"
date:   2017-01-06 13:42:10 +0800
updated: 2017-03-21 00:00:00 +0800
tags:
- sdn
- ovs
- ofdpa
- switch
- ovs-ofctl
---

## Goal
Remote control and management OF-DPA via ovs-ofctl

<!--more-->

## Environment
- Switch
  - IP: 192.168.13.14
  - Hardware
    - Model
      - [Edgecore AS6712-32X](http://www.edge-core.com/productsInfo.php?cls=1&cls2=7&cls3=43&id=12)
  - Software
    - ONL version
      - [ONL-2.0.0-ONL-OS-DEB8-2016-12-22.1828-604af0c-AMD64-INSTALLED-INSTALLER](http://opennetlinux.org/binaries/2016.12.22.18.28.604af0c9b3dc9504870c30273ab22f2fb62746c3/ONL-2.0.0-ONL-OS-DEB8-2016-12-22.1828-604af0c-AMD64-INSTALLED-INSTALLER)
    - OF-DPA version
      - [ofdpa_3.0 EA4](https://github.com/onfsdn/atrium-docs/blob/master/16A/ONOS/builds/ofdpa_3.0.4.0%2Baccton1.0~1-1_amd64.deb)
    - `You need launch ofappagent first`. If you dont know how to start it. Please refer [OF-DPA with ONL Cheat Sheet](http://blog.pichuang.com.tw/ofdpa-with-onl-cheat-sheet)
    - Command
      - `launcher ofagentapp -a2 -d4 -c1 -c2 -c3 -c4 -c5 -l 0.0.0.0:6653`
- Contorl VM
  - IP: 192.168.13.2
  - OpenvSwitch
    - 2.7.0
    - In this case, we just use `ovs-ofctl`, and it's also work if you want to use dpctl which CPqD provide
    - [How to Install OpenvSwitch][3]

### Basic Command
```bash
ovs-ofctl show -O OpenFlow13 tcp:192.168.13.14:6653
```

### Usage

#### Show port information
- `ovs-ofctl show -O OpenFlow13 tcp:192.168.13.14:6653`
- Output
```bash
$ ovs-ofctl show -O OpenFlow13 tcp:192.168.13.14:6653
OFPT_FEATURES_REPLY (OF1.3) (xid=0x2): dpid:000000000000da7a
n_tables:23, n_buffers:0
capabilities: FLOW_STATS TABLE_STATS PORT_STATS QUEUE_STATS
OFPST_PORT_DESC reply (OF1.3) (xid=0x3):
 1(port1): addr:cc:37:ab:d0:d6:30
     config:     0
     state:      LINK_DOWN
     current:    40GB-FD FIBER
     supported:  10MB-FD 100MB-FD 1GB-FD 10GB-FD 40GB-FD OTHER FIBER AUTO_NEG AUTO_PAUSE AUTO_PAUSE_ASYM
     speed: 40000 Mbps now, 40000 Mbps max
 2(port2): addr:cc:37:ab:d0:d6:30
     config:     0
     state:      LINK_DOWN
     current:    40GB-FD FIBER
     supported:  10MB-FD 100MB-FD 1GB-FD 10GB-FD 40GB-FD OTHER FIBER AUTO_NEG AUTO_PAUSE AUTO_PAUSE_ASYM
     speed: 40000 Mbps now, 40000 Mbps max
...
```

#### Show tables information
- `ovs-ofctl dump-tables`
- Output
```bash
$ ovs-ofctl dump-tables -O OpenFlow13 tcp:192.168.13.14:6653
OFPST_TABLE reply (OF1.3) (xid=0x2):
  table 0:
    active=0, lookup=0, matched=0

  table 5: ditto
  table 6: ditto
  table 7: ditto
  table 8: ditto
  table 10: ditto
  table 11: ditto
  table 13: ditto
  table 15: ditto
  table 16: ditto
  table 17: ditto
  table 20: ditto
  table 24: ditto
  table 28: ditto
  table 30: ditto
  table 40: ditto
  table 50: ditto
  table 60: ditto
  table 65: ditto
  table 210: ditto
  table 211: ditto
  table 230: ditto
  table 235: ditto
```

#### Add flow entry
- `ovs-ofctl add-flow -O OpenFlow13 tcp:192.168.13.14:6653 table=60,priority=40000,eth_type=0x0800,ip_dst=55.55.55.55,actions=controller`

#### Show flow entries
- `ovs-ofctl dump-flows`
- Output
```bash
$ ovs-ofctl dump-flows -O OpenFlow13 tcp:192.168.13.14:6653
OFPST_FLOW reply (OF1.3) (xid=0x2):
 cookie=0x0, duration=46.570s, table=60, n_packets=0, n_bytes=0, priority=40000,ip,nw_dst=55.55.55.55 actions=CONTROLLER:65535
```

#### Show queue stats
- `ovs-ofctl queue-stats`
- Output
```bash
$ ovs-ofctl queue-stats -O OpenFlow13 tcp:192.168.13.14:6653
OFPST_QUEUE reply (OF1.3) (xid=0x2): 256 queues
  port 1 queue 0: bytes=0, pkts=0, errors=0, duration=597.4294513152s
  port 1 queue 1: bytes=0, pkts=0, errors=0, duration=597.4294513152s
  port 1 queue 2: bytes=0, pkts=0, errors=0, duration=597.4294513152s
  port 1 queue 3: bytes=0, pkts=0, errors=0, duration=597.4294513152s
...
```

#### Show group information
- `ovs-ofctl dump-groups`

#### Show ports information
- `ovs-ofcll dump-ports`
- Output
```bash
$ ovs-ofctl dump-ports -O OpenFlow13 tcp:192.168.13.14:6653
OFPST_PORT reply (OF1.3) (xid=0x2): 32 ports
  port  1: rx pkts=0, bytes=0, drop=0, errs=0, frame=0, over=0, crc=0
           tx pkts=0, bytes=0, drop=0, errs=?, coll=0
           duration=0s
  port  2: rx pkts=0, bytes=0, drop=0, errs=0, frame=0, over=0, crc=0
           tx pkts=0, bytes=0, drop=0, errs=?, coll=0
           duration=0s
  port  3: rx pkts=0, bytes=0, drop=0, errs=0, frame=0, over=0, crc=0
           tx pkts=0, bytes=0, drop=0, errs=?, coll=0
           duration=0s
  port  4: rx pkts=0, bytes=0, drop=0, errs=0, frame=0, over=0, crc=0
           tx pkts=0, bytes=0, drop=0, errs=?, coll=0
           duration=0s
...
```

## Reference
- [ovs-ofctl][1]
- [Pica8 OVS Configuration Guide 2.8.0][2]
- [Compile OpenvSwitch - pichuang][3]
- [ovs-ofctl help][4]

[1]: http://openvswitch.org/support/dist-docs/ovs-ofctl.8.txt
[2]: http://www.pica8.com/wp-content/uploads/2015/09/v2.8/html/ovs-configuration-guide/
[3]: http://blog.pichuang.com.tw/compile-openvswitch/
[4]: https://gist.github.com/pichuang/a901ea8d9870728bf8eebaab548e037c