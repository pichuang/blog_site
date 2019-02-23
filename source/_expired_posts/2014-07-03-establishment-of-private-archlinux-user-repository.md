---
layout: post
title: '建立 Private ArchLinux User Repository'
date: 2014-07-03 11:09
comments: true
categories: 
---
各公司所需要的套件不盡相同, 可能會依據一些特性做 PKGBUILD hacking, 但這些編譯完之後如果要傳到各個工作站, deploy非常麻煩, 所以這時候就需要建立 Private ArchLinux User Repository (pAUR), 讓工作站只要在 ```/etc/pacman.conf``` 新增 Repository 就可以統一管理兼使用

# 建立 Private ArchLinux User Repository (name: roan-aur)
### NGINX

* 安裝 nginx
> pacman -S nginx

* vim /etc/nginx/nginx.conf
> ...  
location /{  
    root   /srv/http;  
    index  index.html;  
    autoindex on;  
    //allow xxx.xxx.xxx.xxx/24;  
    //allow xxx.xxx.xxx.xxx;  
    //deby all;  
}  
...  

 * Comment out 是可以限定ip存取, 不要讓外面的人來拉檔, 可以自行定義

* Start and enable
> systemctl start nginx && systemctl enable nginx

### pAUR db

* 建立
> mkdir -p /srv/http/roan-aur/any/ && /srv/http/roan-aur/any/  
cp *.pkg.tar.gz /srv/http/roan-aur/any/  
repo-add roan-aur.db.tar.gz htop-1.0.3-3-x86_64.pkg.tar.xz  

 * 這邊是用 htop-1.0.3-3-x86_64.pkg.tar.xz 做舉例, repo-add 會自動幫你更新 roan-aur.db.tar.gz

* 截圖
 <img class="center" src="https://lh5.googleusercontent.com/-qHuKTP8ixzw/U7TSNkniDRI/AAAAAAAAE2A/VAGgn7cplfo/w1916-h394-no/%25E8%259E%25A2%25E5%25B9%2595%25E5%25BF%25AB%25E7%2585%25A7+2014-07-03+11.46.26.png">


# 工作站設定
* vim /etc/pacman.conf
> ...  
[roan-aur]  
SigLevel = Never  
Server = http://xxx.xxx.xxx.xxx/$repo/any  
...  

* update
> pacman -Syu

Reference
- [pacman tips](https://wiki.archlinux.org/index.php/Pacman_tips)
- [自行製作PKGBUILD](http://roan.logdown.com/posts/208154-archlinux-with-his-abs)
