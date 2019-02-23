---
layout: post
title: 'SDN Lab 4$ Upgrade OpenvSwitch in Mininet'
date: 2014-09-03 10:12
comments: true
categories: 
---
mininet 要如何安裝? 這部分 mininet 有給很[詳細介紹](http://mininet.org/download/) 裡面提供的 OpenvSwitch 版本算是非常久遠的, 現在 OpenvSwitch 已經來到 2.5 了, 順應潮流我們也來把 mininet 上的 OpenvSwitch 升級一下

# 安裝過程

## 下載 mininet
> git clone git://github.com/mininet/mininet && cd mininet

## 安裝 mininet core module 並指定 OpenvSwitch 版本 2.5.0
> mininet/util/install.sh -nV 2.5.0  

## (Optional) Disable IPv6
* vim /etc/sysctl.d/90-disable-ipv6.conf  
> net.ipv6.conf.all.disable_ipv6 = 1  
net.ipv6.conf.default.disable_ipv6 = 1  
net.ipv6.conf.lo.disable_ipv6 = 1  
  - 避免 mininet hosts 發送 ipv6 封包產生 packet_in

* sysctl -p /etc/sysctl.d/90-disable-ipv6.conf

#Reference
- [Installing new version of Open vSwitch](https://github.com/mininet/mininet/wiki/Installing-new-version-of-Open-vSwitch)
- [Download/Get Started With Mininet](http://mininet.org/download/)
- [MiniNet as an SDN test platform](http://www.routereflector.com/2013/11/mininet-as-an-sdn-test-platform/)
- Thanks to @tutul
