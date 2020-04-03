layout: post
title: 一名解決方案架構師的自我敏捷
author: Phil Huang
tags: []
categories: []
date: 2020-04-01 00:16:00
---

要來進行今天的分享之前，我得要先來推一下 David Ko 前輩 [David Ko Learning Journey][5]，當初研究所開始接觸 Scurm / Agile 等等的都是主要都是從這位前輩上學習到相關經驗的，直到現在工作一陣子了，觀念還是非常有用，大推

今天要來講一下關於身為一位在 Red Hat 打工了快兩年的解決方案架構師 (Solution Architect) 如何透過敏捷方式提高個人生產力，這篇完完全全就是`個人實際經驗分享` XD

![](/images/trello-2.png)

<!--more-->

## 何謂解決方案架構師?

解決方案架構師 (Solution Architect) 在 Red Hat 定義中是售前職位 (Pre-sales)，主要每日任務簡要版本就是

1. 白天跟著業務跑來跑去開會
2. 晚上寫簡報 / Lab / 回信 / 讀書 / 研究

這個職位有個特性就是，無論來自業務還是客戶，需求都是來得又快又急，很容易一天內被一票不同的事情衝在一起，所以對自己的每月每週每日待辦工作項目掌握要非常清楚，不然很容易會 overloading。而剛好每一季的自我目標管理 (Management By Objectives, MBO)，也需要回顧三個月內做的一些專案和工作負載狀況，故為了提高生產力和準時交付產物，所以...我自己開始了個人敏捷探索

## 常見的個人敏捷方式

Trello 有特別針對`個人`提出了幾種提高生產力的方式 - [以 Trello 提升個人生產力][6]，包含如下 4 種方式:

1. 個人生產力系統：[元老級 Trello 使用者的個人生產力指南][7]
2. 艾森豪矩陣 (Eisenhower Matrix)：相當著名的[工作優先排序法][8]實踐
3. 每週檢討流程：以週為單位 [《搞定！》一書建議養成每週自我檢討的習慣][9]實踐
4. 心智圖：我個人覺得走錯棚

或者是真的走看板方法 (Kanban Method) 的，但是給個人用的

1. [運用個人看板做時間管理 - Ruddy][2]
2. [原來我以為的看板根本不是看板… - Oberon Lai][1]

可能有些人對於看板方法 (Kanban Method) 的定義有所不清楚，這邊強烈建議看一下 [視覺化白板, Proto-Kanban, Kanban 傻傻分不清楚? - David Ko][4]，如同內文所述，只要沒有照著David J Anderson 所提出的以下看板方法實踐走，其實都不算真`看板方法`

- 視覺化 (Visualize)
- 限制同時工作數量 (Limit Work In Progress, WIP)
- 管理工作流程 (Manage flow)
- 為流程訂定明確的方針 (Make policies explicit)
- 實現反饋迴圈 (Implement feedback loops)
- 利用模型和科學方法來協同式改進, 實驗性演進(Improve collaboratively, evolve experimentally)

## 我使用的方式 - 小飛機版視覺化白板

對的，基於對於看板方式的規範，我沒有完全採納該實踐，但從中還是有擷取出 3 個核心概念，從中改成屬於`小飛機版視覺化看板`的樣子

- 視覺化 (Visualize)
- 管理工作流程 (Manage flow)
- 為流程訂定明確的方針 (Make policies explicit)

工具選擇是採用 Trello，主要是簡單、快速和跨平台，當然選擇了好工具不代表專案就能跑得好，只是方便我自己使用而已，[Trello 和看板方法的關係是? - David Ko][3] 一文結語不錯，但我是自己使用，不是跟團隊使用就是了 XD

> 工具不會讓你團隊改善, 唯有團隊具備改善能力, 工具才能改善團隊

可能有人會想推我用矽谷真香的 Notion，我中間有兩個月換成使用 Notion，但使用體驗相當差，網頁開啟反應太慢、有點複雜，最重要是不能離線，我就果斷換回 Trello，另因自己的筆記已經有台灣團隊出品的 [HackMD][13] 及 HappenApps 出品的 Quiver 在使用，我就沒有持續使用 Notion 的理由了

設計上，因為這職業對於時間相當要求，所以視覺化白板設計是以`時間`作為分隔進行流水線設計，範例網址在此 [Trello Template - 小飛機版視覺化看板][15]

![](/images/trello.png)

清單從左至右:
1. Backlog per month：本月收到的任何代辦事項或要求都寫在裡面
2. Todo per Week：這週要處理的待辦事項
3. Do it per Day：今天要處理的待辦事項
4. Ready to go：已處理完的事項，但還沒執行
5. Tracking：已處理完的事項，但還會有延伸需處理之項目
6. Done - Week 1：第一週已處理完的事項
7. Done - Week 2：第二週已處理完的事項
8. Done - Week 3：第三週已處理完的事項
9. Done - Week 4：第四週已處理完的事項
10. Discard：放棄處理的待辦事項

關於卡片排序方式，採用[艾森豪矩陣 (Eisenhower Matrix)][8] 方式，時間先決，故優先處理順序由上至下：

![](https://blog.trello.com/hs-fs/hubfs/Imported_Blog_Media/eisenhower-box2-654x576.jpg?width=654&name=eisenhower-box2-654x576.jpg)

1. 緊急且重要 (Urgent, Important)：一定會列在上去白板上，當前優先處理順序最高
2. 緊急但不重要(Urgent, Not Important)：一定會列上去白板上
3. 不急但重要 (Not Urgent, Important)：一定會列上去白板上，當前優先處理順序最低
4. 不急但不重要 (Not Urgent, Not important)：不列，靠緣分

至於卡片內容的撰寫方式則為

- 抬頭 (Title)：一定是 `執行對象` 和 `要幹麻` 不會有其他多餘的描述
- 截止日期 (Due Date)：要從 Backlog 裡面拉出來，一定要上截止時間
- 描述 (Description)：產出物，可以是簡報網址、網址、文字結論
- 待辦清單 (CheckList)：看複雜度，有時候會用
- 評論 (Comment)：交談後隨手筆記或者是雜資料都可以往裡面塞

而標籤 (Label) 我就比較少用了，沒特別想到怎麼用，而免費版的 Trello 可以選一個 `Power Ups` 使用，這邊是用 `Custom Fields`，可以自己加欄位

## 長期使用下來的缺點?

我這個設計使用大概三年多了，當中的缺點其實也蠻明顯的，前 3 有感如下：

- 對於自身的工作量無法以數字量化，所以工作生活平衡 (Work–life balance) 奇差無比 XD
- 僅能單人使用，只有我自己看得懂在幹嘛
- 不適合用在跑一個中長期專案，或需要定期迭代交付東西的事情，我的理解是改走 Scrum 實踐比較適合，畢竟需要跟其他人協作

雖然我知道有這些缺點，但這個自我設計可以搞定我將近八九成以上的工作型態，我就懶得做成 100% 的樣子

但如果團隊要一起放在同一個板上管理的話，還是先從 Agile 最佳實踐中，先行選出可行的做法後，隨著執行時間越久，慢慢地淬煉出自家團隊的走法是比較有規章的作法


## Red Hat 開源敏捷軟體專案經驗分享?

以下都有公開的資訊可以被 Google 到，只是要知道關鍵字而已 XD

工具選擇上，目前常見走 Kanban 或類似看板 (極難在廣大社群專案上 WIP = =) 的作法，我看到的都是放在 Trello 比較多，而跑 Scrum 的方式則是跑在 Jira 軟體為主的作法居多，少數跑 Kanban 也會放在 Jira 上執行就是了

敏捷方法選擇上，Red Hat 軟體專案管理無論是上中下游大多數都是 `Scrum` 居多，我的觀察是蠻標準的 Scrum 跑法，至於與 Kanban 差異可以看看 [Kanban board 和 Scrum Board 不同之處 - David Ko][16]

## 結語

> 眾里尋他千百度，驀然回首，那妹卻在燈火闌珊處

對，沒錯，我就是沒有遵照最佳實踐，但弄出一套自己運行的很開心的模式，工具在手，揮灑在我。我個人並不是待在軟體開發團隊，而是業務團隊，所以使用敏捷方式會有別於現在大多主流軟體開發的討論情境，各位在思考的時候要留意一下

## References
- [原來我以為的看板根本不是看板… - Oberon Lai][1]
- [運用個人看板做時間管理 - Ruddy][2]
- [Trello 和看板方法的關係是? - David Ko][3]
- [視覺化白板, Proto-Kanban, Kanban 傻傻分不清楚? - David Ko][4]
- [DavidKo Learning Journey][5]
- [看板方法介紹（8）：造物就是造人 - Teddy][10]
- [Kanban: Successful Evolutionary Change for Your Technology Business (English)][12]
- [[工作效率] 決策方法: 艾森豪矩陣(Eisenhower Matrix)][14]
- [Trello Template - 小飛機版視覺化看板][15]
- [Kanban board 和 Scrum Board 不同之處 - David Ko][16]

[1]: https://oberonlai.blog/scrum-kanban/
[2]: https://ruddyblog.wordpress.com/2014/09/21/%e9%81%8b%e7%94%a8%e5%80%8b%e4%ba%ba%e7%9c%8b%e6%9d%bf%e5%81%9a%e6%99%82%e9%96%93%e7%ae%a1%e7%90%86/
[3]: https://kojenchieh.pixnet.net/blog/post/457533491-trello-%E5%92%8C%E7%9C%8B%E6%9D%BF%E6%96%B9%E6%B3%95%E7%9A%84%E9%97%9C%E4%BF%82%E6%98%AF%3F
[4]: https://kojenchieh.pixnet.net/blog/post/470744012-%E8%A6%96%E8%A6%BA%E5%8C%96%E7%99%BD%E6%9D%BF,-proto-kanban,-kanban-%E5%82%BB%E5%82%BB%E5%88%86%E4%B8%8D%E6%B8%85%E6%A5%9A%3F
[5]: https://www.facebook.com/DavidLearningJourney/
[6]: https://trello.com/zh-Hant/teams/personal-productivity
[7]: https://blog.trello.com/work-life-focus-trello-insider-guide-personal-productivity
[8]: https://blog.trello.com/eisenhower-matrix-productivity-tool-trello-board
[9]: https://trello.com/b/O3xTMwoI/weekly-to-dos-review-process
[10]: http://teddy-chen-tw.blogspot.com/2014/08/8.html
[11]: https://en.wikipedia.org/wiki/Kanban_(development)
[12]: https://www.amazon.com/Kanban-Successful-Evolutionary-Technology-Business/dp/0984521402
[13]: https://hackmd.io/
[14]: https://medium.com/@mailtojacklai/%E5%B7%A5%E4%BD%9C%E6%95%88%E7%8E%87-%E6%B1%BA%E7%AD%96%E6%96%B9%E6%B3%95-%E8%89%BE%E6%A3%AE%E8%B1%AA%E7%9F%A9%E9%99%A3-eisenhower-matrix-88bbbf17b454
[15]: https://trello.com/b/tZdYnYrN/%E5%B0%8F%E9%A3%9B%E6%A9%9F%E7%89%88%E8%A6%96%E8%A6%BA%E5%8C%96%E7%9C%8B%E6%9D%BF
[16]: https://kojenchieh.pixnet.net/blog/post/394636169