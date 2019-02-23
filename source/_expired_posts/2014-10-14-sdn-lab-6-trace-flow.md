---
layout: post
title: 'SDN Lab 6$ Trace Flow'
date: 2014-10-14 20:59
comments: true
categories: 
---
若想觀察 Openflow packet 有沒有正確的被 match, ovs-appctl 有提供了一個 function 可以給大家測試用

環境設定請參考拙作 [SDN Lab 5$ REST and Ryu](http://roan.logdown.com/posts/233613-sdn-lab-5-rest-and-ryu)

建立了一個 linear,2 Topology, 此時利用  ```ovs-appctl ofproto/trace``` 來產生出 packet 來 trace flow以下是產生一個來自 in_port=1 的封包, 送到 s1 的處理流程,

> ovs-appctl ofproto/trace s1 in_port=1,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02 -generate

注意這邊 ryu 有使用到 simple_switch_13.py [ryu_rest_testing](https://gist.github.com/pichuang/8ec2ad6eec8b363bd300)

<img class="center" src="https://lh3.googleusercontent.com/-1_0cbqBkJNQ/VD0nx0BgzhI/AAAAAAAAFg8/KHt07WcDZNY/w1916-h352-no/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-10-14%2B21.40.03.png">

我們可以看到 s1 最終決定將 packet 打到 ```actions=out_port:2``` 


若將 Controller 關閉之後, 在啟動同樣 Mininet Topology, 也產生了一樣的 packet, 則會產生下圖

<img class="center" src="https://lh4.googleusercontent.com/-eGmwWg5x46Q/VD0sGY5iZVI/AAAAAAAAFhQ/RLYgoxQ5N_g/w1914-h506-no/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-10-14%2B21.56.53.png">

看官可以看到 s1 進行了 ```actions=controller```, 產生了一個 packet_in 至 Controller, 但因為 Controller 並沒有開起來, 沒人可以處理這個 packet_in, 所以 ```Datapath actions: drop``` 

到這邊應該是用非常簡單地描述針對單一 ovs sw 做 trace flow 的動作, 更詳細的可以在研究一下, 這以下是不是還有什麼可以玩的 
