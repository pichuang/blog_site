layout: post
title: 使用 nmcli  設定 Bonding 或 Teaming
author: Phil Huang
tags:
  - nmcli
  - bonding
  - teaming
  - network
categories:
  - linux
date: 2018-08-26 14:58:00
---

## 前言

過往我們多數會採用已不再維護的 net-tools 套件底下的 `ifconfig` 或具備比前者更豐富且有社群維護的 `ip` 來設定網路，但熟悉的管理者應該也會知道透過這些工具所執行的設定只是暫時性地，重開機就會不見了，那要如何在 Red Hat Enterprise Linux 7 (RHEL 7) 中設定永久地網路設定呢? 本文主要採用 `nmcli` 指令來設定 bonding 和 teaming

<!--more-->

在 RHEL 7 裡，`nmcli` 可以提供你指令去操作 Network Manager Service (可用 `systemctl status NetworkManager.service` 來查詢狀態)，也可以透過此指令去產生正確的網路設定檔出來 (位置於 `/etc/sysconfig/network-scripts/*`)，若是不習慣純指令的方式，也可用 `nmtui` 呼叫文字使用介面 (Text Mode User Interface) 呼叫設定畫面出來

本文將基於 `nmcli` 介紹如何設定
1. 新增 ethernet interface
2. 新增 bonding interface
3. 新增 teaming interface

## Get Started
### 獲得 Network Device 的狀態


看看現在 RHEL 7 認得的這些網卡狀態，想要了解各網卡介面的狀態可以看這裡

```bash
$ nmcli device status
DEVICE      TYPE      STATE         CONNECTION
eth0        ethernet  connected     eth0
eth1        ethernet  disconnected  --
eth2        ethernet  disconnected  --
eth3        ethernet  disconnected  --
```

### 新增 Ethernet Interface

本範例新增一個 connection profile 名字叫做 lab-eth1，指定 eth1 介面的 ip 跟 gateway

```bash
$ nmcli connection add con-name lab-eth1 ifname eth1 type ethernet ip4 192.168.122.100/24 gw4 192.168.122.1
Connection 'lab-eth1' (e93927d0-4ef6-4640-90c5-8dbcd24c00e0) successfully added.
```

`nmcli` 會自動地建立 connection profile 出來，也就是大家比較熟悉的網路設定檔，詳細請參考下列輸出

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-lab-eth1
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
IPADDR=192.168.122.100
PREFIX=24
GATEWAY=192.168.122.1
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=lab-eth1
UUID=b1814873-05cf-4e18-9b82-4c8d09e9b4dd
DEVICE=eth1
ONBOOT=yes
```

依據 Connection Profile - lab-eth1 的描述啟動網路介面

```bash
$ nmcli connection up lab-eth1
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/19)
```

### 新增 Bonding Interface

一樣師法新增 Ethernet Interface 的方式，來新增 Bonding Interface，目前有支援七種 bonding 模式

```bash
$  nmcli con add type bond ifname bond0 mode [按tab]
802.3ad        active-backup  balance-alb    balance-rr     balance-tlb    balance-xor    broadcast
```

本範例新增一個 bond0 介面，模式使用 802.3ad，nmcli 會自動產生一個 Connection Profile `bond-bond0`

```bash
$  nmcli con add type bond ifname bond0 mode 802.3ad
Connection 'bond-bond0' (e817f31b-757e-4903-9230-4ac4eff7101b) successfully added.
```

將 eth2 及 eth3 加入至 bond0 裡面

```bash
$  nmcli con add type bond-slave ifname eth2 master bond0
Connection 'bond-slave-eth2' (cc06f17d-95b2-401e-a2d4-1748ff4f24b4) successfully added.
$  nmcli con add type bond-slave ifname eth3 master bond0
Connection 'bond-slave-eth3' (1b2c4b31-7294-40fc-8f24-a33c98265625) successfully added.
```

啟動介面，順序從 Slave Interface 到 Bonding Interface

```bash
$ nmcli con up bond-slave-eth2
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/23)
$ nmcli con up bond-slave-eth3
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/24)
$ nmcli con up bond-bond0
Connection successfully activated (master waiting for slaves) (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/25)
```

記得要給 bond0 介面一個 ip

```bash
$ nmcli con mod bond-bond0 ipv4.addresses 192.168.122.101/24
$ nmcli con mod bond-bond0 ipv4.method manual
$ nmcli con up bond-bond0
```

因為 bonding 是實作於 kernel module 裡面，故若發現啟動失敗，可以透過 `lsmod |grep bonding` 來檢查有沒有正確 load module，這邊就不做過多闡述。若想要檢查 bonding interface 的狀況則可以去 `/proc/net/bonding/*` 底下檢查內容

```bash
$ cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: IEEE 802.3ad Dynamic link aggregation
Transmit Hash Policy: layer2 (0)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

802.3ad info
LACP rate: slow
Min links: 0
Aggregator selection policy (ad_select): stable
System priority: 65535
System MAC address: 2c:c2:60:27:0a:9b
Active Aggregator Info:
        Aggregator ID: 1
        Number of ports: 1
        Actor Key: 0
        Partner Key: 1
        Partner Mac Address: 00:00:00:00:00:00

Slave Interface: eth2
MII Status: up
...
```

### 新增 Teaming Interface

Network Teaming 技術來自於 `libteam` 這個模組，提供比既有 linux bonding 效能和功能還要好的服務，功能上是可以完全取代 linux bonding，主要的優勢在於 Data aggregation 和 failover 這兩者功能皆較為突出，詳細比較可以參閱官方比較表 [Bonding vs. Team features - jpirko/libteam][2]，而 RHEL7 預設是**建議**採用此方案

Network Teaming 使用 teamd.service 來進行 driver 控制及設定，故在使用該技術之前需要先安裝 teamd 這個模組

```bash
$ yum install -y teamd
$ systemctl list-unit-files | grep teamd
teamd@.service                                static
```

產生 Teaming Interface Connection Profile 的方式跟 Bonding 不太一樣，主要是由 libtam 裡面的 Runner 來監控網路介面並依據所設定的演算法對此作出不同的行為。目前 Runner 有支援以下模式:

- broadcast
- roundrobin = bonding mode 0
- activebackup = bonding mode 1
- loadbalance = bonding mode 6 (最通用)
- lacp = bonding mode 4

詳細演算法可參閱 [Infrastructure Specification - jpirko/libteam][3] 和鳥哥網站 [第 3 堂課 - LACP 與 bonding/team 及 IPv6 簡易設定][6]

```bash
$ nmcli con add type team ifname team0 config '{"runner":{"name": "activebackup"}}'
Connection 'team-team0' (f7b8627e-3847-4383-9f66-7d9c0dba1e5e) successfully added.
```

將 eth2 及 eth3 加入 team-team0 裡面，並分別產生 Connection profile team0-port1 及 team0-port2

```bash
$ nmcli connection add type team-slave con-name team0-port1 ifname eth2 master team-team0
Connection 'team0-port1' (075e1011-8772-459b-85ee-3213b3f74cc5) successfully added.
$ nmcli connection add type team-slave con-name team0-port2 ifname eth3 master team-team0
Connection 'team0-port2' (2e9a8489-ce3f-4509-a305-4d250c77fa46) successfully added.
```

跟 bonding 啟動方式一樣，也是從 slave interface 開始啟動介面至 teaming interface

```bash
$ nmcli con mod team-team0 ipv4.addresses 192.168.122.101/24
$ nmcli con mod team-team0 ipv4.method manual
$ nmcli con up team0-port1
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/35)
$ nmcli con up team0-port2
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/36)
$ nmcli con up team-team0
Connection successfully activated (master waiting for slaves) (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/37)
```

若要觀察 teaming interface 的狀態，可使用 `teamdctl` 來進行觀測，請參閱 [man: Team Netlink Interface][4]

## Conclusion

本文介紹了如何透過 nmcli 指令產生出 connection profile，同時也點出了 bonding 及 teaming 在技術實作上是不同的，有興趣的人可以自行爬文研究 libteam，但筆者比較推薦把 [第 3 堂課 - LACP 與 bonding/team 及 IPv6 簡易設定][6] 這文章先閱讀完，再參閱其他的文件

## Reference
- [使用 NETWORKMANAGER 命令行工具 NMCLI - RedHat Customer Portal][1]
- [Bonding vs. Team features - jpirko/libteam][2]
- [Infrastructure Specification - jpirko/libteam][3]
- [man: Team Netlink Interface][4]
- [Using Network Teaming or Bonding to Configure Aggregated Network Links - YouTube][5]
- [第 3 堂課 - LACP 與 bonding/team 及 IPv6 簡易設定][6]

[1]: https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_networkmanager_command_line_tool_nmcli
[2]: https://github.com/jpirko/libteam/wiki/Bonding-vs.-Team-features
[3]: https://github.com/jpirko/libteam/wiki/Infrastructure-Specification#runners
[4]: https://www.linux.org/docs/man8/teamnl.html
[5]: https://www.youtube.com/watch?v=H4Vwqh7mULQ
[6]: http://dic.vbird.tw/linux_server/unit03.php