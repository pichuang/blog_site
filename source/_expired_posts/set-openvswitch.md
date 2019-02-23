---
layout: post
title: 設定 OpenvSwitch
date: 2014-04-21 23:29:00
updated: 2014-04-21 23:29:00
comments: true
categories: sdn
tags:
- sdn
- ovs
---
# 查表
- `ovs-vsctl list bridge ovs-br`

# 關於 Brdige 及 Port
1. 新增 Brdige
	- `ovs-vsctl add-br ovs-br`
2. 在 ovs-br 上對應 interface
  - `ovs-vsctl add-port ovs-br eth0`
3. (1) + (2) 的寫法可為
	- `ovs−vsctl add−br ovs-br -- add−port ovs-br eth0`
4. 移除 Bridge
	- `ovs-vsctl del-br ovs-br` #如果不存在的話, 會有error log
  - `ovs-vsctl --if-exists del-br ovs-br`
5. 更改 ofport (openflow port number) 為 100
  - `ovs-vsctl add-port ovs-br eth0 -- set Interface eth0 ofport_request=100`
6. 設定 port 為 internal 
  - `ovs-vsctl set Interface eth0 type=internal`

# 關於 Controller
1. 設定 Controller
	- `ovs-vsctl set-controller ovs-br tcp:1.2.3.4:6633`
2. 設定 multi controller
  - `ovs-vsctl set-controller ovs-br tcp:1.2.3.4:6633 tcp:5.6.7.8:6633`
3. 查詢 Controller 設定
  - `ovs-vsctl show`
		- 如果有成功連到 controller 則會顯示 `is_connected:true`, 反之則未連上
  - `ovs-vsctl get-controller ovs-br`
4. 移除 Controller
	- `ovs-vsctl del-controller ovs-br`

# 關於 STP (Spanning Tree Protocol)
1. 開啟 STP 
	- `ovs-vsctl set bridge ovs-br stp_enable=true`
2. 關閉 STP
  - `ovs-vsctl set bridge ovs-br stp_enable=false`
3. 查詢 STP 設定值
  - `ovs-vsctl get bridge ovs-br stp_enable`
4. 設定 Priority
  - `ovs−vsctl set bridge br0 other_config:stp-priority=0x7800`
5. 設定 Cost
  - `ovs−vsctl set port eth0 other_config:stp-path-cost=10`
6. 移除 STP 設定
  - `ovs−vsctl clear bridge ovs-br other_config`

# 關於 Openflow Version
1. 支援 OpenFlow Version 1.3
	- `ovs-vsctl set bridge ovs-br protocols=OpenFlow13`
2. 支援 OpenFlow Version 1.3 1.2
	- `ovs-vsctl set bridge ovs-br protocols=OpenFlow12,OpenFlow13`
3. 移除 OpenFlow 支援設定
	- `ovs-vsctl clear bridge ovs-br protocols`

# 關於 VLAN
1. 設定 VLAN tag
	- `ovs-vsctl add-port ovs-br vlan3 tag=3 -- set interface vlan3 type=internal`
2. 移除 VLAN
	- `ovs-vsctl del-port ovs-br vlan3`
3. 查詢 VLAN
  - `ovs-vsctl show`
  - `ifconfig vlan3`
4. 設定 Vlan trunk
  - `ovs-vsctl add-port ovs-br eth0 trunk=3,4,5,6`
5. 設定已 add 的 port 為 access port, vlan id 9
  - `ovs-vsctl set port eth0 tag=9`
6. ovs-ofctl add-flow 設定 vlan 100
	- `ovs-ofctl add-flow ovs-br in_port=1,dl_vlan=0xffff,actions=mod_vlan_vid:100,output:3`
  - `ovs-ofctl add-flow ovs-br in_port=1,dl_vlan=0xffff,actions=push_vlan:0x8100,set_field:100-\>vlan_vid,output:3`
7. ovs-ofctl add-flow 拿掉 vlan tag
  - `ovs-ofctl add-flow ovs1 in_port=3,dl_vlan=100,actions=strip_vlan,output:1`
  - [two_vlan example](https://gist.github.com/pichuang/b1fc5ebfab471ef896b3)
8. ovs-ofctl add-flow pop-vlan
	- `ovs-ofctl add-flow ovs-br in_port=3,dl_vlan=0xffff,actions=pop_vlan,output:1`
  
# 關於 GRE Tunnel
1. 設定 GRE tunnel
	- `ovs−vsctl add−port ovs-br ovs-gre -- set interface ovs-gre type=gre options:remote_ip=1.2.3.4`
2. 查詢 GRE Tunnel
	- `ovs-vsctl show`
 
# 關於 Dump flows 
1. Dumps OpenFlow flows 不含 hidden flows (常用)
  - `ovs-ofctl dump-flows ovs-br`
2. Dumps OpenFlow flows 包含 hidden flows
  - `ovs-appctl bridge/dump-flows ovs-br`
3. Dump 特定 bridge 的 datapath flows 不論任何 type  
  - `ovs-appctl dpif/dump-flows ovs-br`
4. Dump 在 Linux kernel 裡的 datapath flow table (常用)
  - `ovs-dpctl dump-flows [dp]`
5. Top like behavior for ovs-dpctl dump-flows
  - `ovs-dpctl-top`

# XenServer 開啓 OpenvSwitch 方式
1. 檢查開啟與否
  - `service openvswitch status`
2. 開啓
  - `xe-switch-network-backend openvswitch`
3. 關閉
  - `xe-switch-network-backend bridge`

# 關於 Log
1. 查詢 log level list
  - `ovs-appctl vlog/list`
2. 設定 log level (以 stp 設定 file 為 dbg level 為例)
  - `ovs-appctl vlog/set stp:file:dbg`
  - `ovs-appctl vlog/set {module name}:{console, syslog, file}:{off, emer, err, warn, info, dbg}`

# 關於 Fallback
1. Controller connection: false 的時候, 會自動調成 legacy switch mode
  - `ovs-vsctl set-fail-mode ovs-br standalone`
2. 無論 Controller connection status 為何, 都必須通過 OpenFlow 來進行網路行為 (default)
  - `ovs-vsctl set-fail-mode ovs-br secure`
3. 移除
  - `ovs-vsctl del-fail-mode ovs-br`
4. 查詢
  - `ovs-vsctl get-fail-mode ovs-br`

# 關於 sFlow 
1. 查詢
  - `ovs-vsctl list sflow`
2. 新增
  - [Set sFlow](https://gist.github.com/pichuang/11332074)
3. 刪除 
  - `ovs-vsctl -- clear Bridge ovs-br sflow`

# 關於 NetFlow
1. 查詢
  - `ovs-vsctl list netflow`
2. 新增
  - [Set NetFlow](https://gist.github.com/pichuang/11331998)
3. 刪除
  - `ovs-vsctl -- clear Bridge ovs-br netflow`

# 設定 Out-of-band 和 in-band
1. 查詢
  - `ovs-vsctl get controller ovs-br connection-mode`
2. Out-of-band
  - `ovs-vsctl set controller ovs-br connection-mode=out-of-band`
3. In-band (default)
  - `ovs-vsctl set controller ovs-br connection-mode=in-band`
4. 移除 hidden flow
  - `ovs-vsctl set bridge br0 other-config:disable-in-band=true`

# 關於 ssl 
1. 查詢
  - `ovs-vsctl get-ssl`
2. 設定
  - `ovs-vsctl set-ssl sc-privkey.pem sc-cert.pem cacert.pem`
  - [OpenvSwitch Lab 6$ TLS SSL](http://roan.logdown.com/posts/208707-openvswitch-lab-6-ssl)
3. 刪除
	- `ovs-vsctl del-ssl`

# 關於 SPAN
1. 詳細設定

```
    ovs-vsctl add-br ovs-br
    ovs-vsctl add-port ovs-br eth0
    ovs-vsctl add-port ovs-br eth1
    ovs-vsctl add-port ovs-br tap0 \
         -- --id=@p get port tap0 \
         -- --id=@m create mirror name=m0 select-all=true output-port=@p \
         -- set bridge ovs-br mirrors=@m
```
  * 將 ovs-br 上 add-port {eth0,eth1} mirror 至 tap0

2. 刪除
  - `ovs-vsctl clear bridge ovs-br mirrors`
# 關於 Table
1. 查 table
```ovs-ofctl dump-tables ovs-br```

# 關於 Group Table
參考 [hwchiu - Multipath routing with Group table at mininet](http://hwchiu.logdown.com/posts/207387-multipath-routing-with-group-table-at-mininet)

1. 建立 Group id 及對應的 bucket
  - `ovs-ofctl -O OpenFlow13 add-group ovs-br group_id=5566,type=select,bucket=output:1,bucket=output:2,bucket=output:3`
  - type 共有 All, Select, Indirect, FastFailover, [詳細規格](http://flowgrammable.org/sdn/openflow/message-layer/groupmod/#GroupMod_1.3)

2. 使用 Group Table
  - `ovs-ofctl -O OpenFlow13 add-flow ovs-br in_port=4,actions=group:5566`

# 關於 VXLAN
參考 [rascov - Bridge Remote Mininets using VXLAN](http://rascov.logdown.com/posts/230635-bridge-remote-networks-using-vxlan)

1. 建立 VXLAN Network ID (VNI) 和指定的 OpenFlow port number, eg: VNI=5566, OF_PORT=9
  - `ovs-vsctl set interface vxlan type=vxlan option:remote_ip=x.x.x.x option:key=5566 ofport_request=9`

2. VNI flow by flow
  - `ovs-vsctl set interface vxlan type=vxlan option:remote_ip=140.113.215.200 option:key=flow ofport_request=9`

3. 設定 VXLAN tunnel id
  - `ovs-ofctl add-flow ovs-br in_port=1,actions=set_field:5566->tun_id,output:2`
  - `ovs-ofctl add-flow s1 in_port=2,tun_id=5566,actions=output:1`

# 關於 OVSDB Manager
參考 [OVSDB Integration:Mininet OVSDB Tutorial](https://wiki.opendaylight.org/view/OVSDB_Integration:Mininet_OVSDB_Tutorial)

1. Active Listener 設定
  - `ovs-vsctl set-manager tcp:1.2.3.4:6640`
2. Passive Listener 設定
  - `ovs-vsctl set-manager ptcp:6640`

# OpenFlow Trace
1. Generate pakcet trace
  - `ovs-appctl ofproto/trace ovs-br in_port=1,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02 -generate`

# 其它
1. 查詢 OpenvSwitch 版本
  - `ovs-ofctl -V`
2. 查詢下過的指令歷史記錄 
  - `ovsdb-tool show-log [-mmm]`
  
# Reference
  - [ovs-vsctl](http://openvswitch.org/cgi-bin/ovsman.cgi?page=utilities%2Fovs-vsctl.8)
  - [OpenvSwitch FAQ](http://git.openvswitch.org/cgi-bin/gitweb.cgi?p=openvswitch;a=blob_plain;f=FAQ;hb=HEAD)
  - [OpenvSwitch Debugging](http://openvswitch.org/slides/OVS-Debugging-110414.pdf)
  - [Network flow monitoring with Open vSwitch](http://www.areteix.net/blog/2013/08/network-flow-monitoring-with-open-vswitch/)
  - [Pica8 OpenvSwitch configuration](http://www.pica8.com/document/pica8-OVS-MPLS-configuration-guide.pdf)
  - [hwchiu - Multipath routing with Group table at mininet](http://hwchiu.logdown.com/posts/207387-multipath-routing-with-group-table-at-mininet)
  - [rascov - Bridge Remote Mininets using VXLAN](http://rascov.logdown.com/posts/230635-bridge-remote-networks-using-vxlan)
  - [基于 Open vSwitch 的 OpenFlow 实践（陈沙克）](http://www.sdnap.com/sdn-technology/5114.html)
  - [OpenVswitch Advanced Tutioral](http://vlabs.cfapps.io/openvswitch/openvswitch_tutorial.html)