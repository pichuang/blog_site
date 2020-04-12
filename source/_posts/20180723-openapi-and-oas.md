---
layout: post
title: 進擊的 OpenAPI!!!
date: 2018-07-23 00:01:26
udpated: 2018-07-23 00:01:26
tags:
  - oas
  - openapi
categories:
  - misc
toc: true
---

## OpenAPI Specification, OAS

隨著資訊服務的飛增及複雜，各家廠商所提供的底層服務封裝後提供出來的介面，我們稱做 API (Application Program Interface)，而 OpenAPI 則是將這些 API 依據某些規範公開出來給第三方使用，可包含免費使用、付費使用或限定使用等方式，有一個公開既定的規範可供參考。

<!--more-->

## 誰是 API 提供者?

以目前看到的案例最大宗的就是 Cloud Service Provider (CSP) 包含像 Google、Amazon、Microsoft 等等，他們的具體做法是將產品功能盡可能的 API 化後提供給開發人員做二次開發，語法規範都個別由 CSP 提供，譬如大家都熟知的 Google Map API。

而第二大類的就屬金融業，各家銀行本質提供的內容都一樣，但服務方式卻大不相同，現在科技進步到使用者習慣大為變化，從過去趕三點半到現在只要網路銀行線上點一點就好，誕生各家傳統銀行對於數位轉型和創新的需求，也就是 Fintech 金融科技或者是 Open Bank，利用數位化銀行業務提供個人或法人更細緻或更廣的需求，進而擴大需求及交易量。所以創新概念及服務推出速度是非常大的關鍵。透過統一語意化的 Open API 可以幫助內部服務相互堆疊組合而成一個創新服務，快速推出產品獲得市場回饋。

最後就是政府相關單位，蠻多先進國家最一開始做的都是 Open Data，開放資料供給一般人做使用分析，但到下一個層次就要思考，要如何提供源源不斷且可以維護的資料來源，則就是 Open API 的事情了。各位可以參考最先起頭的美國的經驗 [2.0美國政府開放資料推動現況與執行經驗][19]。有趣的事情是，目前台灣在這方面的成績是非常顯著地，本文撰寫時間 2018/7 台灣在英國開放知識基金會（Open Knowledge Foundation）中的評比是[第一名][20]。

大家都知道有數據者為大，但數據持有者也需要思考怎麼利用數據提供出創新服務給消費者，進而刺激買單，在市場回饋不明確的狀況下，利用 Agile 敏捷開發流程迭代出新服務測試市場反應是目前常見的模式。

## 台灣政府於 Open API 上的努力

近年政府為了擴大資訊服務的效益，經過資訊政委唐鳳幾次的公開討論 ( [2], [3] )，最終國家發展委員會參考 Linux Foundation 底下專案 Open API Initiative (OAI) 標準定義出 OpenAPI Specification (OAS) 並翻成中文文件，目標是希望統一政府機關的 API 能擁有一致性表述，讓人類和機器都能看得懂，詳細前因後果可以參考[幾個小條文，讓政府提供更好的數位服務][4]。

目前在國發會的策劃之下已經個地方政府已經實作了不少 Open API 放至[政府資料開放平臺][15] 上供社會大眾使用。而想要了解一些國外使用 API 的政府企業案例探討，可以參考前陣子舉辦的 [2018-06-28 Red Hat 數位創新技術座談會 - Saylt][18] 的逐字稿內容，了解國外實際發生的狀況。

對於公部門及廠商來說，有三份文件就變得很重要了

1. [共通性應用程式介面規範 - 國家發展委員會 主管法規查詢系統][5]  

2. [1060911 修正總說明及條文對照表 - 機關委託資訊服務廠商評選及計費辦法][8]  
> 第七條 第十二款 機關採購軟體開發服務，前項第二款所定廠商之專業技術能力，得包括在零成本或低成本之前提下，提供可自由存取、使用、修改及散布之共通性應用程式介面開發或整合能力。  

3. [1051206 資訊服務採購契約範本第 2 條及第 5 條修正草案條文對照表][9]
> 第二條 履約標的 第四款 履約標的涉及應用程式介面開發或增修者，應依國家發展委員會訂頒之最新版「共通性應用程式介面開放規範」辦理，並運用國際通用驗證機制（如 Linux Foundation 之 OpenAPI 等），作為驗收之依據。

透過從制度標準面上，直接調整法規，讓後續的政府資訊系統建置有所法源可以來建立 API 服務，以目前來看這是比較有效驅動政府進行數位轉型的有效方式之一。

## Deep Dive to OpenAPI

### 如何寫出符合 OAS 的文件?

基本上都是遵照 OpenAPI Initiative 所釋出的格式所撰寫，可參考 [OAI/OpenAPI-Specification - GitHub][21]；若是針對台灣公部門所需的規範則可參考 [共通性應用程式介面規範 - 國家發展委員會 主管法規查詢系統][5] 所定義之內容為基準，大量使用案例也可參考 [政府資料開放平臺][15] 放在上面的內容，強烈建議要讀一下。

### 驗證 OAS 格式正確與否

開發者寫出來的 OAS 勢必要經過驗證才知道能不能被正確讀取，[Mermade Swagger 2.0 to OpenAPI 3.0.0 converter][16] 專案就是提供驗證 OAS 2.0 及 3.0 的功能，目前大多數應該都還是在 2.0 的版本比較多，3.0 相信在未來的不久會成為主流。

### Swagger - OpenAPI 文件產生服務

目前常見就兩個 [Swagger][10] 及 [ReDoc][11] 服務，如果想要找更多的工具或網站可以參考 [OpenAPI Tools][12] 內有不少文章。個人使用後心得還是覺得 Swagger 比較通用，主因是網頁易讀及近期開始支援 OAS v3.0，各位可以看到下面有截圖，Swagger 文件有特別標記 `OAS3` 支援。

使用方式很簡單，以 Swagger 為例
1. 點開 [petstore.swagger.io][10]  
2. 將符合 OAS v3.0 規範的檔案放到空白欄位  

```bash
# Fork from https://github.com/OAI/OpenAPI-Specification/tree/master/examples/v3.0

https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/examples/v3.0/petstore.yaml
```

3. 點選 `Explore` 即可獲得一份 OpenAPI 說明文件


### 如何管理這一堆 OpenAPI ?

到這邊為止，各位可以從 Swagger 文件內請求不同的資源獲得對應的資料。然而 OpenAPI 的設計多數直接揭露資料給外部進行存取，倘若系統一多，相互系統呼叫的次數越加頻繁，管理會變成很大的問題，故針對 Open API 的管理上，則會有一 `API Gateway` 的設計，來替內部 Open APIs 進行管理及保護，故會有對應的解決方案來負責提供這方面的保護，例如 Red Hat 3scale，關於基本的 OpenAPI Gateway 相關的設計及功能，之後會特別拉一篇來介紹。


## Reference
- [Open API 應用簡介 - 國家發展委員會][1]
- [2017-03-22 共通性應用程式介面開放規範研議 - Saylt][2]
- [2017-06-23 共通性應用程式介面規範 說明會 - Saylt][3]
- [2018-06-28 Red Hat 數位創新技術座談會 - Saylt][18]
- [幾個小條文，讓政府提供更好的數位服務][4]
- [共通性應用程式介面規範 - 國家發展委員會 主管法規查詢系統][5]
- [RAML 加入 Linux Foundation 的 Open API Initiative][6]
- [Open API Initiative - Linux Foundation][7]
- [機關委託資訊服務廠商評選及計費辦法][8]
- 資訊服務採購契約範本第 2 條及第 5 條修正草案條文對照表 105.12.6
- [政府資料開放平臺][15]
- [3Scale by Red Hat][13]
- [Mermade Swagger 2.0 to OpenAPI 3.0.0 converter][16]
- [2.0美國政府開放資料推動現況與執行經驗][19]
- [Open Knowledge Foundation][20]

[1]: https://www.webguide.nat.gov.tw/News_Content.aspx?n=531&s=1762
[2]: https://sayit.pdis.nat.gov.tw/2017-03-22-%E5%85%B1%E9%80%9A%E6%80%A7%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F%E4%BB%8B%E9%9D%A2%E9%96%8B%E6%94%BE%E8%A6%8F%E7%AF%84%E7%A0%94%E8%AD%B0
[3]: https://sayit.pdis.nat.gov.tw/2017-06-23-%E5%85%B1%E9%80%9A%E6%80%A7%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F%E4%BB%8B%E9%9D%A2%E8%A6%8F%E7%AF%84%E8%AA%AA%E6%98%8E%E6%9C%83
[4]: https://jk.pdis.nat.gov.tw/blog/%E5%B9%BE%E5%80%8B%E5%B0%8F%E6%A2%9D%E6%96%87-%E8%AE%93%E6%94%BF%E5%BA%9C%E6%8F%90%E4%BE%9B%E6%9B%B4%E5%A5%BD%E7%9A%84%E6%95%B8%E4%BD%8D%E6%9C%8D%E5%8B%99/
[5]: https://theme.ndc.gov.tw/lawout/LawContent.aspx?id=GL000270
[6]: https://medium.com/@honglong/raml-%E5%8A%A0%E5%85%A5-linux-foundation-%E7%9A%84-open-api-initiative-f65c5e698d1a
[7]: https://www.openapis.org/
[8]: http://lawweb.pcc.gov.tw/LawContent.aspx?id=FL000677
[10]: https://petstore.swagger.io/
[11]: https://rebilly.github.io/ReDoc/
[12]: http://openapi.tools/
[13]: https://www.3scale.net/
[14]: http://microservices.io/patterns/apigateway.html
[15]: https://data.gov.tw/
[16]: https://mermade.org.uk/openapi-converter
[17]: https://community.apemesh.com/topic/58d722b6e6b006492c2a2a34
[18]: https://sayit.pdis.nat.gov.tw/2018-06-28-red-hat-%E6%95%B8%E4%BD%8D%E5%89%B5%E6%96%B0%E6%8A%80%E8%A1%93%E5%BA%A7%E8%AB%87%E6%9C%83
[19]: https://www.bost.ey.gov.tw/DL.ashx?u=/Upload/UserFiles/%E5%88%86%E4%BA%AB%E6%A1%88%EF%BC%9A2_0%E7%BE%8E%E5%9C%8B%E6%94%BF%E5%BA%9C%E9%96%8B%E6%94%BE%E8%B3%87%E6%96%99%E6%8E%A8%E5%8B%95%E7%8F%BE%E6%B3%81%E8%88%87%E5%9F%B7%E8%A1%8C%E7%B6%93%E9%A9%97_%E4%B8%AD%E6%96%87.pdf
[20]: https://index.okfn.org/place/#table
[21]: https://github.com/OAI/OpenAPI-Specification
