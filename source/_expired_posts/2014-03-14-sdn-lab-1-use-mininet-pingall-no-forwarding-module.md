---
layout: post
title: 'SDN Lab 1$ Use Mininet pingall (NO Forwarding Module)'
date: 2014-03-14 00:50
comments: true
categories: 
---
# 前言
其實這個是份作業, 認為非常有趣放上來紀錄一下, 作業目的是利用 Floodlight 配上 mininet 利用 OpenFlow 做 Forwarding, mininet topology 是設定為```--topo=tree,2```(如圖)
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/188239/bp89nmjiQyWnwTWIqT7d_mn_tree_2.png" alt="mn_tree_2.png">
測試目標是利用 mininet 內建的 pingall 來測試 h1, h2, h3, h4 這幾個 host 互 ping 是否可以相通, 當然如果有用過 Floodlight 的人知道說其實他裡面有一個 Forwarding Module 已經達成這個目標了, 但此份作業要將它移除, 需要自己手動下 Flow

# 過程
## 安裝 mininet + floodlight
* 忽略不寫 網路很多教學

## 移除 Floodlight Forwarding Module
> sed -i '/net.floodlightcontroller.forwarding.Forwarding,\\/d' floodlight/src/main/resources/floodlightdefault.properties  
sed -i '/net.floodlightcontroller.forwarding.Forwarding/d' floodlight/src/main/resources/META-INF/services/net.floodlightcontroller.core.module.IFloodlightModule  
ant  

  * 透過前兩者指令可以有效的將 Floodlight 內建的 Forwarding 拿掉
  * 最後一定要做 ant rebuild 的動作

## 開啟 mininet 
> mn --topo=tree,2 --controller=remote,ip=127.0.0.1

* 這時候可以直接在 Floodlight 的 WebUI 上看到三台 OpenFlow Switch 
* ```mininet> pingall``` 如果上面都是正確的話 應該會看到以下燒毀的狀況
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/188239/wrKwjBKWTaeJDe1qsDVg_pingall_fail.png" alt="pingall_fail.png">

## 使用 Floodlight Static Flow Pusher
* [Static Flow Pusher API](http://www.openflowhub.org/display/floodlightcontroller/Static+Flow+Pusher+API)
* 使用 Static Flow Pusher 可以省去很多撰寫 json 格式的麻煩, 雖然他的 code 也非常的簡單, 可以參考 **#Using Static Flow Pusher in practice**, 範例已經可以讓同一個 switch 底下的 hosts 互 ping 到
* 刪除一條是採 ```curl -X DELETE -d '{"name":"flow-mod-1"}' http://<controller_ip>:8080/wm/staticflowentrypusher/json``` 
* 刪除全部則是 ```curl http://<controller_ip>:8080/wm/staticflowentrypusher /clear/<dpid>/json```

## 想法
* pingall 的狀況下應該是可以利用 actions=flood 來達到, 也就是三個 switch 都當成 hub 來使用
* 針對 s2, s3 部份, 無論從哪個 ingress-port 進來都進行 actions=flood
* 針對 s1, 因他只有兩個 port, 所以 actions=ouput:x (x=1,2)
* 以上可以很簡單達到 ```h1 ping h4``` 的效果

## 實作
* [Source Code](https://gist.github.com/pichuang/9533626)
* flow1 ~ flow3 為處理 s2, flow4 ~ flow6 為處理 s3, 兩者皆為進行 actions=flood 的動作
* flow7 flow8 為處理 s1, 這邊做的是將兩個port的封包指定互丟, 其實用 flood 也可以

# 心得
* 其實這個是有點 bypass 的做法, 原先助教應該是想要我們針對每個 ethernet type 下 flow, 進行 Ping 動作其實是有發送ARP + ICMP 的行為
* 實地測過將 actions=flood 換成 normal, all都可以, 
  * [OpenFlow Spec v1.0.0](https://www.opennetworking.org/images/stories/downloads/sdn-resources/onf-specifications/openflow/openflow-spec-v1.0.0.pdf) pages 8
  	* Normal 是指進行 Legacy Switch 行為
    * Flood 是依據 Spanning Tree 結果下去傳 #shanyu
* Floodlight 預設不會把 STP Module 打開
* **hwchiu is my colleague, shanyu is our leader.**
