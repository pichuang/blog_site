---
layout: post
title: 'OpenvSwitch Debug$ Environment'
date: 2014-10-25 04:28:00 +0800
updated: 2014-10-25 04:28:00 +0800
categories: sdn
tags:
- sdn
- openvswitch

---
OpenvSwitch debug 要分三個階段看

1. OpenvSwitch Environment
2. Trace Flow
3. Controller App
  
大多數說網路不通的人在第一個階段就有問題了, 基本上只要知道 OpenvSwitch 架構和 Flow match 的行為, 就能(較)簡單的進行 Debug, 這篇從第一個階段開始講

# OpenvSwitch 架構
<img class="center" src="https://lh5.googleusercontent.com/-bVDBhgNuV0c/VEqhfco9aTI/AAAAAAAAFik/ksDKXPmaDdk/w1640-h1230-no/openvswitch_overview.001.png" width="50%" height="50%">

透過上圖可以清楚知道 OpenvSwitch 主要分為三個 Componets: kmod_openvswitch, ovs-vswitchd, ovsdb-server

* kmod_openvswitch 就是我們平時講的 datapath, 屬於在 kernel space 的層級, Packets 實際上轉發都是要透過這邊進行處理, 在這邊會有一個屬於這台 hosts 通用的 table, 可以透過 ```ovs-dpctl show``` 做觀察
* ovs-vsswitchd 是整個 OpenvSwitch 的核心所在, 屬於在 user space 的層級, 可以實作不少 protocols 在上面, 而 OpenFlow 實作也是在此, 這邊會針對 ```不同的 ovs bridge 會分別有不同的 Flow table```, 可以透過 ```ovs-ofctl show ovs-br``` 做觀察
* ovsdb-server 是儲存 OpenvSwitch 相關設定的地方, 基本上操作 ```ovs-vsctl``` 都是在與 ovsdb-server 溝通, 這點非常重要

詳細一點可以參考小弟於 SDNDS-TW Meetup #2 分享的投影片

<iframe src="//www.slideshare.net/slideshow/embed_code/40428437" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> </div>

亦或是 rajdeep , Director, Developer Relations VMware India at VMware India 於 slideshare 發表的投影片

<iframe src="//www.slideshare.net/slideshow/embed_code/27768124" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> </div>


# OpenvSwitch 環境檢查
## Check daemon

> ovs-ctl status
ps aux | grep ovs

* 確定有無 ovsdb-server 和 ovs-vswitchd
* ovs-ctl 打哪來的? ```/usr/share/openvswitch/scripts/ovs-ctl``` 這個在安裝的時候並沒有被安裝在 /usr/bin 底下, 如果要用的話可以自行 link 至目錄底下

## Logging

> ovsdb-tool show-log [-mmm]

* 這個是專門去撈你 ovs-vsctl 操作的紀錄, 多下-m可以看到更詳細的行為, 這邊可以驗證你的 db 操作是有沒有問題

> cat /var/log/openvswitch/{ovsdb-server,ovs-vswitchd}.log

* 這邊要在啟動的時候後面要加 ```--log-file``` 才會有的資訊, 詳細可以看[安裝流程](http://roan.logdown.com/posts/220671-compile-openvswitch-v230-on-ubutnu-14041-lts)
* 透過裡面資訊可以用時間來判斷各個 componets 發生了什麼事
* log 資訊詳細與否需配合 ```ovs-appctl vlog/list``` 做設定

## Interface 配置

> ovs-vsctl show 

這邊主要處理是由 ovsdb-server 負責

<img class="center" src="https://lh6.googleusercontent.com/UZ_Nie_Pdj0qa2i88YcHDvrEUXVpp0PWWL-oU2jirKE=w1558-h1230-no" width="50%" height="50%">

* Controller
	* is_connected: true 看到這個出現代表 ovs bridge 已經正常的連在 Controller 上面
	* port number 是 [6633 還是 6653](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=openflow) 
	* connected method 是 tcp? ssl? ...?
	* OpenvSwitch 連到 controller 的方式一開始還是走 L2 Switching 出去的 (Hidden flow), 意思是在佈建 OpenFlow 環境時, Legacy network 要先搞定, 不然就免談
* fail_mode
	* secure 一定要連上 controller 才處理封包
	* standalone 若沒有連上 controller 則自動轉換為 L2 Switching, 直到連上 Controller
	* Debug legacy network 是否有通, 則會開 standalone 來先進行檢查
	* Trace Flow 時會開 secure 來進行檢查
* Port
	* 只是驗證 conf.db 是否確切存有 interface 的資訊在裡面, 以便重啟服務時還可以回復至現在環境

> ovs-ofctl show ovs-br

這邊主要處理是由 ovs-vswitchd 負責

<img class="center" src="https://lh5.googleusercontent.com/-Tca6P8QOQm0/VEqviHZwBkI/AAAAAAAAFi8/byX-ycJKaB0/w1862-h1230-no/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-10-25%2B03.57.37.png" width="50%" height="50%">

* OFPT_FEATURES_REPLY
	* xid 裡面的值代表你現在支援哪個版本的 Protocols (例如: 0x2 = OpenFlow1.1)
	* OpenvSwitch 從 2.3.0 後, 預設採用 OpenFlow 1.1+, 如果要使用 OpenFlow 1.0 請使用 ```ovs-vsctl set bridge ovs-br protocols=OpenFlow10```
* dpid
	* ovs bridge id, Openflow controller 透過不同 dpid 來分別辨識不同 ovs bridge
* 1(wlan0)
	* 1 是代表 ofport, 你進行 ACTIONS 指定一些如 ```in_port=1,actions=output:2``` 裡面所述的數字, 都是從這邊對應的, Controller 能控制的 ofport 也是在 ovs-vswitchd 處理
	* ofport 的數字可以更換, [請搜尋 ofport_request](http://roan.logdown.com/posts/191801-set-openvswitch)
	* wlan0 是 ofport 所綁定的 interface, 無論哪個 interface 至多只能被歸納至某一個 ovs bridge 底下, 而不能同時加入多個 ovs bridge
  
> ovs-dpctl show

這邊主要處理是由 datapath 負責

<img class="center" src="https://lh3.googleusercontent.com/-uAZ00gDgDto/VEqx029n1zI/AAAAAAAAFjQ/9mUAfG4NP5A/w1824-h792-no/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-10-25%2B04.08.33.png" width="50%" height="50%">

* lookups
	* hit 表示 packet 進來後, match datapath flow table 成功次數
	* missed 表示 packet 進來後, 有 unmatched datapath flow table 情形, 則會透過 upcall 將封包送至 ovs-vswitchd 的次數
	* lost 表示在 ovs-vswitchd 回應前, datapath 就把封包 drop 掉的次數
* flows
	* 現存在 datapath flow 的個數
* port 1: wlan0
	* port 1 這邊是專門給 datapth 辨識的 port number, 跟上述 ```ovs-ofctl show ovs-br``` 所看到的 ofport 無相關, 但後續在 datapath 中 trace flow 時會需要他


各位觀察一下 ```ovs-ofctl show ovs-br``` 與 ```ovs-dpctl show``` 兩者 interface 及 port number 的關係, 可以發現 ovs-vswitchd 針對綁定的 interface 有自己的 ofport number, 然而相同 interface 在 datapath 又有不同的 number, 所以在 trace flow 要特別注意這類 mapping 問題, 千萬不要弄混

接下來還會有 OpenvSwitch Debug$  Trace flow 的文章, 請讓我繼續拖搞, 謝謝各位
