---
layout: post
title: 'OpenvSwitch Lab 7$ Setting OpenWrt'
date: 2014-10-25 01:43:00 +0800
updated: 2014-10-25 01:43:00 +0800
categories: sdn
tags:
- sdn
- openvswitch
- openwrt
---
在看本篇之前, 我預設你已經把 OpenWrt 刷好, 也把 OpenvSwitch 安裝好了, 那從現在開始我們就要開始設定 OpenWrt 讓他變成一個 OpenFlow switch, 那如果沒刷過的也別難過, 可以參考 [編譯 OpenWrt](http://roan.logdown.com/posts/165911-compiled-openwrt) 來完成編譯, 那如果有需要 ```.config``` 的話, 可以參考我的 [config](https://gist.github.com/pichuang/8703dcae2ec113047bed)


## 環境
* DLink DIR-835
    * 5 LAN Port  
    * 2 WLAN Port  
* 這邊要稱讚一下他的 recovery mode 做得還不錯, 有 web 上傳介面, 節省很多時間

## 切出 5 個 interface
將原先 eth0.{1,2} 切成 5 個 interface eth0.{1,2,3,4,5}, 供 OpenvSwitch 針對 Port 做控制 
  
[network 設定檔](https://gist.github.com/CliffLin/a48a461acbd2ee391079) (contributor : nosignal,CliffLin)

裡面的 eth0.4 為什麼要特別設定呢? 在先舊的 config 我們都是透過 OpenWrt 替我們建好的 bridge 'lan' (192.168.1.1)來進行連線, 新的設定檔因為把 lan 拿掉, 所以要特別針對某個 interface (例如: eth0.4) 上一個 ip, 好方便連線設定

而這個 interface 之後也不會將它 ```ovs-vsctl add-port```, 理由是避免你設定錯誤導致整台 AP 進不去的防制手段, 可以視為 ```Console port``` 

由於wlan設定中, 我們把wifi network 接到名為 lan 的介面上，故我們需要在network設定檔中新增一個介面

然而我們希望在開一個vlan來處理wlan，因此我們開了一個eth0.6的vlan

* eth0.6 <->(bridge)<->lan<->wlan


## 開啟 WLAN
將 /etc/config/wireless 裡面的兩行```option disabled 0 ``` remove 之後, 再下 ```wifi``` 啟動 wireless

[wireless 設定檔](https://gist.github.com/pichuang/fbaa2d4b234da112e6c5)

## 設定 OpenvSwitch
* 新增 interface 至 ovs-br 做管理
> ovs-vsctl add-br ovs-br  
ovs-vsctl add-port ovs-br eth0.1  
ovs-vsctl add-port ovs-br eth0.2  
ovs-vsctl add-port ovs-br eth0.3  
ovs-vsctl add-port ovs-br eth0.5  
ovs-vsctl add-port ovs-br eth0.6  
    * eth0.4 不加進去的理由已經寫在上面
    * eth0.6 是wlan的port

* 檢查 service
> ovs-vsctl show  
ovs-ofctl show ovs-br  

    * 這邊要注意所有的 interface 都應該要被正確的顯示, 尤其是 ovs-ofctl 這裡面的資訊, 大家最常遇到的問題是 ```ovs-vsctl show``` 明明有相關資訊, 但是卻不會動, 有過半的可能是因為你的 ovs-vswitchd 沒有啟動, 你下 ```ovs-ofctl show ovs-br``` 會毫無反應


## 設定網路
設定網路是非常重要的事情 "網路不會通, 就不會有 SDN"

讓 ovs-br 連上網路 (Network: 192.168.77.0/24, GW: 192.168.77.1), 這邊採用 in-band 作法, out-of-band 作法之後再另外說

* 上 ip (例: 192.168.77.111)
> ifconfig ovs-br 192.168.77.111  
route add default gw 192.168.77.1 ovs-br  

* 檢查 route table
> route -n  

<img class="center" src="https://lh3.googleusercontent.com/-rL1RwjTaauE/VEqXVvGldGI/AAAAAAAAFh4/uD9-4v8EM0Q/w1916-h342-no/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-10-25%2B02.15.06.png" width="50%" height="50%">

* 檢查網路狀況  
> ping 192.168.77.1  
ping 8.8.8.8  

<img class="center" src="https://lh5.googleusercontent.com/-TaazWl4gJo8/VEqYDJHhfJI/AAAAAAAAFiM/9gX9HfZyhHQ/w1916-h1048-no/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-10-25%2B02.18.36.png" width="50%" height="50%">

* 設定 controller 連線
> ovs-vsctl set-controller ovs-br tcp:x.x.x.x:6633  

    * 應該要可以看到 ```is_connected: true```, 若無的話, 你上面一定有個步驟弄錯, 再仔細檢查看看

到這邊應該大家都可以順利地建立 OpenFlow switch, 那如果有問題的話, 歡迎加入 [SDNDS-TW](http://sdnds.tw/) 討論, 我們有很多志同道合的人都在這邊一起努力, 希望大家都能一起加入討論或研究
