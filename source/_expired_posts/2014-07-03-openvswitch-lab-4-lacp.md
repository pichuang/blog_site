---
layout: post
title: 'OpenvSwitch Lab 4$ LACP'
date: 2014-07-03 12:03
comments: true
categories: 
---
# 前言
有陣子以前實做了 [Ubuntu 12.04 LTS bonding (LACP)](http://logdown.com/account/posts/177335-ubuntu-1204-lts-bonding-lacp/edit) 利用了modprobe 的方式加入了功能, 現在要利用 OpenvSwitch 來達到這方面的實做, 環境依然跟前述一樣, 並會測試效能及比較.

#環境
1. ovs OS: Ubuntu 12.04 LTS x86_64 
2. NIC: 
 * Motherboard wired port (eth0)
 * D-Link 1 port PCI NIC  (eth1)
 * D-Link 1 port PCI NIC  (eth2)
 * D-Link 1 port PCI NIC  (eth3)
 
#過程
## 安裝 OpenvSwitch
 - [編譯 OpenvSwitch v2.1.0 on Ubuntu 12.04 LTS](http://roan.logdown.com/posts/165399-compile-openvswitch-on-ubuntu-1204-lts)

## 設定 Interface
> ovs-vsctl add-br ovs-br  
ovs-vsctl add-bond ovs-br ovs-bond eth0 eth1 eth2 eth3 lacp=active  

## 觀察
1. Show NIC status 
  - ```ovs-appctl bond/show ovs-bond```
2. Show LACP information 
  - ```ovs-appctl lacp/show ovs-bond```
3. Show detail configuration
  - ```ovs-vsctl list port ovs-bond```
