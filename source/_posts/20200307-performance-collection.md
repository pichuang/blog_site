layout: post
title: 尋找當下 Linux OS 效能瓶頸
author: Phil Huang
tags:
  - linux
  - performance
categories:
  - linux
date: 2020-03-07 00:16:00
toc: true
---

常常聽到客戶說他們的系統很慢，但說慢在哪裡也沒有一個很明確的說明，這邊提供一個簡單的做法，當遇到`當下`系統很慢的時候，可以有個`數字判斷`的依據尋找瓶頸點 (Bound)，這篇單純就 OS 層面說明，不討論應用程式等級的效能問題

![](http://www.brendangregg.com/Perf/linux_observability_sar.png)

<!--more-->

## 系統壓力觀測目標

> sar - Clliect, report, or save system activity information

本篇將會用 `sar` 系統活動報告器來貫穿全部操作，主要觀測點會有 5 個

1. CPU
2. Disk I/O
3. Network
4. Memory
5. Swap

選用 sar 的理由是
1. 數據收集過程盡可能對系統本身不會造成無謂的壓力
2. 大多數 Linux 發行版皆可用
3. 簡單指令
4. Linux 目前較為全面的系統效能分析工具之一

## 環境介紹
- Red Hat Enterprise Linux 8.1

## 安裝

```bash
sudo yum install -y sysstat
```

## 常常遇到的問題

> [聲明] 如果底下的欄位解釋，覺得翻的很爛，可以參考 `man sar` 的英文版解釋

### 懷疑系統是不是真的忙?

```bash
sar -q 1 60
```
- 指令解釋: 每隔 1 秒收集一次，一共收集 60 次平均常見的負載 (1m / 5m / 15m)數字

![](/images/sar-q.png)

- 欄位解釋:
    - runq-sz: 運行中的 Quene 長度 (等待運行的 Tasks 數)
    - plist-sz: Tasks 列表中的 tasks 數量
    - ldavg-1: 最後 1 分鐘的系統平均負載量
    - ldavg-5: 最後 5 分鐘的系統平均負載量
    - ldavg-15: 最後 15 分鐘的系統平均負載量

- 數字解讀:
    - 現在都是支援多核 CPU，故 `n` 個 CPU Core，可接受的系統負荷最大 `n.0`
    - 確認系統核心數的指令是 `grep -c 'model name' /proc/cpuinfo`
    - 通常來講是看 `ldavg-15`，如果長期接近或大於 n 的話，就是系統負擔過大

### 懷疑是 CPU Bound?
```bash
sar -u 2 10 //看 CPU Overview 使用率 (與 mpstat 相同)
sar -P ALL 2 10 //看個別 CPU 使用率 (與 mpstat -P ALL 相同)
```
- 指令解釋: 每隔 2 秒收集一次，一共收集 10 次所有 CPU 的效能數據

![](/images/sar-u.png)

![](/images/sar-P.png)

- 欄位解釋
    - %user: 於 User (Application) 等級之下，消耗的 CPU 時間比例
    - %nice: 透過 `nice` 改變優先級，於 User (Application) 等級之下，消耗的 CPU 時間的比例
    - %system: 於 System (Kernel) 等級之下，消耗的 CPU 時間比例
    - %iowait: CPU 等待 Disk I/O 導致空閑狀態 (Idle) 消耗的時間比例
    - %steal: 利用虛擬化技術，等待其他 vCPU 計算佔用的時間比例
    - %idle: CPU 空閑時間比例

- 數字解讀
    - `%iowait` 過高，有可能是 Disk I/O 瓶頸，要改善 Disk
    - `%idle` 過高，有可能是 CPU 等待分配 Memory，要看一下記憶體是不是不夠
    - `%idle` 持續低於 10，則代表 CPU 處理效率低，需要解決 CPU 瓶頸

### 懷疑是 Network Bound?
```bash
sar -n DEV 2 10
```
- 指令解讀: 每隔 2 秒收集一次，一共收集 10 次所有網卡的效能數據

![](/images/sar-n.png)

- 欄位解釋:
    - rxpck/s: 每秒接收到的封包量
    - txpck/s: 每秒傳送出的封包量
    - rxkB/s: 每秒接收到的 bytes 數量
    - txkB/s: 每秒傳送出的 bytes 數量
    - rxcmp/s: 每秒接收到的壓縮後封包
    - txcmp/s: 每秒傳送出的壓縮後封包
    - rxmcst/s: 每秒接收到的群播封包
    - %ifutil: 網路介面使用率

- 數字解讀:
    - `rxpck/s` 和 `txpck/s` 是衡量一張網卡效能的指標，數字越大壓力越大
    - `rxkB/s` 和 `txkB/s` 是代表吞吐量效能，數字越大壓力越大

### 懷疑是 Memory?
```bash
sar -r 2 10
```
- 指令解釋: 每隔 2 秒收集一次，一共收集 10 次記憶體的效能數據

![](/images/sar-r.png)

- 欄位解釋
    - kbmemfree: 閒置的 Free Memory 大小
    - kbavail: 無需透過 Swapping 即可使用的 Memory 大小
    - kbmemused: 使用中的 Memory
    - %memused: 物理記憶體使用率
    - kbbuffers: Kernel 拿去作為 Buffer 的大小
    - kbcached: Kernel 拿取當作 Cached 的大小
    - kbcommit: Kernel 當前工作負載所需要的量，這是預估需要多少 RAM/Swap，以保證不會有記憶體不足的估計值
    - %commit: Kernel 當前工作負載所需記憶體佔記憶體總量 (RAM + Swap) 的百分比
    - kbactive: 活躍中的記憶體，除非必要，通常不會回收 (Reclaimed)
    - kbinact: 不活躍中的記憶體，最近被使用的次數較少，有可能會被回收
    - kbdirty: 等待寫回至磁碟的記憶體內容

- 數字解讀
    - 透過 `%memused` `kbbuffers` `kbcached` 可以了解到記憶體實際使用量
    - `%commit` 這是 `kbcommit` 與記憶體總量 (包含swap) 的百分比數字，越小代表記憶體越足夠
    - `%commit` 有可能會大於 100%，有可能 Kernel 使用過量記憶體

### 懷疑是 Swap Page Bound?
```bash
sar -W 2 10
```
- 指令解讀: 每隔 2 秒收集一次，一共收集 10 次 Swap page out/in 的效能數據

![](/images/sar-W.png)

- 欄位解釋
    - pswpin/s: 每秒系統 Swap page IN 的數量
    - pswpout/s: 每秒系統 Swap page Out 的數量

- 數字解讀
    - 這個只要有數字跳出來，伺服器處理吞吐量會大幅下降，這時候要先確定是不是記憶體夠不夠

### 懷疑是 Disk I/O Bound？
```bash
sar -d 2 10
```
- 指令解讀: 每隔 2 秒收集一次，一共收集 10 次 Disk I/O 的效能數據

![](/images/sar-d.png)

- 欄位解釋
    - tps: 設備每秒數據傳輸量
    - rkB/s: 每秒從設備讀取的資料量
    - wkB/s: 每秒寫入到設備的資料量
    - areq-sz: 發出到設備 I/O 請求的平均資料大小
    - aqu/sz: 發出到設備的 I/O 請求的平均 Queue 長度
    - await: 發出到服務的設備請求 I/O 的平均反應時間
    - svctm: 不用看，未來要移除
    - %util: 像設備發出 I/O 請求的經過時間百分比

- 數字解讀
    - `%util` 數值越大越忙
    - 如果剛好有 raid 之類的話，`%util` 有可能會大於 100%
    - `iostat` 比較詳細，針對 Disk I/O 更建議使用

## 結語

當然， `sar` 並不是一個長期監控很好的方案，如 Red Hat Enterprise Linux 更推薦的使用是用 `Performance Co-Pilot (pcp)` [pcp 快速門][3] 來進行系統長期監控維運

但透過此篇，希望大家未來聽到類似的系統很慢的問題時，可以先利用 `sar` 先進行快速了解，建立一個簡單的了解，再來花時間計畫比較長期的系統監控方式

## References
- [sar 找出系统瓶颈的利器][1]
- [理解Linux系统负荷][2]
- [Performance Co-Pilot][3]
- [Linux ate My RAM][4]
- [Linux 的記憶體快取（Cache Memory）功能：Linux 系統把記憶體用光了？][5]

[1]: https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/sar.html
[2]: https://www.ruanyifeng.com/blog/2011/07/linux_load_average_explained.html
[3]: https://pcp.io/
[4]: https://www.linuxatemyram.com/
[5]: https://blog.gtwang.org/linux/linux-cache-memory-linux/