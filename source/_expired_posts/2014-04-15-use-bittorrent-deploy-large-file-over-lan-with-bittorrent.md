---
layout: post
title: '利用 BT 在 LAN底下快速部屬大型檔案'
date: 2014-04-15 21:22
comments: true
categories: 
---
# 前言
電腦教室很多時候會遇到要傳送 image 等級的大檔, 如果利用 NFS 掛載方式, 要傳送一份 Image 給一百台電腦, 想當然爾會非常久, 這時候就是 BitTorrent 出場的時候. 有機會再補充內文好了

#  BitTorrent Server 
## 環境介紹
- Ubuntu 12.04 x86 LTS 
  - 若臨時架設 BT Server 傳檔的話, 可考慮使用 LiveUSB 
- Test Filename : papa.testfile (size: 10G)
- Tracker Server ip: 192.168.0.1
- Tracker Server port: 55688

## 安裝 BitTornado
> wget http://download2.bittornado.com/download/BitTornado-0.3.17.tar.gz  
tar zxvf BitTornado-0.3.17.tar.gz && cd BitTornado-CVS  
python setup.py install  
  - [BitTornado 官網](http://www.bittornado.com/)
  - 支援 Linux, BSD, OSX 需要 Python 2.x+

## 產生 .torrent  
> btmakemetafile.py http://192.168.0.1:55688/announce papa.testfile
  - 會獲得一個叫做 ```papa.testfile.torrent``` 檔案, 可rename

## 開啟 Tracker
> bttrack.py --port 55688 --dfile dstat --logfile papa.log
  - 可以在 http://192.168.0.1:55688 看到一些 Tracker 資訊

## 開啟 headless
> btdownloadcurses.py --minport 15926 --maxport 16888 papa.testfile.torrent --saveas papa.testfile

<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/194300/Y8o7grXKTReN2gnRKhoX_btheadless.PNG" alt="btheadless.PNG">

## (Optional) 開啟 HTTP server 讓 Client 下載 Torrent
> python -m SimpleHTTPServer 8000
  - Client 可透過Browser輸入 ```http://192.168.0.1:8000/papa.testfile.torrent``` 來下載 torrent
  - 8000 可以換成任意port

## 使用
- 大家...應該都知道怎麼下載吧
