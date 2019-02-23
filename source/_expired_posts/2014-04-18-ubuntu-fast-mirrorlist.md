---
layout: post
title: 'Ubuntu Fast Mirrorlist'
date: 2014-04-18 03:35
comments: true
categories: 
---
一般我們去下載回家 Ubuntu iso 裡預設 Ubuntu mirror site 應該都是 ```us.archive.ubuntu.com``` 這 mirror 在國外, 所以下載更新檔這種速度會非常慢, 這時候換個 mirror 可以節省很多下載時間.

> cat /etc/apt/source.list

以下推薦兩個 mirror

1. ubuntu.cs.nctu.edu.tw
2. free.nchc.org.tw/ubuntu/

前者是交大資工系計中 mirror 學網內較為推薦使用, 如果有人覺得速度還是不太行可以考慮國家高速網路與計算中心, 這 mirror 站大學時期用很久, 速度非常好.

> cp /etc/apt/sources.list /etc/apt/sources.list.bak  
sed -i 's/us.archive.ubuntu.com/ubuntu.cs.nctu.edu.tw/g' /etc/apt/sources.list  
aptitude update  

如果都認為速度難看的話, 那還能透過網路分析選擇

1. softwareproperties (Ubuntu Desktop only) 
> sudo /usr/bin/software-properties-gtk
  - Download from > Others > Select Best Server 
  - 有興趣的人可以研究 [python-software-properties 套件文件清單](http://packages.ubuntu.com/saucy-updates/all/python-software-properties/filelist)
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/194634/AlnmT5NZRBe34RwtiMYl_software-properties-gtk.PNG" alt="software-properties-gtk.PNG">
  

2. getfastmirror (Ubutnu Desktop/Server)
  - Source: [hychen/getfastmirror](https://github.com/hychen/getfastmirror) 
  - 透過 CLI 下去跑該 Server 較為適當的 mirror site
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/194634/w6pQHCkARQOu0eLn1lMn_getfastmirror.PNG" alt="getfastmirror.PNG">

3. [Official Archive Mirrors for Ubuntu](https://launchpad.net/ubuntu/+archivemirrors) 自己選


小記: 不知道為什麼官方一直都不出 software-properties-cli



