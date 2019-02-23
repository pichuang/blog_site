---
layout: post
title: 'Dir-835 recovery mode'
date: 2014-01-07 19:02
comments: true
categories: 
---
#前言
之前刷 AP Firmware 從未刷掛過, 這次因緣際會之下碰到了把 Dir-835 刷掛的現象, 透過學長的教學, 以此篇文章紀錄.

# 步驟

1. 開啟 Dir-835 recovery mode
  1. 關閉電源
  2. 按住"reset"不放
  3. 開啟電源
  4. 等候五秒 前置面版的LED燈開始閃爍即可放開reset鍵

2. 將網路線連接至lan port上
3. 設定ip為 ```192.168.0.2``` 
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/174691/5Na1rJTrRkeGVB64Xn2q_1.PNG" alt="1.PNG">

4. 開啟瀏覽器輸入 ```http://192.168.0.1``` 會出現 D-Link Router Recovery Mode
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/174691/C2yUciJSuabgrRMLH31Q_2.PNG" alt="2.PNG">

5. 上傳 Firmware及等候
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/174691/MhYBxRxbQ7SxDhv47GEP_3.PNG" alt="3.PNG">

6. 上傳完畢
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/174691/r9uMPWTAqL9ZlPH7t7AF_4.PNG" alt="4.PNG">

7. 將設定ip改為dhcp
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/174691/U67uDsaHT1eK9os5P2Xa_5.PNG" alt="5.PNG">

# 說明
1. Dir-835 內部有一塊 partition 是放置 Recovery mode 的功能, 刷 OpenWrt Firmware 時並不會覆蓋到他.
2. 看到前置面板還在閃燈的時候, 代表它還在開機中.
3. Dir-835 的 Recovery mode 時用的router ip為 ```192.168.0.1```, 預設不開dhcp, 需手動輸入.
4. 如是刷成 OpenWrt後, 預設router ip會變成 ```192.168.1.1```, 需特別注意.

# 結語
一開始被 default router ip 搞很久, 一直沒有意識到他的ip可能不是 192.168.1.1, 後來經高人指點後, 即順利解決問題.