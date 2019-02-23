---
layout: post
title: 'OpenvSwitch Lab 1$ OpenvSwitch Setup'
date: 2014-01-26 05:35
comments: true
categories: 
---
#前言
先前用 OpenvSwitch fot OpenWrt 實作一些網路設定, 但是問題是如果有個網路設定不小心設錯, 可能導致ssh進不去該AP, 所以退而求其之, 實裝一台 x86_64 pc 加上多 Port 網卡來實作, 避免浪費時間一直重刷 AP.

#環境
1. ovs OS: Ubuntu 12.04 LTS x86_64 
2. NIC: 
 * Motherboard wired port (eth0)
 * D-Link 4 port PCI NIC  (eth1~4)
3. Controller: [Floodlight](http://www.projectfloodlight.org/floodlight/)

 
# 架構圖
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/177333/VO93J8oKSH68gLiTd7jW_ovslab1.png" alt="ovslab1.png">

#步驟
## 安裝 OpenvSwitch
* 參考 [編譯 OpenvSwitch v2.0.0 on Ubuntu 12.04 LTS](http://roan.logdown.com/posts/165399-compile-openvswitch-on-ubuntu-1204-lts)

## 設定 ovs-vsctl
> ovs-vsctl add-br ovs-br  
ovs-vsctl add-port ovs-br eth0  
ovs-vsctl add-port ovs-br eth1  
ovs-vsctl add-port ovs-br eth2  
ovs-vsctl add-port ovs-br eth3  
ovs-vsctl add-port ovs-br eth4  
ovs-vsctl set-controller ovs-br tcp:172.10.10.3:6633  

  * 172.10.10.3 盡量不要用domain name, 會連不到.

## 設定 ifconfig 
> ifconfig eth0 up  
ifconfig eth1 up  
ifconfig eth2 up  
ifconfig eth3 up  
ifconfig eth4 up  
ifconfig eth0 0.0.0.0  

 * ```ifconfig eth0 0.0.0.0```目的不明, 但不設定的話封包出不去

## 設定 ovs-br
> ifconfig ovs-br 172.10.10.1/24  
route add default gw 172.10.10.254 dev ovs-br  

 * 這邊要注意如果```route -n```有其他default route rule, 要記得清除.
 * 這邊動作是要給 ovs-br 一個ip, 讓他可以連接到 controller, 如果不上ip的話, 這台機器就會變成很單純的L2 Switch, ping也ping不出去, 所以給他上一個ip是較好的.
 
# 測試 (以Host-A角度來看)
> ping 172.10.10.101 #ping Host-B  
ping 172.10.10.254 #ping gateway  
ping 172.10.10.3 #ping Floodlight  

  * 基本上應該都要能通, 如果不通的話建議檢查一下```route -n```看一下有沒有問題

> http://172.10.10.3:8080/ui/index.html

 * 這是 Floodlight 提供的 Web GUI, 如果成功的話可以從上面看到該OVS的資訊.

# 結語
基本上如果 NAT Server那邊有對外連線的話, 設個dns就可以連出去了, 這架構已經經過確認沒有問題, 希望對後人有幫助. 中間可能會遇到非常多網路不通的問題, 一定會有各種意外, 記得有些好工具可以幫助你ping, mtr, route, ifconfig...可以解決你很多問題.

