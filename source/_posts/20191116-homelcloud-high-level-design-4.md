layout: post
title: 2019 大改造宅宅雲架構系列文-4 pfSense
author: Phil Huang
tags:
  - linux
  - infra
  - homecloud
categories:
  - homecloud
date: 2019-11-16 16:11:00
toc: true
---
先前工作太忙、失戀打擊過大，以致懷疑人生，導致有陣子沒寫文章，最近 Red Hat 贊助小弟來魚尾獅王國上課，趁空擋把長久以來用的一些技術經驗寫下來分享給大家，[2019 大改造宅宅雲架構系列文 快速傳送門][1]，歡迎大家把自己 HomeLab/HomeCloud 的經驗分享出來蕉流蕉流

![](/images/pfsense-logo.png)

如 Logo 所示，本篇將會承襲上篇 [2019 大改造宅宅雲架構系列文-3][2] 的結尾，到底 pfSense 放到 VM 裡面後是如何搞得? 

<!--more-->

## 服務太多? 切網卡! 切網段!

![](/images/pfsense-8.png) 

上篇有人如果眼睛很尖的話，應該會發現我紅框那邊有好幾個網路介面，個別 VLAN 都會定義一個服務群，把相同性質的服務全部塞在一起，避免互相干擾

Q: 是不是一定要在 VM 外插多隻腳出來才能做?

這個如同 [2019 大改造宅宅雲架構系列文-3][2] 的網路架構一樣，實際上我只有插兩張網卡在裡面，其他多張虛擬網卡都用 802.1q VLAN Subinterface 切出來

![](/images/pfsense-9.png)

上圖可以在 `Interfaces > Interface Assignments > VLANs` 裡面找到，所有透過 802.1q VLAN Subinterface 都會顯示在這邊

![](/images/pfsense-10.png)

以下我新增一個 `VLAN 104` 當作範例，記得要選正確的 Interface，這邊有兩張介面

1. vmx0: WAN
2. vmx1: LAN <-- 選他

因為在 pfSense 安裝時會需要定義 WAN / LAN，所以這個理論上要知道，記得不要選錯，不然切出來的網路介面不會動，有一次眼殘選錯，看了足足一天才發現

Q: 這樣就搞定了嗎!?

![](/images/pfsense-11.png)

還要回到 `Interfaces > Interface Assignments` 正式把這張虛擬網路介面卡加進去才算數

![](/images/pfsense-12.png)

把虛擬網路介面的設定填一填就搞定了第一階段，所以架構圖上就會像下圖一樣

![](/images/pfsense-2.png)

## 還有 NAT 呢...

在 pfSense 裡面關於 NAT 的部分，其實有[不少功能介紹][4]，這邊體恤小弟是個懶人，主要會用到

1. SNAT 懶人功能: [Automatic NAT Rules Generation][6]
2. SNAT 正常功能: [Advanced Outbound NAT][7]
3. DNAT 正常功能: [Forwarding Ports with pfSense][5]

![](/images/pfsense-14.png)

相關設定都在 `Firewall > NAT > Outbound` 裡面找得到，關於 `Outbound NAT Mode` 毫無懸念地開 `Hybrid Outbound NAT rule generation`，主要是因為 `Automatic NAT Rules Generation` 真的是一個`好棒棒`的功能 (我被我老闆影響了...) ，當你把上一個章節的虛擬網路加進去後，他會自動加一條 SNAT Rules 進去，啥都不用設定

關於 DNAT  Port Forward 的使用就是直接開 `Firewall > NAT > Port Forward` 裡面上 Rule 就完工了，詳細設定介紹 [Forwarding Ports with pfSense][5]

## 我知道你忘記了 Firewall 這件事

Q: 為什麼照文章設定完之後，還是不會動!!!???

![](/images/pfsense-15.png)

魔鬼就在這張圖裡面，可以在 `Firewall > Rules > <指定的網路介面>`找到她，關於這個問題搞了半天是防火牆預設 Firewall Deny All...意思上圖的 Rules 是表示全部不給過

> All incoming connections on this interface will be BLOCKED until PASS RULES are ADDED

![](/images/pfsense-16.png)

所以你需要多一條 Rule

- Family: IPv4
- Protocol: any
- Source: any
- S Port: any
- Destination: any
- D Port: any

基本上就會通了，其他 Rules 就是按照自己需求自己加就好

## DHCP 相關

因為我個人 Lab 都不用 DHCP 都是手動配 IP，但最近 Red Hat OpenShift 4 安裝忽然要弄 DHCP，剛好 pfSense 也有支援 [`DHCP Static Mappings for this Interface`][8]，所以做起來非常方便使用

## 還有更多嗎?

除了以上基本的功能，如果是以 pfSense 為核心的架構，你也可以把以下服務放在上面都沒什麼問題

- Time Synchronization: ntp
- DNS: bind
- VPN: OpenVPN

## 總結

雖然用 pfSense 自架自己用沒什麼太大問題，但如果是給小公司 (~10) 使用或者是 SMB，訂閱他們提供[軟體支援][9] 或產品會是一個比較完善的保護，倘若如果是大型企業的話，還是建議乖乖看商用方案比較好

最後我想說，企業支持開源軟體的具體實踐，用原始碼或錢實際支持這些宅宅，幫助這些軟體永續經營，宅宅們也是需要吃飯喝水養家的

[1]: https://blog.pichuang.com.tw/categories/infra/
[2]: https://blog.pichuang.com.tw/20191116-homelcloud-high-level-design-3/
[3]: https://www.facebook.com/groups/pve.tw/
[4]: https://docs.netgate.com/pfsense/en/latest/nat/index.html
[5]: https://docs.netgate.com/pfsense/en/latest/nat/forwarding-ports-with-pfsense.html
[6]: https://docs.netgate.com/pfsense/en/latest/nat/automatic-nat-rules-generation.html
[7]: https://docs.netgate.com/pfsense/en/latest/nat/advanced-outbound-nat.html
[8]: https://docs.netgate.com/pfsense/en/latest/dhcp/dhcp-server.html
[9]: https://store.netgate.com/Global-Support-C320.aspx