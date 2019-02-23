---
layout: post
title: 'Ubuntu 12.04 LTS Kernel 降級'
date: 2014-03-03 03:34
comments: true
categories: 
---
# 前言
前陣子 Ubuntu 12.04.4 出了 kernel version 為 3.11.0-15, 而 OpenvSwitch 2.0.0 不支援太新的版本, 會噴出 
```configure: error: Linux kernel in /lib/modules/3.11.0-15-generic/build is version 3.11.15,  
but version newer than 3.10.x is not supported```的錯誤, 所以只能將 kernel downgrade 至 < 3.10.x

# 環境
- OS: Ubuntu 12.04 Server LTS X86_64 
- Kernel current version:  3.11.0-15-generic
- Kernel downgrade version: 3.8.0-36-generic
  
# 安裝
## 安裝套件
> aptitude install linux-image-3.8.0-36-generic linux-headers-3.8.0-36-generic

* 如果有錯誤的話, 可利用 ```aptitude search linux-image-3.8.0``` 找一下正確的版本號

## 移除舊 kernel 
> aptitude remove linux-image-3.11.0-15-generic linux-headers-3.11.0-15-generic

* 系統會自行跑 update-grub 自動建立好 /boot/grub/grub.cfg

## 重啟
> reboot

