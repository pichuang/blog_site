---
layout: post
title: "Prepare Virtual Enviroment"
description: ""
date: 2015-07-28 00:00:00 +0800
updated: 2015-07-28 00:00:00 +0800
category: Infra
tags:
- infra
- vmware
---

## Goal
此篇目標是要利用一些實際數值來估算虛擬環境所需實際用量, 多數虛擬環境使用效能不佳, 都是起始規劃有問題, 規劃包含 Networking, Storage, CPU, Memory 等. 若能事先透過較為精準的估算, 即可讓此狀況大幅改善.

### 估算虛擬環境用量
1. 利用一些 Monitor 的專案, 抓取重要且日常皆會使用的資料, 包含 CPU, Memory, Read/Write IO
2. 利用一般環境的資料做估算, 但此方式不是針對維運狀況做估算, 可能會有不小的標準差. 時間允許的狀況下, 建議還是採用方案一會比較適當

### 估算單台 Server 上所能負載的 VM 數
依據 Server 上的 CPU/Memory 所決定, 這邊使用 ```IBM E5520 @ 2.27GHz``` 為例
- 4 Cores
- 不開 HT
- 開 vt-d

### 利用 CPU 估算 VM
假設一 Linux VM (800 Mhz), CPU 預估 Buffer 20%  
故 ```4 * 2270 / 800 * 1.2 = 9 台VM```

### 利用 Memory 估算 VM
若有一台 Memory 80 GB Server, 假設一 Linux VM 使用 1GB, 預估 15% buffer for overhead, 20% over-commit rate  
故 80 - 1 (ESXi Server) - 2 (VSA) = 77  
```77 / 1 * 1.15 * 0.8 = 83 台VM```  

### Reference
- [VMware 虛擬化技術實務問答 (上)](http://wiki.weithenn.org/cgi-bin/wiki.pl?VMware_虛擬化技術實務問答_(上))
