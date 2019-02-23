---
layout: post
title: '編譯 OpenvSwitch v2.3.1 on Ubuntu 14.04.1 LTS'
date: 2014-08-15 09:13
comments: true
categories: 
---
2014/12/3 推出 OpenvSwitch 2.3.1, 主要為 OpenvSwitch 2.3.0 fixed 一些 bugs 

2014/8/14 OpenvSwitch 發出了 OpenvSwitch 2.3.0 新版本, 這是繼 1.9.0 之後最新的 LTS (Long-term Support), 也是第一版有支援 multithreading 及 megaflows 的 LTS 版本, 如果有在開發及研究的可以往 2.3.0 發展會是比較恰當的選擇

* Linux kernel 支援到 3.14 
	* Ubuntu 14.04.1 可以用了 大家不用再回頭找 12.04 舊版本的 iso 來使用了
* 預設開啟 OpenFlow 1.1 1.2 1.3
* 針對 Linux Kernel datapath megaflows 部分有做改善
* 增加實驗性質的 DPDK 支援
	* 最近有個很出名的 vswitch project 就是用 Intel DPDK [Lagopus](http://lagopus.github.io/)
* 對於 [IPFIX (IP Flow Information Export)](http://en.wikipedia.org/wiki/IP_Flow_Information_Export) 新增了 SCTP 及 ICMPv4/v6 templates
* 當 ovs-vswitchd fails 時, ovs-vsctl 要 add-port 或 add-br 現在會回報錯誤
	* 以前如果 ovs-vswitchd 沒正確開啟的話, 使用 ovs-vsctl 是還可以add的
* 針對 Ryu 有特別寫了個 "check-ryu" 測試 ovs 與 Ryu 的功能, 喜愛 Ryu 的可以研究一下
* 支援 custom vlog pattern (Python)

安裝步驟大致上跟 [編譯 OpenvSwitch v2.1.2 on Ubuntu 12.04 LTS](http://roan.logdown.com/posts/165399-compile-openvswitch-on-ubuntu-1204-lts) 一樣 但為求謹慎還是在寫個一篇紀錄

# 安裝過程
## 環境
  - OS: Ubuntu 14.04.1 Server LTS X86_64 
  - Kernel version:  3.13.0-34-generic

## 預先安裝
> aptitude install dh-autoreconf libssl-dev openssl

## 編譯 OpenvSwitch
> wget http://openvswitch.org/releases/openvswitch-2.3.1.tar.gz  
tar zxvf openvswitch-2.3.1.tar.gz && cd openvswitch-2.3.1  
./boot.sh  
./configure --with-linux=/lib/modules/`uname -r`/build  
make -j && sudo make install  
sudo make modules_install  
sudo modprobe gre  
sudo modprobe openvswitch  
sudo modprobe libcrc32c  

  * 使用 ```lsmod |grep openvswitch``` 確認ovs正確地掛載起來
  * ./configure 部份其實還可以下一個 ```--prefix=``` 參數, 可以讓openvswitch 完全裝在該目錄底下而不會亂掉

## 設定 ovsdb
> ovsdb-tool create /usr/local/etc/openvswitch/conf.db /usr/local/share/openvswitch/vswitch.ovsschema
* 建立一次即可, 如果設定有問題的話可以把conf.db砍掉重建

## (二選一) 開啟 ovsdb-server (no ssl)
> ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \  
--remote=db:Open_vSwitch,Open_vSwitch,manager_options \  
--pidfile --detach --log-file

* ```cat /usr/local/var/log/openvswitch/ovsdb-server.log``` 可查到 ovsdb log 

## (二選一) 開啟 ovsdb-server (ssl)
> ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \  
--remote=db:Open_vSwitch,Open_vSwitch,manager_options \  
--private-key=db:Open_vSwitch,SSL,private_key \  
--certificate=db:Open_vSwitch,SSL,certificate \  
--bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \  
--pidfile --detach --log-file

  * 若要使用 ssl 連線, 務必在編譯前要先準備好 ```libssl-dev``` 及 ```openssl```, 否則會噴出 ```Private key specified but Open vSwitch was built without SSL support``` 的錯誤
  * [OpenvSwitch Lab 6$ TLS SSL](http://roan.logdown.com/posts/208707-openvswitch-lab-6-ssl)


## 開啟 ovs-vsctl 
> ovs-vsctl --no-wait init

## 開啟 ovs-switchd 功能
> ovs-vswitchd --pidfile --detach --log-file

  * ```cat /usr/local/var/log/openvswitch/ovs−vswitchd.log``` 可查到 ovs-vswitchd log

## 開機自動化
### 設定開機自動 load module 
> echo "openvswitch " >> /etc/modules  
echo "gre" >> /etc/modules  
echo "libcrc32c" >> /etc/modules  

### 設定開機自動啟動
1. vim /etc/init.d/openvswitch

> \#!/bin/sh
start-stop-daemon -q -S -x /usr/local/sbin/ovsdb-server -- --remote=punix:/usr/local/var/run/openvswitch/db.sock --remote=db:Open_vSwitch,Open_vSwitch,manager_options --pidfile --detach --log-file  
sleep 3 # waiting ovsdb-server  
start-stop-daemon -q -S -x /usr/local/bin/ovs-vsctl -- --no-wait init  
start-stop-daemon -q -S -x /usr/local/sbin/ovs-vswitchd -- --pidfile --detach --log-file  

2.
> chmod +x /etc/init.d/openvswitch

3.
> update-rc.d -f openvswitch defaults 

### 設定 interface
1.
> ovs-vsctl add-br ovs-br  

2.
vim /etc/network/interfaces

> \# The loopback network interface  
auto lo  
iface lo inet loopback  
\# The primary network interface  
auto eth0  
iface eth0 inet manual  
up ifconfig $IFACE 0.0.0.0 up  
down ifconfig $IFACE down  
\# OpenvSwitch Interface  
auto ovs-br  
iface ovs-br inet static  
address x.x.x.x  
netmask 255.255.255.0  
gateway o.o.o.o  
dns-nameservers 168.95.1.1  

### 修改 Failsafe
vim /etc/init/failsafe.conf

> $PLYMOUTH message --text="Waiting for network configuration..." || :  
sleep 1  
$PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :  
sleep 1  
$PLYMOUTH message --text="Booting system without full network configuration..." || :  
