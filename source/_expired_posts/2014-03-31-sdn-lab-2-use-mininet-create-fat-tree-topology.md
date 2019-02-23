---
layout: post
title: 'SDN Lab 2$ Use Mininet create Fat Tree Topology'
date: 2014-03-31 00:15
comments: true
categories: 
---
# 前言
這是份作業, 目的是利用 Mininet 產生出一個 Fat Tree Topology, 並且針對其中幾對連線做 bandwidth 及 packet loss rate 的設定, 最後還要利用 iperf 打出數據出來
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/191753/T0ivt4hXQVulJFxCTdVD_fat%20tree%20topology.PNG" alt="fat tree topology.PNG">

# 過程
## [Source Code](https://gist.github.com/pichuang/9875468)
1. 建立 Fat Tree Topology
	- line 16 ~ 81
	- 先分別建立 Core Layer, Aggregation Layer, Edge Layer 及 Host 的 node 
	- 使用 `addSwitch` 產生 node, 並且利用 python 內建的 `append` , 將產生出的 nodes 放進去到 array 裡
  - 如果看不懂的話 [mininet/topo-2sw-2host.py](https://github.com/mininet/mininet/blob/master/custom/topo-2sw-2host.py) 這個是 sample code 可以參考參考
  
2. 產生出 Link
	- line 83 ~ 106
	- 可以分為 Core-Aggregation, Aggregation-Edge, Edge-Hosts 三部分來建立
	- 建立的方式很簡單, 透過 `addLink(a.switch, b.switch)` 就建立完成, 如果還有要另外設定 bandwidth 或者是 packet loss rate 可以另外在設定

3. 開啟 ovs STP (Spanning-Tree Protocol)
	- line 108 ~ 123
	- 使用 `os.system` 執行 `ovs-vsctl set Bridge switch stp_enable=true` 開啟相關功能
	- Floodlight 內建有專門處理 STP 的 Module, 可以不用開

4. iperf 測試
	- line 125 ~ 137
	- 這邊是分別測試兩個數據
		- 同 Pod 互連 依照作業需求是不會掉封包的
    - 不同 Pod 互連 會有 5% packet loss rate
    
## 截圖
- 這是產生 Topology 和 PingAll 的畫面
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/191753/smWG5mTsQSWsJHi73nHk_Create%20Fat%20Tree%20Topology.png" alt="Create Fat Tree Topology.png">

    
# Reference
- [Introduction to Mininet](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet#working)
- [Mininet Python API Reference Manual](http://mininet.org/api/classmininet_1_1link_1_1TCIntf.html)
- [iperf – 統計 jitter 以及 packet loss](http://benjr.tw/3030)


 

 