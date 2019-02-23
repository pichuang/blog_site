---
layout: post
title: 'ArchLinux 與它的 ABS'
date: 2014-07-03 09:37
comments: true
categories: 
---
ArchLinux 管理套件的方式是利用 ```pacman``` 來進行管理, 通常我們下載下來的時候都是編譯好的 binary code, 副檔名為```.pkg.tar.gz```, 位置在 ```/var/cache/pacman/pkg/```, 直接進行安裝, 省略掉編譯的過程. 這樣做除了在大量部屬的時候可以確保全部電腦的一致性, 也可以降低不少手動編譯出錯率. 

但有時候 System Administrator 認為有些套件不符自己的需求想要客製化編譯, 這時候 abs ([Arch Build System](https://wiki.archlinux.org/index.php/Arch_Build_System)) 就派上用場了. abs 是類似於 ports 的一種系統, 可以幫助管理者從 source code 編譯後打包成 packages, 進行管理.

## 安裝
> pacman -S abs 

## 使用 ( 以htop為例 )

1. 尋找 Repository
> pacman -Si htop |grep Repository
 * 我們可以知道 htop 是在 extra 裡面
 * 若是在 aur 的話, 只能透過做尋找 PKGBUILD 的動作 https://aur.archlinux.org/

2. 下載 PKGBUILD
> abs extra/htop
 * 從 ```/var/abs/extra/htop``` 我們可以看到 htop 的 PKGBUILD
 * 如果看不到的話, 要去 ```/etc/abs.conf``` 在 REPOS 把 extra 加進去
 
3. 搬動 htop PKGBUILD 
> cp -r /var/abs/extra/htop /tmp/ && cd /tmp/htop
 * 這是怕當 pacman 有新更新的時候, 原先寫在裡面的 PKGBUILD 會被覆寫

4. Hack htop PKGBUILD
> vim PKGBUILD
 * 這邊就是看個人要怎麼做, 此篇範例採更動 pkgrel
 * 如果有動到 source 本身的檔案, 需要下 ```updpkgsums``` 來更新 checksums

5. 編譯 PKGBUILD
> makepkg -s
 * makepkg 會自動處理相依性問題, 且會將 Source code 下載下來編譯後打包成 ```htop-1.0.3-3-x86_64.pkg.tar.xz```
 * 這邊盡量不要使用root做編譯, 如果 PKGBUILD 內有 ```rm -rf /```, 會真的執行下去
 * 如果已經編譯過了, 還想要再編譯一次的話, 可以下 ```makepkg -f```

6. 安裝 htop-1.0.3-3-x86_64.pkg.tar.xz
> pacman -U htop-1.0.3-3-x86_64.pkg.tar.xz

7. (Optional) 如果有自己的 Repository 想要放進去更新的話
> repo-add xxx.db.tar.gz htop-1.0.3-3-x86_64.pkg.tar.xz
 * 這只有對有自行建立 Repository 的管理者才有幫助, 關於詳細內容可以參考 [建立 Private ArchLinux User Repository](http://roan.logdown.com/posts/208168-establishment-of-private-archlinux-user-repository)



 
 
