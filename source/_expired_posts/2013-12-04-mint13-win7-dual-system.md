---
layout: post
title: 'Mint 13 & Win 7 雙系統 grub 問題'
date: 2013-12-04 13:36
comments: true
categories: [Linux, grub, Win7, Mint]
---
# 前言
  以前有安裝過 Mint 9 一次, 用不到兩天就砍了, 印象中原因是將 Mint 9 的 soruce.lst 替換成 Ubuntu 的 source.lst 一更新套件庫就噴一堆版本不符的錯誤, 之後就換回 Ubuntu 10.04 繼續用.

  最近有機會裝到 Linux desktop 再給 Mint 13 機會, 但在最後遇到一個 Grub 的問題

# 問題
安裝完雙系統後, Grub 沒有被正確安裝, 導致會直接進入 Win7 而不會有 Grub 選單
 
# 安裝順序
```
1. Win7
2. Mint 13
```

## Win7
  一般安裝完如果是切C槽D槽的話, 應該會產生三個磁區, 其中 100 Mb 是給 Win7 開機使用, 所以我們應該會有
  1. 開機磁區
  2. C:
  3. D:
  
## Mint 13
  下載 [Mint 13](http://free.nchc.org.tw/linuxmint/isos/stable/13/linuxmint-13-cinnamon-dvd-64bit.iso), 利用 [Universal USB Installer](http://www.pendrivelinux.com/downloads/Universal-USB-Installer/Universal-USB-Installer-1.9.5.1.exe) 製作 LiveUSB, 重開機之後將開機順序設定為 USB 優先, 依照上面指示安裝, 這邊切兩個磁區
  4. swap
  5. /
  
  安裝完後問題來了, Grub 不是有安裝嗎怎麼不見了?
  
# 解決方案
再次使用 LiveUSB 開機, 我們磁區是以下的配置, 可透過 fdisk 觀察
- sda1 開機磁區
- sda2 win7 c:\
- sda3 win7 d:\
- sda4 swap
- sda5 /

打開 terminal
```
sudo su -
mount /dev/sda5 /mnt
grub-install --root-directory=/mnt /dev/sda
reboot
```
這樣就會看到正確的 Grub 選單了

# 結語
基本上其他 Ubunutu 或 Mint 安裝, 如果遇到類似問題, 都可以用這方式解決, 至於為什麼安裝會產生這問題, 這我沒有很細節的去看他安裝過程, 或許有心人可以trace一下.
  
  