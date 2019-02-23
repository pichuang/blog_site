---
layout: post
title: 'Ubuntu 12.04 LTS bonding (LACP)'
date: 2014-01-26 06:38
comments: true
categories: 
---
#前言
因系上電腦教室(100多台)要利用 Clonezilla 做還原, 每個image動輒好幾G, 原先使用的1G頻寬有點不太夠, 故想要利用 Link Aggregation Control Protocol (LACP) 的方式來加大頻寬, 利用兩張網卡 Bind 一個IP, 架構不變, 流量變大, 再正式使用前先來做點實驗.

#環境
1. ovs OS: Ubuntu 12.04 LTS x86_64 
2. NIC: 
 * Motherboard wired port (eth0)
 * D-Link 1 port PCI NIC  (eth1)
 * D-Link 1 port PCI NIC  (eth2)
 * D-Link 1 port PCI NIC  (eth3)

#安裝
##安裝套件
* ``` aptitude install ifenslave-2.6 ```

##設定 /etc/network/interface

> auto lo  
iface lo inet loopback  
auto eth0  
iface eth0 inet manual  
bond-master bond0  
auto eth1  
iface eth1 inet manual  
bond-master bond0  
auto eth2  
iface eth2 inet manual  
bond-master bond0  
auto bond0  
iface bond0 inet manual  
address <your ip>  
netmask <your netmask>  
gateway <your gateway ip>  
bond-slaves none  
bond-mode 6  
bond-miimon 100  

* 這邊要注意一下 bond-mode 有六種, 推薦用 balance-alb 6
* 這樣子即可順利把 eth0 eth1 eth2 綁在 bond0 上

## 觀察bond狀態

* ```cat /proc/net/bonding/bond0```

# 檢查網卡支不支援 bonding 

## 安裝套件
* ```aptitude install net-tools```

## 觀察網卡狀態
* ```mii-tool```	
* 如果顯示 
> eth0: negotiated 1000baseT-HD flow-control, link ok  
eth1: negotiated 1000baseT-HD flow-control, link ok  
eth2: negotiated 1000baseT-HD flow-control, link ok  

* 反之即不支援, 有可能會再舊款網卡或速率不一的網卡發生

#推薦閱讀
* [Ubuntu bonding 小技巧](http://www.geego.com.tw/technical-discussion-forum/tech-tips-the-tips-of-ubuntu-bonding-ubuntu-%E5%B0%8F%E6%8A%80%E5%B7%A7)
* [鳥哥 server bonding](http://linux.vbird.org/linux_enterprise/0110network.php#server_bonding)
