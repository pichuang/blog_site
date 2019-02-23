---
layout: post
title: "Hackathon Taiwan 7th 網路部署紀錄"
description: "Cable Hackathon =_="
date: 2015-05-19 00:00:00 +0800
updated: 2015-05-19 00:00:00 +0800
category: Event
tags:
- event
- hackathon
---

2015/05 因為 lzy 的各種勸說的關係, 我參加了 [Hackathon Taiwan 7th](https://hackathon.tw/) 網路工作人員, 還順便推坑了[舜翔](https://www.facebook.com/keats.hu)和[少友](https://www.facebook.com/shaoyou.wu) XD, 來幫忙協助處理網路佈建的部分, 會場長得如下  
 
<center><img src="https://fbcdn-sphotos-g-a.akamaihd.net/hphotos-ak-xpa1/t31.0-8/s2048x2048/11217746_357265184463516_633408084956955373_o.jpg" width="400" height="200"></center> 

佔地超大, 我看應該是拿大樓的停車場拿來改建的場地, 猜想出資的人應該來頭不小.

我們到的時候是禮拜五的下午一點多, 首先先遇到了 [Kevin](https://www.facebook.com/chiehwenyang), 他是黑客松台灣第七屆的總招, 真是辛苦他了. 首先一開始就請他跟我們介紹一下場地概況和設備狀況, 這是因為我跟兩個同學都沒有參加過行前會 (新竹跟台北還是有段距離的), 以下開始就是辦正事的紀錄

## 收集資料

一開始我要的資料就是要設備清單和平面圖, 首先是設備清單, 這活動因為有 [NetgearTaiwan](https://www.facebook.com/NetgearTaiwan) 的贊助, 所以有一堆價格貴鬆鬆的 AP 可以用

1. Netgear R8000 * 1
2. Netgear R7500 * 7
3. HP Procurve 2900-24g
4. Cisco ASA 5510
5. IBM 3250M2
6. 少數幾台 L2 switch {D-link, Cisco, Linksys}

其次平面圖主要是要決定拉線及 AP 的部署位置, 這件事很重要, 所有的網路工作都會透過圖來表示, 以下是我們當日畫得非常爛的圖XDDDDD  

<blockquote class="instagram-media" data-instgrm-version="4" style=" background:#FFF; border:0; border-radius:3px; box-shadow:0 0 1px 0 rgba(0,0,0,0.5),0 1px 10px 0 rgba(0,0,0,0.15); margin: 1px; max-width:658px; padding:0; width:99.375%; width:-webkit-calc(100% - 2px); width:calc(100% - 2px);"><div style="padding:8px;"> <div style=" background:#F8F8F8; line-height:0; margin-top:40px; padding:50% 0; text-align:center; width:100%;"> <div style=" background:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACwAAAAsCAMAAAApWqozAAAAGFBMVEUiIiI9PT0eHh4gIB4hIBkcHBwcHBwcHBydr+JQAAAACHRSTlMABA4YHyQsM5jtaMwAAADfSURBVDjL7ZVBEgMhCAQBAf//42xcNbpAqakcM0ftUmFAAIBE81IqBJdS3lS6zs3bIpB9WED3YYXFPmHRfT8sgyrCP1x8uEUxLMzNWElFOYCV6mHWWwMzdPEKHlhLw7NWJqkHc4uIZphavDzA2JPzUDsBZziNae2S6owH8xPmX8G7zzgKEOPUoYHvGz1TBCxMkd3kwNVbU0gKHkx+iZILf77IofhrY1nYFnB/lQPb79drWOyJVa/DAvg9B/rLB4cC+Nqgdz/TvBbBnr6GBReqn/nRmDgaQEej7WhonozjF+Y2I/fZou/qAAAAAElFTkSuQmCC); display:block; height:44px; margin:0 auto -44px; position:relative; top:-22px; width:44px;"></div></div><p style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; line-height:17px; margin-bottom:0; margin-top:8px; overflow:hidden; padding:8px 0 7px; text-align:center; text-overflow:ellipsis; white-space:nowrap;"><a href="https://instagram.com/p/2tyb_3iiHE/" style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; font-style:normal; font-weight:normal; line-height:17px; text-decoration:none;" target="_top">小飛機（@inceptioninx）張貼的相片</a> 於 <time style=" font-family:Arial,sans-serif; font-size:14px; line-height:17px;" datetime="2015-05-15T20:08:51+00:00">2015 年  5月 月 15 1:08下午 PDT</time> 張貼</p></div></blockquote>
<script async defer src="//platform.instagram.com/en_US/embeds.js"></script>

這張圖主要著重在: AP 放哪裡? 對外專線出口在哪個位置? 線怎麼拉? 這些決定好了, 有些事情就能先開始做了, 像拉線這類耗時耗人力的工作, 就可以趁體力好的時候先趕快弄一弄, 那天剛好有個阿伯會拉線, 替我們省了不少體力.

整體的規劃理念是: ```有多少資源做多少事, 設備盡量專一服務, 穩定網路比網路快速重要```

## 對外出口
* 中華電信線路
    * 300/100 * 2 (PPPoE)
    * 100/100 * 1

## AP 規劃

1. 走 Bridge mode
  * 不走 Router mode 的理由是, 讓 AP 完全將資源投注處理 wifi authentication 和 association, 不讓他另外多做其他的事情, nat 及 dhcp 等其他服務都另外拉出來做
2. 2.4GHz 選 802.11n, 單純跑 5GHz 選 802.11ac
    - 802.11n 同時支援 2.4Ghz 及 5GHz, 是非常建議的選擇
3. Wifi Power tranfer 調小
4. 2.4GHz channel 選擇 1, 6, 11 這三個互相不會有 overlapping 情形的 channels
5. 使用 2.4GHz AP 總台數, 要多於 5GHz AP 台數
    - 2.4GHz 繞射較 5Ghz 好
    - 目前僅支援 2.4GHz 裝置數目依然是頗為可觀
6. 密碼採 WAP2-PSK + AES 加密 
    - 無線封包不加密, 就像在出門沒穿內褲一樣

## Server 規劃
1. IBM 3250M2 做 NAT Server + DHCP Server
    - based on Debain, 所有不必要的服務全部都 stop 兼 disable
    - 分配 Private subnet /22
    - nat 用 iptables 做掉, dhcp 則用 dnsmasq
2. Cisco ASA 5510 做 NAT Server + DHCP Server
    - 因為 dhcpd 支援不力的關係, 僅能切 /24 出去
    - IPS 功能我沒開, 開了整個 throughput 僅有 150 Mbps, 速度大打折扣
3. 防止 DoS 及 Broadcast storm
    - 針對 IBM 3250M2 調了不少的 kernel parameters 及 iptables 防禦
4. Squid3 
    - 原先有在 IBM 3250M2 上弄好 Squid3 來做 web cache 的動作, 但後來因為 debug 某個狀況就把它關掉了, 沒有使用到有點可惜
5. Tools
    - htop
    - iftop
    - iptstate
    - hping3
        - 壓測用

## Switch 規劃
1. HP Procurve 2900-24g 
    - Broadcast stroam control
    - Vlan tagging/untagging
    - QoS based on L3 protocol
2. 剩餘 switch, 全部都是 dummy switch 沒做任何修改及設定

## 實測
其實我覺得這次比較沒有參考價值, 同一時間有[政大黑客松](http://www.hacknccu.org/), ModernWeb 及 Intel IoT 等三個中大型活動分散掉了人潮. 人少, 設備不差, 網路當然好, 所以對於之後實測的數據都不覺得有什麼可以拿出來說嘴

## 後記
以上看似很簡單且零零總總的紀錄, 竟也讓我從禮拜五中午 13:00 一路忙到禮拜六早上 07:00, 中間沒有睡覺, 提早進行網路佈建馬拉松. 這當中要非常感謝少友和舜翔的協助, 如果沒他們兩位我應該就累死在那邊了, 此外還要感謝兩位明道中學的小高一, 從半夜跟我弄東弄西到早上, 因覺得態度還不錯, 所以教了一些有的沒的給他們, 他們也當場現學現賣幫忙把線路拉好, 裝置都準備好, 幫助不小, 這邊也要特別感謝那兩位, 期望他們之後能成為社群弄網路的中堅戰力

因我是第一次參加這活動, 跟全部的工作人員都不熟, 自然也鬧出笑話, 工作人員不知道我是幫忙弄網路的, 結果碰設備的時候被大力地關切了 XDDD

身為弄 SDN 的小小, 這種場地非常適合用 ONOS Cluster + OpenFlow enabled AP 來做控制, App 採用 nat, proxyarp, netconf...etc 來做, 我在想理論上應該效率及管理方便性會非常好, 但問題推測應該沒有意外地會卡在 AP 的處理效率上, 感覺 wireless sdn 還是有段不小的路要走

## 後續檢討
1. 有一區線路規劃並沒有設計的很好, 短距離內有多台 AP 互相干擾, 導致無線網路不穩定, 多人用的時候會很明顯
2. Netgear R8000 這台機器太強大, 整個會場都收得到他的訊號, 想把他的 power 關小一點, 還找不到地方可以調, 我應該要把它放在中間的位置來處理大部份的 Station, 以他為中心
3. 線路應該要統一線材及規格, 還有要打標籤
4. AP Management 機制太晚處理, 導致後續要再花一倍時間上設定
5. R8000/R7000 或許刷 OpenWrt 會比較好控制, 傳輸效率上或許能更好, 但需要實測.
6. NAT Server 應該要拿個 Software Router OS 來做, 譬如 [pfSense](https://www.pfsense.org/), [VyOS](http://vyos.net/)...等
7. 若之後有用 OpenWrt 的話, 設定部分可以找個 IT Automation software 來做, 譬如拿 [Ansible](http://www.ansible.com/home) 就很適合 

## Feedback
- [FB討論串](https://www.facebook.com/paulintoro/posts/1070092773005774)
- hwchiu 牛說 broadcom 有支援 ctf (Cut Through Forwarding), 主要是做 bypassing linux's stack, Forwarding 沒意外效率會好很多, 剛好 Netgear R8000 就有這顆晶片, 查了下文章 [Cut-Through Forwarding necessary on dd-wrt and tomato for throughput higer than 300Mbit](http://www.kmggroup.ch/?p=604) 說刷 DDWrt 就能用了

## Reference
- [2.4Ghz 與 5Ghz 穿透性與傳輸效果](http://diveduino.blogspot.tw/2014/05/24ghz-5ghz.html)
- [Cisco ASA 5510 UTM 基礎設定](http://wiki.weithenn.org/cgi-bin/wiki.pl?%E5%BB%BA%E7%BD%AE%E9%AB%98%E5%8F%AF%E7%94%A8%E6%80%A7_Cisco_%E9%98%B2%E7%81%AB%E7%89%86_(%E4%B8%8A)#Heading9)
- [List of router and firewall distributions](http://en.wikipedia.org/wiki/List_of_router_and_firewall_distributions)
- [Broadcom's hardware acceleration](http://www.snbforums.com/threads/broadcoms-hardware-acceleration.18144/)
