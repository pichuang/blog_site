---
layout: post
title: 'OpenvSwitch Lab 3$ NetFlow & sFlow'
date: 2014-04-27 07:20
comments: true
categories: 
---
#前言
通常做網路實驗都會想要撈一些統計數據回家研究, OpenvSwitch 內有實作 sFlow 及 NetFlow, 可供網管分析流量, 本 Lab 採用 ntopng 來完成實驗

# 安裝
## ntopng 
1. 安裝套件
> aptitude install subversion libxml2-dev libsqlite3-dev libpcap-dev redis-server libglib2.0-dev

2. 下載
> svn co https://svn.ntop.org/svn/ntop/trunk/ntopng/  
cd ./ntopng  
./autogen.sh  
./configure  
make -j && make install  
ldconfig #For nProbe lib link problem  

3. Start
> /etc/init.d/redis-server start  
ntopng --local-networks "192.168.77.0/24" \  
--interface eth0 \  
--interface "tcp://127.0.0.1:55566" \  
--user nobody \  
--daemon  
  - interface 請填你要 monitor 的介面, 另外也支援 ZeroMQ , 可使用 tcp:127.0.0.1:55566 來建立一個 interface, 來做 Data collector
  - user 是使用 nobody 來執行 ntopng 較為安全
  - local-networks 是手動設定 local 網段為何, 顯示時會有 `remote` 和 `local` 的差別, 可設定多個, 之間用`,`隔開
  - 開啓 Browser 連 <server_ip>:3000

## 設定 nProbe
> nprobe --zmq "tcp://*:55566" -i none -n none --collector-port 2055
 - zmq 要跟 ntopng interface裡面設定的 port 要一致
 - collector-port 是要設定接收 switch 來的封包

## 增加 NetFlow 設定
- [Set netFlow](https://gist.github.com/pichuang/11331998)
  - COLLECTOR_PORT 要填 2055

## 增加 sFlow 設定
- [Set sFlow](https://gist.github.com/pichuang/11332074)
  - COLLECTOR_PORT 要填 2055

## 有圖
<img class="center" src="https://lh6.googleusercontent.com/-IgY1Qi_s9Js/U1w9ztN_sdI/AAAAAAAADTk/l_t_eH2Iees/w909-h380-no/ntopng.PNG" alt="ntopng">
  - 紅框為配合 [SDN Lab3$ Mininet connect to real internet](http://roan.logdown.com/posts/195601-sdn-lab3-mininet-connect-to-real-internet) 的 Lab 將其中一個 h1 的 netflow 導至遠端的 ntopng 所顯示的圖片
  
#後記
整體上來說 ntopng 可以視為一個 server 專門顯示資料, 而 nprobe 可視為專門收集資料後轉送給 ntopng 的一支程式, 然而 switch 的資料都會先送到 ntopng 上處理, 而 ntopng 是透過 [ZerpMQ](http://zeromq.org/) 來接收資料


# Reference
 - [sflow](http://blog.sflow.com/)
 - [Monitoring VM traffic using sFlow](http://openvswitch.org/support/config-cookbooks/sflow/)
 - [ntopng User’s Guides](https://svn.ntop.org/svn/ntop/trunk/ntopng/doc/UserGuide.pdf)
 - [Pica8 Open vSwitch Configuration Guide](http://www.pica8.com/document/pica8-OVS-MPLS-configuration-guide.pdf)
 - [Why nProbe+JSON+ZMQ instead of native sFlow/NetFlow support in ntopng?](http://www.ntop.org/nprobe/why-nprobejsonzmq-instead-of-native-sflownetflow-support-in-ntopng/)
 - [For ntopng Developers](https://svn.ntop.org/svn/ntop/trunk/ntopng/README.ntopng)
