---
layout: post
title: 'RIP (Route Information Protocols) Introduction'
date: 2014-08-30 20:52
comments: true
categories: 
---
介紹
===
* 屬於 IGP (Interior gateway protocol) 之一
* 屬於 Distance-Vector routing protocols 之一
	* 利用 hops count 做度量(metric)
  	* 每經過一台 router, hops count ++
* 採用 Shortest Path Problem 的 Bellman-Ford Algorithm [動畫展示](http://www.comp.nus.edu.sg/~stevenha/visualization/sssp.html)
* 可傳遞範圍至多 15 hops, 超過視為不可到達
* 透過與 Neighbor 交換 routing table 進行更新
* 共有 RIPv1, RIPv2, RIPng [詳細比較](http://blog.xuite.net/lichangying/wretch/176501055-RIP+v1%E8%88%87RIP+v2%E8%B7%AF%E7%94%B1%E5%8D%94%E8%AD%B0%E5%B0%8D%E6%AF%94%E5%88%86%E6%9E%90)

Timer
=====
* Update timer
  * 每 30 秒 Broadcast 自己的 routing table 
* Invalid timer
	* 每 180 秒 若 Routing table 內的 Path 未被確認會被視為 Invalid, 且開始計算 holddown timer
  * Path 還存在 Routing table 內
* Holddown timer
	* 當 Path 被判斷為 Invalid 時, 此時間內不再接收同一 dest source, 除非從其他 dest source 接收到更好的 path metric, 或直到 timer expiers 後, 才重新接收
* Flush timer
	* 每 240 秒 若超過時間該 Path 會被 Flush
  * Path 不存在 Routing table 內

<img class="center" src="https://lh4.googleusercontent.com/-x7LpS-q5WUg/VAHgDjZqjLI/AAAAAAAAFTU/mC6N7YedrA8/w1916-h1078-no/rip_timer.jpg">
  
RIP loop prevention mechanism
=============================
* Split Horizon
	* 保證 router 記住接收 Path 的 interface, 而不會再同一個 interface 發送它
	<img class="center" src="http://study-ccna.com/images/split_horizon.jpg">

* Poison Reverse
	* 情境
		1. 當 hosts 與 R1 連線中斷時
		2. 因處於 Invalid time 180s, 故 R2 認為 hosts 至 R1 還存在連線
		3. 導致整個網路資料不正確, 有可能會有 loop 情形產生
		<img class="center" src="http://study-ccna.com/images/route_poisoning.jpg">
    
	* Poison Reverse 做法
		1. 發現 hosts 至 R1 連線中斷
		2. R1 將此 Path metric 設為 16 (Hop count = Max + 1 ), 即為不可到達之狀態
		3. R1 發送 Routing update 至 R2
		4. R2 更新資料將該 Path 列為 Inaccessible

* Trigger Update
	* Routing table 發生變化時, 無視 Update Timer, 透過 broadcast 直接發送 routeing update 至所有設備
  
不連續網路問題
============
* 針對不同 Router 上同一 subnet 進行 summary, 雖會減少 Routing Table rule, 但卻會造成 Equal cost load balance 問題
* 解法是 ```no auto-summary```
	* [蕭老師 200-120 CCNA 認證教學 5-32](http://youtu.be/VtjEui7PgnE?t=11m34s)
  
NBMA (Non-Broadcast MultiAccess)
================================
* 有些設備不允許傳送 Broadcast 封包
* 解法是指定某一個 Neighbor




Reference
=========
* [IP Routing Protocol -- RIP 之介紹](http://www.clyeh.org/iprip/iprip.htm)
* [路由信息協議-wiki](http://zh.wikipedia.org/wiki/%E8%B7%AF%E7%94%B1%E4%BF%A1%E6%81%AF%E5%8D%8F%E8%AE%AE)
* [100分鐘，我教你配CISCO RIP](http://fanqiang.chinaunix.net/a3/b1/20010419/135120_b.html)
* [How basic are RIP timers? Test your knowledge now.](http://blog.ine.com/2010/04/15/how-basic-are-rip-timers-test-your-knowledge-now/)
* [RIP loop prevention](http://study-ccna.com/rip-loop-prevention)
* [【CCNA】實驗RIP協定](http://my.stust.edu.tw/49790118/doc/5686)
* [RIP v1與RIP v2路由協議對比分析](http://blog.xuite.net/lichangying/wretch/176501055-RIP+v1%E8%88%87RIP+v2%E8%B7%AF%E7%94%B1%E5%8D%94%E8%AD%B0%E5%B0%8D%E6%AF%94%E5%88%86%E6%9E%90)
* [RIP路由協議](http://ccna2012.weebly.com/rip.html)