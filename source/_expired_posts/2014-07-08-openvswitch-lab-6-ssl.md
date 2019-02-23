---
layout: post
title: 'OpenvSwitch Lab 6$ TLS SSL'
date: 2014-07-08 18:25
comments: true
categories: 
---
依先前寫的一篇 [編譯 OpenvSwitch v2.1.2 on Ubuntu 12.04 LTS](http://roan.logdown.com/posts/165399-compile-openvswitch-on-ubuntu-1204-lts) OpenvSwitch 與 OpenFlow controller 溝通, 並無任何的安全性, OpenFlow control meeage 等於裸奔在網路上, 十分不安全, OpenvSwitch 內建有 ```ovs-pki``` 可以產生相對應的 ssl key 來加以保護傳輸行為

# OpenvSwitch 相關
* 產生 key 
```
ovs-pki init --force
cd /usr/local/etc/openvswitch
ovs-pki req+sign roan-controller-ssl controller
ovs-pki req+sign roan-switch-ssl switch
```
  * 如有開 logfile 的話, 可以查詢  ```/usr/local/var/log/openvswitch/ovs-pki.log```
  * 你應該要把以下三個檔案傳到 OpenFlow controller server 上
 		1. `/usr/local/etc/openvswitch/roan-controller-ssl-cert.pem`
 		2. `/usr/local/etc/openvswitch/roan-controller-ssl-privkey.pem`
		3. `/usr/local/var/lib/openvswitch/pki/switchca/cacert.pem`
  
 
* OpenvSwitch setting TLS
```
ovs-vsctl -- --bootstrap set-ssl /usr/local/etc/openvswitch/roan-switch-ssl-privkey.pem \
									/usr/local/etc/openvswitch/roan-switch-ssl-req.pem \
                  /usr/local/var/lib/openvswitch/pki/controllerca/cacert.pem                  
```
 * ```ovs-vsctl get-ssl``` 可查詢狀況

* ovs-vsctl 設定
```
ovs-vsctl add-br ovs-br
ovs-vsctl set-controller ovs-br ssl:x.x.x.x:6633
```

* Screenshot
<img class="center" src="https://lh5.googleusercontent.com/-ooRqHtzvjjY/U7wh-F4-nZI/AAAAAAAAE6Y/U0C7lsRy7ak/w1764-h696-no/%25E8%259E%25A2%25E5%25B9%2595%25E5%25BF%25AB%25E7%2585%25A7+2014-07-09+0.52.47.png">

# OpenFlow Controller
## Pox
* Try Failed
* 依照 [Re: [pox-dev] Does pox supports SSL?](https://www.mail-archive.com/pox-dev@lists.noxrepo.org/msg01278.html) 

## Floodlight
* Not Support

## Ryu
* Try Failed
* 依照 [Setup TLS Connection](http://ryu.readthedocs.org/en/latest/tls.html) 產生 ```EOF occurred in violation of protocol``` 錯誤

## Trema
* Not Support

# Q&A
Q: 在 ovs-vswitchd.log 發現到 ```Private key specified but Open vSwitch was built without SSL support``` 該如何處理
A: 少裝 libssl 或 openssl 相關套件

#Reference
- [Configuring Open vSwitch for SSL](http://git.openvswitch.org/cgi-bin/gitweb.cgi?p=openvswitch;a=blob_plain;f=INSTALL.SSL;hb=HEAD)
- [Open vSwitch with SSL and Mininet](http://techandtrains.com/2014/04/27/open-vswitch-with-ssl-and-mininet/)
