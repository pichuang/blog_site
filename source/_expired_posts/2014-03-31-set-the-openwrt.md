---
layout: post
title: '設定 OpenWrt'
date: 2014-03-31 01:27
comments: true
categories: 
---
1. 設定 5 ports<->interface
	- [OpenWrt 5 interface](https://gist.github.com/pichuang/9876454)

2. /etc/config/network 裡有關於 config switch => option name 
	* ```swconfig list``` 可找到相對應的 switch name
	* ```swconfig dev switch0 help``` 可以看到該晶片可用的功能
	* ```swconfig dev switch0 show ``` 可以看到該晶片現在的設定
	* [OpenWrt swconfig](http://wiki.openwrt.org/doc/techref/swconfig)

3. Allow specific wan IP use SSH
	- [OpenWrt firewall setting](https://gist.github.com/pichuang/9876547)
  
4. Default gateway
  * ```route add default gw x.x.x.x```
  * ```route delete default gw x.x.x.x```
  