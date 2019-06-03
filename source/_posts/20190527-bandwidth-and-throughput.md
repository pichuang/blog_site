layout: post
title: Bandwidth & Throughput
author: Phil Huang
tags:
  - misc
categories:
  - network
date: 2019-05-27 11:42:00
---
![](/images/throughput-bandwidth.png)

- 頻寬 (Bandwidth)：水管的大小  
- 吞吐量 (Throughput)：從水管實際流出的水量  
- 延遲 (Latency)：水從一端到另一端所花費的時間  

<!--more-->

## 理論值頻寬 (Bandwidth) 換算

```
1 Gbps = 1024 Mbps = 128 MBps = 0.125 GBps
7 Gbps = 7168 Mbps = 896 MBps = 0.875 GBps
10 Gbps = 10240 Mbps = 1280 MBps = 1.25 GBps
20 Gbps = 20480 Mbps = 2560 MBps = 2.5 GBps
25 Gbps = 25600 Mbps = 3200 MBps = 3.125 GBps
40 Gbps = 40960 Mbps = 5120 MBps = 5 GBps
50 Gbps = 51200 Mbps = 6400 MBps = 6.25 GBps
100 Gbps = 102400 Mbps = 12800 MBps = 12.5 GBps
```

## 資料傳輸換算

理論傳輸公式：Time (s) = File Size / Bandwidth  
實際傳輸公式：Time (s) = File Size / Throughput

