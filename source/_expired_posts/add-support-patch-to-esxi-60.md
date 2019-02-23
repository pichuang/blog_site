---
layout: post
title: "Add support patch to ESXi 6.0"
description: "patch your heart"
date: 2015-05-19 00:00:00 +0800
updated: 2015-05-19 00:00:00 +0800
category: Infra 
tags:
- infra
- vmware
- esxi
---

## Goal
目標很簡單, 我只是想要在桌電裝一個 ESXi6.0 來玩玩, 遇到了一個硬碟找不到的問題, 幸好學弟有先寫了篇文章提醒, 這邊紀錄一下打 patch 的經過

## Environment
- G1 SNIPER Z97
- E3-1231v3
- 4GB Memory
- WDC WD10EZEX Blue
- Intel NIC

### Download ESXi 6.0 iso
[官方網站下載, 要申請帳號密碼](https://my.vmware.com/web/vmware/evalcenter?p=free-esxi6) , 若嫌懶的話, 可以看一下 Reference 第一個連結

### Download Sata-xahci
- [Download-sata-xahci-1.30.1](http://vibsdepot.v-front.de/depot/bundles/sata-xahci-1.30-1-offline_bundle.zip)
    - 主要是要替 ESXi iso 上一個 AHCI patch, 讓 ESXi 能順利讀到硬碟
    - 若想要找其他 patch 的話, 可以透過 [List of currently available ESXi packages](https://vibsdepot.v-front.de/wiki/index.php/List_of_currently_available_ESXi_packages) 找到適合的套件

### Download ESXi-Customizer
- [Download-ESXi-Customizer-v2.7.2](http://vibsdepot.v-front.de/tools/ESXi-Customizer-v2.7.2.exe)
    - 雖然官方沒寫說有支援 ESXi 6.0 但我實裝起來是毫無問題的

### Use ESXi-Customizer
設定如下圖所示

<center><img src="https://lh5.googleusercontent.com/-okhfaLY9Jko/VVtCGLEd99I/AAAAAAAAG74/7DEl_7cvumc/w879-h755-no/esxi6-sata-ahci-2.PNG" width=100% height=100%></center>

- 第一格填下載下來的 ESXi 6.0 iso 位置
- 第二格填下載下來的 sata-xahci-1.30.1, 記得要更改檔案格式為```*.zip```
- 第三格填客制化後的 ESXi image location 

點選 ```Run``` 後, 會進行 patch 確認, 這邊就是進行 ```sata-xahci```的確認

<center><img src="https://lh6.googleusercontent.com/-GtYNg-Ib7mE/VVtCGJl5CHI/AAAAAAAAG8A/XUvrRxaGPhA/w930-h798-no/esxi6-sata-ahci-4.PNG" width="100%" height="100%"></center>

跑完後, 你可以再你填的第三格位置裡找到一個名為 ```ESXi-5.x-Custom.iso``` 的檔案

### Use UNetbootin

找一個適當大小的 usb, 將 ```ESXi-5.x-Custom.iso``` 放進去後就大功告成了

<center><img src="https://lh6.googleusercontent.com/-vlV-NyCal3o/VVtCGxj3CpI/AAAAAAAAG78/DKWoZHJ3YLs/w945-h697-no/esxi6-sata-ahci-5.PNG" width="100%" height="100%"></center>

### Install ESXi 6.0
請參考 [STEP-BY-STEP 教你如何安裝與升級 ESXI 6.0 / 5.5 / 5.1 / 5.0 伺服器](http://isite.tw/2015/03/19/13091)

唯一畫面會有所不同的是在 "Select a Disk to Install or Upgrade" 會多出一個原先 ESXi 沒支援的 Device
<center><img src="https://lh5.googleusercontent.com/-9IlFi9oEQFY/VVtCGOa3yfI/AAAAAAAAG70/gUAluAL_E2I/w1064-h798-no/esxi6-sata-ahci-3.jpg" width="100%" height="100%"></center>


Reference
=========
- [STEP-BY-STEP 教你如何安裝與升級 ESXI 6.0 / 5.5 / 5.1 / 5.0 伺服器](http://isite.tw/2015/03/19/13091)
- [V-Front Software Depot for VMware ESXi](https://vibsdepot.v-front.de/wiki/index.php/Welcome)
