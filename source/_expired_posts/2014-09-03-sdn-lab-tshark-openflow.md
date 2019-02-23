---
layout: post
title: 'SDN Lab $ OpenFlow Enabled tshark'
date: 2014-09-03 11:22
comments: true
categories: 
---
Dump OpenFlow packetes in CLI enviroment

# 環境
- OS: Ubuntu 14.04.1 LTS

# 安裝過程
## 安裝 tshark
> aptitude install software-properties-common -y && aptitude update -y  
add-apt-repository ppa:whoopie79/trusty && aptitude update -y  
aptitude update -y  
aptitude install tshark  

## 確認 tshark 是否有支援 OpenFlow
> tshark -G protocols | grep -i openflow

# 使用
1. filter packet and write to file ""
> tshark -d tcp.port==6633,openflow -Y "tcp.port==6633" -w openflow_study.pcapng
	* -d Decoding
	* -Y display filter
  * -w write to a file

2. 
filter packet and read to file
> tshark -V -d tcp.port==6633,openflow -Y "tcp.port==6633" -r arp
	* -r read file



# Reference
- [Wireshark - OpenFlow (openflow)](http://wiki.wireshark.org/OpenFlow)
- [OpenDaylight OpenFlow Plugin:OF1.3 Enabled Wireshark](https://wiki.opendaylight.org/view/OpenDaylight_OpenFlow_Plugin:OF1.3_Enabled_Wireshark)
