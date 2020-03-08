layout: post
title: 成為一名技術留跡者
author: Phil Huang
tags: []
categories: []
date: 2020-03-08 00:16:00
---

小弟 Blog 陸陸續續也從 2013 年寫到現在了，從 `roan.logdown.com` 到現在的 `blog.pichuang.com.tw` 自己寫作風格好像一直以來都是差不多的，這邊分享一下幾個我覺得蠻有用的資訊

<!--more-->

## 我為什麼要寫作?

> 寫作不是為了別人，是為了自己

很簡單，資訊太多，透過文字消化資訊和整理思緒

## 時間花費?

跟看文章完全不一樣，寫文章 `非常花時間`，通常我自己寫一篇都要坐在椅子上要寫上 2 - 3 小時，有些比較複雜的，要截圖翻書和找引用文件，可能 4 - 5 小時都有可能

所以我非常佩服寫書的大大們...

## 推薦使用套路?

- 語法選擇: `Markdown`，基本上沒有什麼懸念，Markdown 文字移植性太好，到哪裡基本上都可以用，但缺點就是對於圖片處理不好，但身為一個大部分都是寫字居多的，沒什麼問題
- 建立靜態網站: 現在要搞一個自己的 Blog 已經不是很麻煩的事情了，也不需要用到 Wordpress / Dropul 這種 CMS 比較大的等級，有個人 GitHub 帳號就可以建立靜態網頁，搭配 [Hexo][3] 或 [Jekyll][2] 或 [Hugo][1] 三個選一個使用，我是用 `Hexo`
- 申請個人域名: `.tw` 結尾的應該不會太貴我記得，我個人是選 `Gandi` 域名商 (感恩 哈維 讚嘆 Haway)，掛載到 Cloudflare 的話就有 SSL 的連線能力囉

## 給工程師看的純技術文件寫作風格選擇?

> 請給我標準的技術文件規範

老實說，這個問題見仁見智，每個人的思緒理解和風格不同，所寫出來的用字和排版也會有很大的差異，但如果目標是要寫標準的技術文件的話，想要了解格式的話，倒是可以看看各大原廠的技術文件規範

[Write the Docs][4] 是一個關心技術文件規範的人所組成的社群，每年會有[Conferences][5]、[Meetups][6]，裡面有不少學習資源，條列幾個比較有名的風格文件給大家參考

1. [Microsoft Writing Stle Guide][8]
2. [IBM DeveloperWorks Style Guide][9]
3. [Red Hat Style Guide][10]
4. [SUSE Documentation Style Guide][11]
5. [Google Developer Documentation Style Guide][12]

但實際上來說，你看完上面的所有文件，大概就是英文作文能力變好，實際上很難完全應用，因為畢竟自己寫文章跟這種大型組織要寫的文件是有點不一樣的，所以以下再提供幾個文件比較落地一點，但因為工作關係，所以會非常偏 Red Hat 的格式

- 如果是要建立`模組化`的技術文件，可以參考 [Red Hat Modular Documentation Reference Guide][20] 所提的方法
- 如果是要建立`與社群合作`建立相關技術文件，可以參考 [Red Hat Community Collaboration Guide][19]
- 如果是想要寫出`口語化`的文章，可以參考 [Red Hat Developer][21]，什麼風格都有，但都是小而美的內容
- 如果只是想要寫出`安裝手冊`的等級，可以參考 [Linuxconfig.org](https://linuxconfig.org/)，目標明確，指示清楚，我最近的寫作風格有受這網站影響 XD

## 有什麼教學網站嗎?

![](https://developers.google.com/tech-writing/images/TechWritingCoursesLogo.png)

> Every engineer is also writer

前陣子 Google 發布了一個 [Technical Writing Courses][12] 網站，主要是英文，分為 2 個課程

1. [基礎技術寫作知識科普](https://developers.google.com/tech-writing/one)
2. [中階技術寫作主題教學](https://developers.google.com/tech-writing/two)

個人是認為這是標準的英文寫作課 XD，對我還蠻有用的，以後可以嘗試寫點英文技術文件下來

除了教學網站，連基礎的技術文件網站模板都 OpenSource 了，可以參考以 `Hugo` 為底的 [google/docsy][18] 專案，此外還弄了個 [Season of Docs][13] 供對於這類技術作家們的開源社群，不得不說 Google 真的是蠻有心的，整套包好好

## 寫作工具?

![](/images/hackmd.png)

這也沒什麼好選的，基本上就是環繞 [HackMD][15] 的工具，HackMD 是我長久使用的 Ｍarkdown 以來最好用的一個平台，我看有些國外會議紀錄也都改用 HackMD 來做共筆

- 線上用 [Hackmd.io][15]
- 線下用 VSCode + [HackMD Markdown VSCode Extension][14]
- Server 上
    - vim
    - VSCode + [Remote - SSH][16]

## 後話

我蠻鼓勵只要是 IT 技術從業人員，無論是在學還是就職者，都應該要花點時間把一些常遇到的技術或者是觀察記錄下來，寫多了其實最大的受益者真的不是別人，是自己。資訊量這麼大的狀況下，培養出個第二大腦是相當重要的，或許不需要像我這樣習慣寫作後公開，你也可以建立自己的私人 Wiki 或者是使用 HackMD 紀錄相關的資訊，好自己做個整理，臨時要用的時候是相當方便的

## References
- [Hugo][1]
- [Jekyll][2]
- [Hexo][3]
- [Write the Docs][4]
- [Write the Docs - Conferences][5]
- [Write the Docs - Meetups][6]
- [google/docsy][18]

[1]: https://gohugo.io/getting-started/quick-start/
[2]: https://jekyllrb.com/
[3]: https://hexo.io/zh-tw/docs/github-pages.html
[4]: http://www.writethedocs.org/
[5]: http://www.writethedocs.org/conf/
[6]: http://www.writethedocs.org/meetups/
[7]: https://www.writethedocs.org/guide/writing/style-guides/#selecting-a-good-style-guide-for-you
[8]: https://docs.microsoft.com/en-us/style-guide/welcome/
[9]: https://www.ibm.com/developerworks/library/styleguidelines/
[10]: https://stylepedia.net/style/
[11]: https://doc.opensuse.org/products/opensuse/Styleguide/opensuse_documentation_styleguide_sd/
[12]: https://developers.google.com/tech-writing
[13]: https://developers.google.com/season-of-docs/docs
[14]: https://marketplace.visualstudio.com/items?itemName=HackMD.vscode-hackmd
[15]: https://hackmd.io/pricing
[16]: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh
[17]: https://linuxconfig.org/
[18]: https://github.com/google/docsy
[19]: https://redhat-documentation.github.io/community-collaboration-guide/
[20]: https://redhat-documentation.github.io/modular-docs/
[21]: https://developers.redhat.com/