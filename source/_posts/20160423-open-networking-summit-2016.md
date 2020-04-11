---
layout: post
title: "ONS 2016 之旅"
description: "Open Networking Summit 大進擊"
date: 2016-04-23 00:00:00 +0800
updated: 2016-04-23 00:00:00 +0800
tags:
  - event
  - ons
categories:
  - misc
---

### 前言

在 2016/2 - 2016/3 的時候，我接到了一個頗重的任務，目標是趕在 [ONS 2016](http://events.linuxfoundation.org/events/open-networking-summit) 要把 [ON.LAB](http://onlab.us) 所主導 SDN Controller 專案 [ONOS](http://onosproject.org) 當中的一個 Use Case  [SDN-IP](https://wiki.onosproject.org/display/ONOS/SDN-IP) 於台灣交通大學建立起來，並跟國際 REN (Research and Education Network) 連接在一起，專案執行時間不到一個月。

經過連續三週沒日沒夜各種[爆肝](https://goo.gl/photos/HLnD84dBMPqkh5ap8)及近乎 24HR 的資訊轟炸後，終於順利完成，可以去美國玩啦！

### 關於 SDN-IP Project in NCTU, Taiwan
若對此專案介紹有興趣的話可以參考如下資訊：

- [Video of Global SDN-IP Deployment at NCTU, Taiwan](https://youtu.be/a8LR1DyzGY4 )
- [Slide of Global SDN-IP Deployment at NCTU, Taiwan](https://url.fit/ZmrEK)
- [交通大學新聞稿](http://www.nctu.edu.tw/component/k2/item/1619-ons-2016)

之後有講到更詳細的部分會再將資訊補上來，這邊不琢磨太多細節。

### 出發前

因為案子是趕在出國前一週才完成的，所以在那之前有非常多的事情都是麻煩我同學及學長幫忙處理，譬如訂機票、飯店、行程...等，老實說我直到前一週才知道要去 "San Francisco" 這個地方 XD，那時候我連他在東岸還是西岸還是在美國中部都不知道。

在出國前三天之前，我跟 Jserv 及 Powen Tsai  吃飯，有談到如何宣傳此事，因為當時完全不清楚整個會議流程是怎樣的 (可能是英文太爛，沒問對問題)，所以先預設現場需要講，加急寫了中英逐字稿跟錄音稿，直到飛機在跑道上的時候，才把錄音檔完整的下載到我的筆電上。這邊要特別感謝一下團團、張欣、郭靖三位超強後勤 XD。

### 攤位 Booth

Global Deployment ONOS SDN-IP 計畫是由 ON.Lab 所主導，而全球部署負責人為 Luca Prete，我們的專案建立成果就是透過他所呈現的。對於 Luca 來說，整個 Demo Story，除了要展現全球採用 ONOS SDN-IP 部屬的情況以外，同時指定台灣交通大學當做故事核心，反覆地利用 SDN-IP 的特性展示 BGP Redundant 效果，讓會眾能充分地感受到實際使用的狀況。

下圖是這次交大去美國參與 ONS 2016 的長官及同學和 Luca Prete 的合照。  
<center><img src="https://lh3.googleusercontent.com/Kt_vBeDBxLYF_Gfk_PujHAhDUb7mkffMKcJDrIevBmb8u9DYDeWLMi32xHRaczyjI53LzIu8dmIQSSjrms1BvgXtvhNHRI-Y5QepWhoEsuJW9Kth3nAVhK4aJ_BWnguw-V4DnMT2tudw0QLJ1budZguRSMex6PDkc7tFGkPEQUvFGVxLdFOs7LW5fmsln3tzucQvLwatcoftoOMhyzsEHB6ofcNowbzDwYvIQG6k9mAjjmF3lwBqAv3PEK4YYg_7vYfjT-bMD7Gf5wkbpoNyfvvUz4mJdng77azOI-imvEz-E_7EwIONyen0lu_NdS-qOpCUzAMjxyxQBfqOYbPnu1Hi6N-DzCQH-YCH-ncR9Mg48y22m_cWTNItzv7J5aehLBfhi3J5bgyNdKl-q9XnIej0lUauDnniTlHmMy16UUTLgPIRFMcthaQpP0xcTb_DbEEjeuHOyvmNWopuopUSQ0vcXAXpq6KwvIwx8HLHrqwtISvjJsEgKFtb4TMR-cAjBlL4uqvs8CJS11i4aCMAU6IRTMPUcgXvRagNMzegE6tpCA9_nVCO8YEf75Jikg4XLqxe=w2560-h1440-no" width="50%" height="50%"></center>

### 會議 Event
我這次去有參加兩個會議，除了 ONS 2016 以外，還有 [DevOps Network Forum 2016](http://events.linuxfoundation.org/events/devops-networking-forum)，後者是一天型會議

#### Open Networking Summit 2016
去美國之前，外星人等級之強者我學長 [Charles Chan](https://github.com/rascov) 告訴我們說 "大部分的 ONS 會議內容都會放在網路上播放，建議實際到攤位多看及多跟其他人交流"，所以參加的期間除了 Keynote 都有聽以外，大部分時間都在逛攤位，跟各路廠商交流意見，譬如像是第一天我就跑去找 [OpenSwitch](http://www.openswitch.net) 的人詢問:

我: 請教一下你們現在哪個版本號是可以正常運行在  Edge-Core AS5712 上?  
美國大叔: 哦\~現在還不穩定啦\~大概要到六月的時候才會有一個正式的穩定版出來\~  
我: = = 可是你們 xxx bug 已經起碼壞掉兩個月了耶... 那你們現在有幾個人在開發?  
美國大叔: 哦\~我們有兩千多人在開發啦\~我們都在生 Feature，Bug 會晚一點再修  
我: ...ok....  
美國大叔: 誒\~我們有一個 VR 的專案現場展示中\~要不要玩玩啊? 很好玩的喔!!  
我: (OS: 你們是來展示 Toy Porject 嗎? XDDD)

我們可以從以上對話得知，好的情報可以讓你省下很多時間 (因為當初專案中的 L2 ToR Switch 預設要使用 OpenSwitch ，但不論我怎麼弄都有問題，而當中足足燒掉我兩天的時間，最後不得已放棄，改採購 Legacy L2 Switch)

#### DevOps Network Forum 2016

<center><img src="https://lh3.googleusercontent.com/UtU-Lx4jwZeN6nSvmFkauVv3H8tL2B-DqjgszAKQT_LtXeiSCpKuQQkOa3W0LcmlW9b0bGhTEgTB9KM6Dogz5vHKKV2iCfdeRbSuswIEERwExHt-k4NaCKkIiZ_04Bld4Mwx0HoXFCBwg8zOql2XlSw-_-k2AWu_i5wcRcBOJCF8naGug1_Z2QMS__t-KwzUd0xXAK5ioIGP56J3-7xemC-9tOU-NYMa5BdXu7WvLIM0uMvtDvhFMop3WRN8S8xfdDDqj8TcHcbwB1Ci7Y1iKgPPro3p-dvVQ1SOgzbHJgSsQMbjxXa9L6UZaHBPPt_YQn-9P1LOZhPNzrrAaDMQn_lkkfy1no1F0UxpkJbCW-cedpHkoWOr1qmHBze8LzetUE2ReaD0q7AsXR4FyyKQmjIOCD8xrcHKXef806Yn6-ItTQro0jaeGePvgvxrfXKt8h2hY5PWdQ9u9PNRUyJKscrfBcQ2tcmdNArySoZAIpWZgTnpEIzxrGgbOqqINnFIzyVwGqMiCBRUoEc6-ZpxeaE7Z7G2E5m1pnxuYVwANMBmu3ZvTQt7ZWv8b8sJdNIKs6J5=w832-h1478-no" width="30%" height="30%"></center><br>

這個會議是我直到會議前一天我才報名的，需要多加一百塊美元。而這會議也有將所有的影片都放出來，個人建議可以參考以下三個影片:

- Test-Driven Network Automation
- What is NetDevOps? Why?
- Move Fast, Unbreak Things!

在外國來講 DevOps 概念已經被宣導很久了，而台灣大概是前年開始才有聽到比較多的討論，而我個人在 "SDN-IP Project in NCTU, Taiwan" 裡面有採用 Ansible 來降低大量且單調的網路測試時間，因現在的 SDN Network OS 多數都是具有 Python 及 SSH Daemon，可以非常簡單地導入 Ansible Ad-Hoc 機制來管理 Switch，讓管理人員盡量專注在 "要做什麼 What to do"，而不是 "要怎麼做 How to do" 目標導向為主的使用上，期望有一天在台灣也有人可以分享針對 DevOps 概念於 Network 管理上的演講。

### 旅遊 Travel

這是我第一次到美國，很多事情都非常新鮮，以下列舉一下

#### 出海關

<center><img src="https://lh3.googleusercontent.com/It0GuIwVVa655M4iXDthPJ11WbWwf3p2dOmmathhsbnBZGyUjnuoh57P22aiAxMq8t2dSOLTbgNka-6gfeeCpdzmWhN5-mcpSyTay0bZqfOJ5LR9A5Z4CDPD3VKUxdMWuY58jIiHn1Ky953cxBnM3L8zBwsh9N1Q-9ZzuLujqeH94BoPPMKJSwxrDrKuU93PJFouubKeOOARgPbgOk-RCkOJ5hGxCo4s7oAGnF_5g9gYMWx3t9X8P_FY8e7zjGDftZELR2SyYajFD38Wc_sNbyPeTXwmseuB9PKwBbcSfMnx6EiPIoKryBZT9bGkKVAvrY-XWBCgPQ6w2jWZYPq5SZrkWH12vqfuaZ4CH35beJxTEPk2nD-z1JRZKtTcPgz2STmXYrRrj0tmKsRtE-rZRJsu7nibDUeGv-iqyGmDZn19GmCQIgfU7vOfaLZZU65okVTU24oH4Nt9g6LsNO86qzzywBDJoTGZYMq8I-HCm_sxVGOJAshsccBHNtFi-ab7BpPz-wwkSVa9Os7t_FvLkrqziqBIpCEAWADGK02SrMp6QnEZnRWVffxRE141u0nneUPR=w2560-h1440-no" width="50%" height="50%"></center><br>

我去之前就知道，美國海關不太好搞，而且我英文頗爛，長相一點都不友善，有點擔心因為講錯被抓去小房間問話。實際到了現場，海關問了我一系列的問題: 你家住哪裡? 你什麼學位? 來這邊幹嘛? 待幾天? 然後他多問了兩個問題聽不太懂，我就開始亂唬爛了 XD，也就是答不對題。對方就開始大笑，最後問了我兩個我印象超深的問題

- 你的包包有放米嗎? Do you have rice in your vag?
- 你叫什麼名字? What's your name?

因為被一連串問下來有點閃神，聽到這問題那瞬間有點傻眼，還傻傻地跟他說 "Yes, I have one." ；而後者是我講錯名字，我回他英文名 (Phil Huang)，但護照是寫我中翻英名字 (Ping-Chun Huang)，所以又重講了一次，回想起來都覺得好笑 XD

#### 吃飯

我只能說美國太恐怖了，天天吃一堆冷食快死掉了。回國前一天，大學及高中學妹牛牛剛好在 San Jose 工作，所以中午她開車跑來 San Clara 的會議飯店找我去吃雞腿飯時，都快掉淚了 QQ

從物價上來講，每一餐 10 美元到 15 美元都是正常的，所以吃飯時如果腦袋一直將金額換算成台幣的話，大概沒有一餐吃得下，何況如果是有服務人員服務的話，每餐都需要給 10% - 15% 的服務費。幸好這次去大會有提供早午餐的  Buffet，所以整體上省了不少錢。

### 結語
這次出國要十分感謝台灣交通大學及 [開放文化基金會](http://ocf.tw) 之 [國際交流計劃](http://ocf.tw/donate/intl.html) 的贊助，讓我有這個機會可以到美國去看看，雖然很多事很難用文字完全描述出來，但對我個人的收穫來講是十分的多，也因為這個出國的機會更加深我三十歲前想要去美國工作的想法，讓人生階段性計畫也隨之改變。

### Reference
- [ONOS Lightning Talk: Global SDN deployment powered by ONOS](https://youtu.be/orI2FtyxN1I) by Luca Prete, ON.Lab
- [Open Networking Summit (ONS) 2016 Recorded Session](https://url.fit/ZtN68)
- [Open Networking Summit (ONS) 2016 Session Slides](https://url.fit/ZwNKV)
- [Enjoy All 13 Videos from DevOps Networking Forum](https://www.linux.com/watch-devops-summit-2016-videos)

