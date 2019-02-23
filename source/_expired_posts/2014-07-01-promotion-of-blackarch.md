---
layout: post
title: '推廣 BlackArch'
date: 2014-07-01 22:14
comments: true
categories: 
---
<img class="center" src="http://www.blackarch.org/logo.png">


# 介紹
[BlackArch](http://www.blackarch.org/) 是由 [nullsecurity team](http://nullsecurity.net/index.html) 這個組織基於 [ArchLinux](https://www.archlinux.org/) 整理各式 [security tools](http://www.blackarch.org/tools.html) 而成的一套 distribution. 提供的平台有 x86_64 i686 及 arm

BlackArch 的 issue tracker 是利用 [GitHub](https://github.com/BlackArch/blackarch/issues) 作管理, 如果有問題也可以加入IRC: ```#blackarch``` 在上面作詢問, 回覆速度都很快, 而且大部分都是主要維護者在幫忙回覆.


# 下載 iso
* [交大資工系計中 BlackArch mirror](http://blackarch.cs.nctu.edu.tw/iso/)
 * 目前最新版是 2014.07.01
* BlackArch 預設帳密為 root/blackarch

# 使用
## 已是 ArchLinux 用戶
可直接透過新增 pacman 的方式做使用, 不用特別更換 os

1. curl -s http://blackarch.org/strap.sh | sudo sh
2. vim /etc/pacman.conf
```
[blackarch]
Server = http://blackarch.cs.nctu.edu.tw/$repo/os/$arch
```
3. pacman -Syyu

## 若是使用 iso 用戶
可以更換 mirror 拉檔較為快速

1. sed -i 's/www.blackarch.org\/blackarch/blackarch.cs.nctu.edu.tw/g' /etc/pacman.conf
2. pacman -Syu

## 關於 PKGBUILD
既然是 Based on Archlinux 當然會有 PKGBUILD 可以看, BlackArch packages PKGBUILD 放置在 [github](https://github.com/BlackArch/blackarch/tree/master/packages) 上做管理

我們可以利用 Arch Build System (abs) 作下載及更改 PKGBUILD 自行 ```makepkg```, 若是想要對 BlackArch 作點 contribution 的話, 可以參考[這個](http://www.blackarch.org/guide.html#SECTION00400000000000000000)

## More...
* 如果還要更多詳細的操作, 可以參考 [BlackArch Guide](http://www.blackarch.org/guide.html)