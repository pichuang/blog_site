---
layout: post
title: '編譯 OpenWrt'
date: 2013-12-07 04:20
comments: true
categories: 
---
# 前言
最近在研究 Wireless AP SDN 需要準備一個可以開發 AP 的環境, 原廠的韌體沒有放出 Source Code, 也沒有辦法把自己寫的程式 porting 上去, 故採用 OpenWrt 達成需求. 

建議各位如果要作類似的編譯環境及開發環境, 利用 Virtual Machine 獨立出來, 避免因環境不乾淨造成可能會踩到的地雷. 再來 Virtual Machine 有一個很棒的功能就是 Snapshot, 這功能各家都有做, 在進行大幅度開發或者是更動前, 先 Snapshot, 避免炸了還要整個重來一次, 時間是花在刀口上的.

# 安裝流程
## 環境準備
- Ubuntu 14.04.1 LTS x86_64
```
sudo apt-get install build-essential subversion git-core libncurses5-dev zlib1g-dev gawk flex quilt libssl-dev xsltproc libxml-parser-perl unzip
```
- 需要一般用戶帳號, 不能使用 root 編譯, 不然會噴錯 *Build dependency: Please do not compile as root.*


## 下載原始檔
1. trunk (develope version) ```git clone git://git.openwrt.org/openwrt.git```
2. 14.07 (stable version) ```git clone git://git.openwrt.org/14.07/openwrt.git```

一般建議採用 stable version, 避免再未知的 develope version 踩到雷, 目前 stable version 為 **14.07 Barrier Breaker**

## 編譯
```
cd openwrt
mv feeds.conf.default feeds.conf
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
make -j 
```
- make menuconfig 較為重要的以下兩個資訊, 可以上 OpenWrt 官網查, 每台 AP 都不一樣
  - Target System 
  - Target Profile 
- OpenWrt 編譯的設定檔會吃 ```.config``` , 檔案放在openwrt目錄底下
- 若需 Predefined config 的話, 可以再 package 底下進行設定

## 編譯後的 binary 
- 位置於 ``` openwrt/bin/<chip name>/``` 底下有兩個更新方式
  1. "*-factory.bin"
		* 適用於從未安裝過 OpenWrt 的 AP 就是我們經常看到的韌體檔案, 請將他上傳至韌體(Firmware)更新頁面即可
  2. "*-sysupgrade.bin"
		* 適用於已經安裝過 OpenWrt 的 AP, 請使用 scp 將檔案放置到 OpenWrt-AP 的 /tmp 資料夾
		* 使用 ```sysupgrade -v *-sysupgrade.bin``` 更新, 記得要按 [w]
    
## 連線登入
- 使用 ```ping 192.168.1.1``` 是否有回應, 建議檢查一下網路, 網路線要插在 lan port上
- OpenWrt 預設第一次登入開放 telent (port 23 無密碼), 請使用 ```telnet 192.168.1.1``` 登入後, 執行 ```passwd```更改root 密碼後, 登出後, 再使用 ```ssh root@192.168.1.1```, 輸入密碼即可, 這時候 OpenWrt 就會將telnet上防火牆關閉  

## 結語
這是一個很不錯的 embedded Linux system, 且這專門針對市面上的 AP 作為基礎, 對於無線網路這塊的性能相較會比較要求, 如果有在玩 Linux 的話, 相信上手應該會很快.

# Reference
  - [設定 OpenWrt](http://roan.logdown.com/posts/191761-set-the-openwrt)
  - [Internal Device Network](http://www.dd-wrt.com/wiki/index.php/Internal_device_network)
 



