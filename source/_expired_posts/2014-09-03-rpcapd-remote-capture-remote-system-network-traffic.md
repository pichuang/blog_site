---
layout: post
title: 'rpcapd: Capture Remote System Network Traffic'
date: 2014-09-03 15:08
comments: true
categories: 
---
若要將某台 Linux Server 某個 interface 上的封包倒至 Wireshark GUI 裏 (參考下圖), 則需要利用 rpcapd 這支程式來幫助我們達成這件事情, 其實 rpcapd 是被包在 WinPcap 裡面

<img class="center" src="https://lh6.googleusercontent.com/5usweKZKqJ78CdeeeRRhss3EzQIeYyjiaTEeFSRXpkw=w1390-h508-no"  width="50%" height="50%">

做這件事的好處有不少
1. 即時 Capture remote interface
2. 有完善的 Wireshark GUI 可以看
3. Linux server 不用裝 X-Windows
4. 不用傳 .pcap 做封包研究

## Linux Server 部分
- OS: Ubuntu x86_64 14.04.1 LTS

### 下載 rpcapd-linux
> apt-get build-dep libpcap -y
git clone https://github.com/frgtn/rpcapd-linux

### 安裝 rpcapd
> cd ./rpcapd-linux/libpcap
./configure && make
cd ../ && make

### 啟動 rpcapd
> sudo ./rpcapd -4 -n -p 8888

* -4 ipv4
* -n NULL authentication
* -p port numbers (example: 8888)
* [more parameter](https://www.winpcap.org/docs/docs_40_2/html/group__remote.html)

## Wireshark GUI setting
- Version: Development Release (1.12.0rc3)
	- 主要是為了 decoding OpenFlow

### 設定 interface 
1. 選擇 Capture > Interfaces > Options > Manage Interfaces > Remote Interfaces > Add > 填入ip and port> OK > Close
<img class="center" src="https://lh5.googleusercontent.com/-pUp9pbSYObQ/VAbhK4vG1MI/AAAAAAAAFUg/VaGuXYkqck4/w1598-h883-no/remote-capture-wireshark-1.PNG" width="75%" height="75%">
<img class="center" src="https://lh5.googleusercontent.com/-wz3fMm5mQwM/VAbhNxpAT4I/AAAAAAAAFUo/zXt674wUKYk/w716-h338-no/remote-capture-wireshark-2.PNG" width="75%" height="75%">

2. 選擇 montior interface > start
<img class="center" src="https://lh5.googleusercontent.com/-2Za6ELqPe3A/VAbhwMZg4tI/AAAAAAAAFVA/q-WdfPjguKE/w1317-h623-no/remote-capture-wireshark-3.PNG" width="75%" height="75%">

3. 結果
<img class="center" src="https://lh6.googleusercontent.com/-ObOaOhJMxT4/VAbiG0wMXfI/AAAAAAAAFVU/eYjOu6y5w30/w1353-h823-no/remote-capture-wireshark-4.PNG" width="75%" height="75%">

### (Optional) OpenFlow decode
* Edit > Preferences > Protocols > openflow > OpenFlow TCP port 6633
<img class="center" src="https://lh6.googleusercontent.com/cNtPZP9jdrtdSU3gcXelLYuB0RhM8MWsSiRC_TNQvIk=w1124-h387-no" width="75%" height="75%">
	* 為什麼預設是 6653 ? 這是因為其實在 2013-07-18 後 [IANA 規定 OpenFlow protocol 是 6653](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=openflow), 而不是 6633
  

# Reference
- [WinPcap - Remote Capture](https://www.winpcap.org/docs/docs_40_2/html/group__remote.html)
- [frgtn/rpcapd-linux](https://github.com/frgtn/rpcapd-linux)
- [Wireshark - The “Remote Capture Interfaces” dialog box](https://www.wireshark.org/docs/wsug_html_chunked/ChCapInterfaceRemoteSection.html)

