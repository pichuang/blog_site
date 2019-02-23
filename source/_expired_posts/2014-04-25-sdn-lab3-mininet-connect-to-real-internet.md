---
layout: post
title: 'SDN Lab3$ Mininet connect to real internet'
date: 2014-04-25 14:21
comments: true
categories: 
---
# 前言
  原先使用 Mininet 建立 topology 的時候, 都是鎖在自己的虛擬出來的網路環境, 預設是不能往 Internet 連線, 我們需要改一些設定來讓他可以連接到外面, 可以要到 dhcp 可以做 nat, 讓 mininet 融入真實網路當中.

# 環境
- Internet
  - DHCP/NAT
- Ubuntu 14.04 LTS x86_64 in VM 
  - NIC: eth0 (set bridge mode)
- Mininet with OpenvSwitch 2.0.1
  - c0 
  - s1
  - {h1, h2}
   
  
# 過程
- Source: [Mininet_connect_to_internet](https://gist.github.com/pichuang/11280233)
  - Line 18: 這 API 應該是跟 `ovs-vsctl add-port s1 eth0` 同樣效果 但是不知道為何沒有執行, 所以才會有 Line 30的產生

  若使用該 source 的話, 可透過 `h1 ping 8.8.8.8` 或 `h2 ping 8.8.8.8` 到外面, 但是從 internet 卻不能連線進 Ubuntu, 這時候還需要改些東西
  
> ifconfig s1 <ip/netmask>  
ifconfig eth0 0.0.0.0  
route add default gw <nat_ip> s1  
route del default gw <nat_ip> eth0  
  
  這邊的目的是把原先處理傳送封包的 eth0 改為讓 s1 處理封包, 所以相關的 route, ip 都得全部重設, 到這裡, 不論是 h1, h2 還是 Ubuntu 本身都可以全網路暢通.

