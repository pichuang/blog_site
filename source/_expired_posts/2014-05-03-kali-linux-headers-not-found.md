---
layout: post
title: 'Kali linux-headers not found '
date: 2014-05-03 23:58
comments: true
categories: 
---
#前言
自從 Backtrack 停止發佈之後, 官方作者另起爐灶弄了 Kali Linux, 這兩套其實差不多, 差別在於前者 based on Ubuntu 後者是 Debian, 且支援的平台越來越豐富了, 詳情請洽 [Kali Linux documentation](http://docs.kali.org/category/introduction), 以下要解決一個問題, 若要將 Kali Linux 裝在 VM 裡 (VirtualBox, VMWare...etc), 因各 VM platform 的 tools 需要用到 linux-headers, 而 Kali Linux 預設的 sources.list 裡未包含, 所以以下是在 Parallels Desktop 下進行的動作.

# 過程
## Add Kali Linux kernel source  
> mv /etc/apt/sources.list /etc/apt/sources.list.bak  
wget https://gist.githubusercontent.com/pichuang/dfde2e34a1f39cadaec7/raw/sources.list  
cat sources.list > /etc/apt/sources.list  
aptitude update  
aptitude install linux-headers-\`uname -r\`  

  - 以上是針對 Kali Linux 未包含 linux-headers 通用解法

## Install Parallels Tools (Only Parallels Desktop need)
> umount /media/cdrom  
mount -o exec /dev/sr0 /media/cdrom  
cd /media/cdrom  
./install  
\#next next next...  
reboot  

## 有圖有真相
<img class="center" src="https://lh3.googleusercontent.com/-S6pUiiZrgCE/U2UQ3ckAfFI/AAAAAAAADU8/N45xxB556c4/s720/%25E8%259E%25A2%25E5%25B9%2595%25E5%25BF%25AB%25E7%2585%25A7%25202014-05-03%252023.52.06.png">
- 從圖可知解析度變得非常好 原先 PD Tools 該支援的功能也都有支援

