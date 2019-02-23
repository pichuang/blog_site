---
layout: post
title: 'MoMorc 自動安裝 rc config '
date: 2014-07-06 23:23
comments: true
categories: 
---
因為平時經常安裝虛擬機, *rc 經常cp來cp去的覺得不方便, 而且缺乏一個統一控管的方式, 這時候就用 python3 土砲出 [MoMorc](https://github.com/pichuang/momorc) 這小 project

## 特色
* 利用 Git 做個人化 *rc 的統一控管
* 一鍵安裝
* 利用 Symbolic Soft Link 做安裝
* 維護方便, 只需更新 MoMorc 底下的檔案即可
* Support python3

## 安裝
```
git clone https://github.com/pichuang/momorc.git .momorc
cd .momorc

//Modify or Add your rc.conf

chmod +x setup.py
./setup.py
```

## Debug mode
```
./setup.py debug
```

## Reference
- [MoMorc](https://github.com/pichuang/momorcrc)
