---
layout: post
title: 通用抽象網路邏輯介紹
author: Phil Huang
toc: true
tags:
  - redhat
  - network
  - openshift
categories:
  - misc
date: 2020-06-02 00:31:45
udpated: 2020-06-02 00:31:45
---

> 這世界上沒有完美的架構，只有權衡後的辦法，但...

來一個圖文不符的 RHEL 網路線路圖，架構圖百百種，每個角色跟角度看的樣子都會不太一樣，這張圖目的是要呈現 OS 上的網卡跟網段對應的關係，算是蠻常見的網路架構表示法，一般習慣上我都是畫成像是五線譜ㄧ樣，比較好清楚表示每個角色的相呼關係，大家可以參考參考

![](/images/network-logical.png)

但今天要特別講一下關於 OpenShift 本身的角度，我是怎麼看待這些抽象網路邏輯的，這跟如果要整第三方 SDN 解決方案有直接的關係

<!--more-->

## RHEL 網路流量抽象邏輯

其實因為他本身就是一台 VM 你加多少網卡，決定該網卡要幹嘛，他就是什麼角色，就像上面那張圖一樣，這比較沒什麼特別的

## OpenShift 網路流量抽象邏輯

以一個 `OpenShift 的 Networking` 為觀點出發的話，針對網路的流量分類，常見會分為下面 3 + 1 種但不限於之類型

1. 資料層 (Data Plane, DP)：有時候有會被叫做轉發層 (Forwarding Plane, FP) 或 User Plane (UP)，主要講的就是實際上資料封包傳輸所流經的路線

```bash
$ oc get daemonset.apps/ovs -n openshift-sdn
NAME   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
ovs    5         5         5       5            5           kubernetes.io/os=linux   77d

$ oc get daemonset.apps/sdn -n openshift-sdn
NAME   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
sdn    5         5         5       5            5           kubernetes.io/os=linux   77d
```

如果更詳細要講的話，其實就是透過基於 CNI 定義下實做出來的網路模組，如預設的 `ovs-networkpolicy`，裡面就是節點間建立歸屬能辦到 L2 over L3 的 VXLAN 進行 Pod to Pod 或 Pod to Services 等 Kubernetes 內建之機制溝通

2. 控制層 (Control Plane, CP)：主要是針對 Data Plane 的資料流控制進行能管理和上一些規範，在 OpenShift 角度上來說就是

```bash
$ oc get daemonset.apps/sdn-controller -n openshift-sdn
NAME             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
sdn-controller   2         2         2       2            2           node-role.kubernetes.io/master=   77d
```

3. 管理層 (Management Plane, MP)：能夠管理和監視 Control Plane 的路線，譬如透過 SSH / SNMP / Telnet 登入管理皆屬此類

```bash
$ ssh core@control-plane-2 -i ~/.ssh/id_rsa_ocp4_vcenter.pub
Red Hat Enterprise Linux CoreOS 44.81.202005062110-0
  Part of OpenShift 4.4, RHCOS is a Kubernetes native operating system
  managed by the Machine Config Operator (`clusteroperator/machine-config`).

WARNING: Direct SSH access to machines is not recommended; instead,
make configuration changes via `machineconfig` objects:
  https://docs.openshift.com/container-platform/4.4/architecture/architecture-rhcos.html

---
Last login: Sat May 30 15:21:52 2020 from 10.0.97.100
[core@control-plane-2 ~]$
```

4. 儲存層 (Storage Plane, SP)：嚴格來說，這個不算在常見的網路架構中，連 OpenShift 網路都沒有特別定義，這是從 Data Plane 拆分一個角色出來，特別針對 OS 跟儲存設備中間拉一個專門的網路，譬如你的 nfs mount point `通常`都不會跟你的一般流量混在一起

### 所以?

正常 OpenShift 架構下，其實 CP/MP/DP 都是透過同一個 IP 出去溝通的，有特別要拆開的話就會跟其他家的 SDN 廠商合作了，譬如像

- [Cisco ACI][3]

![](/images/cisco-aci.png)

- [Juniper Contrail][4]

![](/images/juniper-contrail.png)

- [VMware NSX-T][5]

![](https://blogs.vmware.com/networkvirtualization/files/2019/02/Screen-Shot-2019-02-11-at-16.47.08-1024x553.png)

如果想要了解更多 CNI 的話，可以參考一下這份列表 [容器平台強固化保護思路][5]

## 後話

最後啊...要比較留意就是

> 不同的設備觀點，對於 CP / DP / MP 會有不一樣的解釋，所以要講架構前一定要很清楚知道是從哪個設備觀點出發

譬如說以我比較熟的 SDN 架構來說，對照如下

![](http://thenewstack.io/wp-content/uploads/2014/11/img1.png)

- DP: SDN Switch，實際轉發跟處理封包的路線
- CP: SDN Controller，負責決定 SDN Switch 內部資料流量的走向
- MP: SDN Switch 內建的 SNMP / Telnet / SSH 等標準協定可供日常管理的路線

當然啦，現在大多數的架構，都還是混合在一起的，總是有利有弊

我開頭講了一段好像很有道理的話，但後面缺了一小段，有興趣知道的可以當面找我聊 XD

## References
- [Difference between control plane, data plane and management plane?][1]
- [SDN Series Part One: Defining Software Defined Networking][2]
- [Cisco - ACI Plugin for Red Hat OpenShift Container Architecture and Design Guide][3]
- [容器平台強固化保護思路 - Phil Huang][5]

[1]: https://networkengineering.stackexchange.com/questions/38573/difference-between-control-plane-data-plane-and-management-plane
[2]: https://thenewstack.io/defining-software-defined-networking-part-1/
[3]: https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/white_papers/Cisco-ACI-CNI-Plugin-for-OpenShift-Architecture-and-Design-Guide.html
[4]: https://www.juniper.net/documentation/en_US/contrail20/topics/task/configuration/install-openshift-using-anible-311.html
[5]: https://blog.pichuang.com.tw/20190723-container-and-container-platform-hardening/