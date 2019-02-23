---
layout: post
title: 'Windows 7 USB Download tool'
date: 2014-07-12 13:13
comments: true
tags:
- infra
- win7
---

現在製作 win7 USB 安裝碟變得十分容易, 只要下載 [Windows 7 USB Download Tool](http://images2.store.microsoft.com/prod/clustera/framework/w7udt/1.0/en-us/Windows7-USB-DVD-tool.exe) 把 iso 放進去即可

但很多時候會遇到一個問題 `We were unable to copy your files. Please check your USB device and the selected ISO file and try again.` 此時就需要對 USB 做 Format 的動作

<!--more-->

## 過程
1. 開啟 cmd
2. Format disk
```bash
diskpart
list disk //找尋你的 USB 位置, 假設disk 1為 USB  
select disk 1  
clean  
create partition primary  
active  
format quick fs=fat32  
assign  
exit
```

3. 開啟 Windows 7 USB Download Tool 即可

