layout: post
title: 2019 大改造宅宅雲架構系列文-3 宅宅雲網路架構設計
author: Phil Huang
tags:
  - linux
categories:
  - homecloud
date: 2019-11-16 02:03:00
---
2019 大改造宅宅雲系列文來到第 3 篇，有興趣想看前面兩篇的可以參考：
1. [2019 大改造宅宅雲架構系列文-1][1]：介紹大多數用錢包君威力扛回家的硬體設備和架構設計總體目標
2. [2019 大改造宅宅雲架構系列文-2][2]：介紹宅宅雲塔式伺服器硬體裝備

而第三篇要來承續第二篇，講一下關於下圖藍色區塊 vcenter.phil.lab 那塊虛擬網路我個人是怎麼設計的

![](/images/homecloud-1.png)

<!--more-->

## 1 台 2 埠雙臂網路架構

雙臂網路 (Two-Arm) 需使用 2 個網路介面，Inbound/Outbound 分別走不同介面；單臂網路則是 (One-Arm) 僅使用 1 個網路介面，Inbound/Outbound 都走同一個介面，裡面的網路隔離是採用 802.1Q VLAN Subinterface 進行劃分，所以後者相較前者，Overlay Switch (這邊指的就是 Synology RT2600ac ) 需要多支援 VLAN 能力才有辦法做，但可惜它沒有，就保持做 Two-Arm 網路。而我自己經驗基本都是做 Two-Arm，單純好理解

### 最小 Port 數?

關於服務器硬體規格可以看 [2019 大改造宅宅雲架構系列文-2: 硬體菜單][2]，主要核心硬體需求是

> 1 ~ 2 Port

對...只要兩個 Port 就什麼都可以做，無論速率、無論架構皆可，但最小 1 Port 也可以做，只是雙臂架構概念會有點不太一樣，看角度，下面會詳細畫圖講一下。

基本相同概念適用於實體機直上、VMware vSphere、Linux KVM、Red Hat Virtualization、Proxmox、Red Hat OpenStack 等等虛擬化平台都可以做。

Q: Hyper-V 呢?

Sorry，這個我沒研究，無法評論


### vFW? vRouter?

主要網路轉發核心功能需求是

> NAT + VLAN Subinterface

這邊採用的 vFirewall 是 [pfSense][3]，先前在 Edgecore Networks 機櫃環境是用 [OPNsense][4]，而我個人是比較偏好 OPNsense。

![](/images/pfsense.png)

Q: 說好的 vRouter 不能搞嗎?

先前也有嘗試用 [VyOS][https://vyos.io/] 當做虛擬路由器，但三年前測試的時候，它沒有支援 NAT 啊啊啊啊啊！就放棄回到 pfSense / OPNsesne 的懷抱中，不知道現在有沒有支援，徵求勇者試試

Q: 疑? 這樣 iptables 土炮好像也很 ok 啊?

無論是 Linux iptables / FreeBSD pf 系列的都可以做，技術上沒什麼問題，只是我懶得每次都在想 iptables 的 `masquerade` 跟 `Port Forwarding` 怎麼下

### IP Design?

想當爾，要提供 WAN IP 給對應的網路介面，這邊假設以下：

1. WAN IP: 192.168.99.250/24
2. VLAN 100 Subnet: 10.100.0.0/24
3. VLAN 200 Subnet: 10.200.0.0/24
4. Rescue IP: 192.168.1.1/24

目標上就是拿到 10.100.0.0/24 或 10.200.0.0/24 的 IP 都要能透過 WAN IP 走 SNAT 方式出去；反之，可以從 WAN IP 透過 DNAT 做 Port Forwarding 進 10.100.0.0/24 或 10.200.0.0/24  網段

Q: Rescure IP?? 這幹嘛的

關於 Rescue IP 的作用是救援用，假設我的 VLAN IP 或者是有什麼異常狀況不能透過正常設定好的網路進去修改，我會拔掉原先接在 LAN Port (eth1) 上的線，用自己筆電直連該 Port 進去修

### 1 . vFW 雙臂直上實體機架構圖

> 一台 Server 直接裝 pfSense

![](/images/pfsense-1.png)

這個架構缺點顯而易見，整台實體機資源都被 pfSense 咬走了，完全沒有用到任何虛擬化平台，但可以外接支援 VLAN Tagging/Untag 的交換器擴大可使用實體 Port 數

![](/images/pfsense-3.png)

Q: 我家沒有具備 VLAN 功能的 Switch...

如果家中有一台小小的電腦，剛好也有兩個 Port，就很適合做上圖， 而 eth1 不一定要用 VLAN Subinterface 處理，可以直接掛 IP 在上面使用，外面接 L2 Switch 起來用，擴大可使用實體 Port 數

### 2. vFW 雙臂 + 虛擬化平台 + 1 實體 Port 架構圖

> 一台 Server 裝虛擬化平台，起 VM 裝 pfSense，網段用 VLAN Subinterface 切割，不考慮外接其他設備

下圖是以 VMware vSphere 作為範例

![](/images/pfsense-2.png)

關於 Virtual Switch 的部分，無論是 vSS (virtual Standard Switch) /vDS (virtual Distributed Swtich) 皆可使用，詳細相關技術評估請洽友商 VMware。

Q:  `vDS: WAN` 端的設定示意圖?

![](/images/vds-1.png)

右邊那邊 Port Group 可以多加網卡做點事情，詳細功能請參考官方文件 [vSphere Distributed Switch Architecture][5]，左邊留意 `VLAN 識別碼` 為 `--`，但我不確定是不是代表是 `VLAN 1` 的意思，哪天遇到問題再研究

Q: `vDS: Trunk` 端的也來一下

![](/images/vds-2.png)

右邊留意 Port Group 是完全沒有實體網卡在裡面的，左邊則是針對 VLAN ID 設定個別的 `VM Network` (注：我忘記正確描述是什麼了，有錯請敲我更正)

Q: 多問一句...有魔鬼嗎?

![](/images/vds-3.png)

Of course！這個設定應該是有做過類似事情的朋友都會遇到的最大遺漏，那個被設定為 Trunk 的 VM Network... 要把混合模式 (Promiscuous Mode)開啟來，避免來自於各式各樣的網路的封包在 ESXi 層就被丟掉，具體技術文件請參照 [Promiscuous Mode Operation][6]

下圖是採用 libvirt 為主的技術架構

![](/images/pfsense-4.png)

Q: 疑? 怎麼看起來都差不多?

其實簡單來說就是透過 Bridge 技術搭橋把實體網卡和虛擬網卡塞在同一個 Linux Bridge ( or OVS Bridge) 裡面，就很像是多台虛擬 L2 交換機在虛擬化平台裡面，概念就是這麼簡單，因為 VMware vSS/vDS 設定方式比較特別，所以特別拉出來講

### 3. vFW 雙臂 + 虛擬化平台 + 2 實體 Port 架構圖

> 一台 Server 裝虛擬化平台，起 VM 裝 pfSense，網段用 VLAN Subinterface 切割，須考慮外接其他設備

![](/images/pfsense-5.png)

這個就是先前在 Edgecore Networks Lab 最完整的架構，詳細投影片請參照 2017 年有一天不知道在哪裡講的 [Build Testing Infrastructure][7] 投影片

Q:  為什麼要特別做雙臂?

![](/images/rack.png) 

因為完全是為了管理這整櫃機櫃裡面設備使用的，那時候有另外接兩台 Cumulus Linux + AS4610-54T 做 CLAG，讓底下設備只要可以咬 IP 的全部配發一個 IP，網段全部切開做 VLAN Routing 避免被廣播封包打爆

Q: VLAN Routing !!! 是不是漏講了什麼? 那個 pfSense 裡面怎麼做的?

第四篇，我覺得也蠻有趣的部份，傳送門 [2019 大改造宅宅雲架構系列文-4][9]


## Traffic 分解圖

### SNAT Traffic

![](/images/pfsense-6.png)

這個很直覺，VLAN Routing + SNAT 搞定

### DNAT Traffic

![](/images/pfsense-7.png)

這個也很直覺，DNAT 做 Port Forwarding，這邊提供一個小祕技，搭 `SSH ProxyCommand`蠻好用的，參照 [20190720 Better Practice  Day 2 Operaition with Ansible][8]

## 總結

其實這年頭看到虛擬網路技術不用感到特別害怕，畫點圖，上個 IP，想像一下網路怎麼流，其實很多問題就能解決了，平時可以多多做點 Lab 培養觀落陰能力

[1]: https://blog.pichuang.com.tw/20191114-homelcloud-high-level-design-1/
[2]: https://blog.pichuang.com.tw/20191115-homelcloud-high-level-design-2/
[3]: https://www.pfsense.org/
[4]: https://opnsense.org/
[5]: https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html
[6]: https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.security.doc/GUID-92F3AB1F-B4C5-4F25-A010-8820D7250350.html
[7]: https://speakerdeck.com/pichuang/build-testing-infrastructure?slide=17
[8]: https://speakerdeck.com/pichuang/20190720-better-practice-day-2-operaition-with-ansible?slide=12
[9]: https://blog.pichuang.com.tw/20191116-homelcloud-high-level-design-4/