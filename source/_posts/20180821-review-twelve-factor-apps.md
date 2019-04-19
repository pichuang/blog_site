layout: post
title: 再讀 Twelve-Factor Apps
author: Phil Huang
tags:
  - openshift
  - container
  - 12apps
categories:
  - openshift
date: 2018-08-21 15:00:00
---

## 前言
前陣子因為各種原因，被問及要如何在大型軟體開發專案中，在容器平台上有效避免程式發生相依性錯誤 (Dependency Hell)? 這問題的其實可以轉換成，有沒有一個容器構建規範是可以讓開發者可以參考的開發指南？

其實早在好幾年前 SaaS 服務盛行時，Heroku 這公司就汲取了大量 Software-as-a-Service 的經驗，撰寫出 [The 12-Factor Apps][1] 供開發者參考，幾年後 PaaS 服務盛行， Red Hat 也基於該原則之上，提出了額外考量點出來，供 OpenShift/k8s 的開發者設計參考，因 [Optimizing Twelve-Factor Apps for OpenShift by Red Hat][3]並沒有寫得如同 12-factor app 詳細，故僅列出並帶點闡述。

<!--more-->

## Twelve-Factor Apps

筆者先建議看原網站的解釋 [The Twelve-Factor Apps][1] 或二次解釋的中文版網站 [十二因子應用程式][7]，臨時複習的時候再看以下的小抄 XD。

### 簡介

以下從[The Twelve-Factor Apps][1]直譯

如今，軟件通常會作為一種服務來交付，它們被稱為網絡應用程序，或軟件即服務（Software-as-a-Service, SaaS）。12-Factor Apps 為構建如下的SaaS應用提供了方法論：

  - 使用**標準化**流程自動配置，從而使新的開發者花費最少的學習成本加入這個項目。
  - 和操作系統之間盡可能的**劃清界限**，在各個系統中提供**最大的可移植性**。
  - 適合**部署**在現代的**雲計算平台**，從而在服務器和系統管理方面節省資源。
  - 將開發環境和生產環境的**差異降至最低**，並使用**持續交付**實施敏捷開發。
  - 可以在工具，架構和開發流程不發生明顯變化的前提下實現**水平擴展**。

這套理論適用於任意語言和後端服務（數據庫，消息隊列，緩存等）開發的應用程序。

### 1. Codebase 基礎代碼

One codebase tracked in revision control, many deploys

  - 每一個 Application 只能有一個 Codebase，必須要使用版本控制工具 (e.g Git / SVN) 進行管理
  - 可以部署不同版本的 Applications，但要基於同一份 Codebase
  - 多個應用不能共享同一個 Codebase，需先拆分後成個別獨立 Codebase，使用依賴管理進行載入
  - OpenShift 可以支援從 Codebase 直接編譯及部署應用程式

### 2. Dependencies 相依性
Explicitly declare and isolate dependencies

  - 採用**顯式**聲明依賴關係，不使用隱式，明白表示所有需要依賴的函式庫
  - 一個正確的應用程式不應該依賴系統層級的函式庫，不應讓專案影響或被影響系統函式庫，兩邊需隔離使用
  - 不隱式依賴系統工具，譬如說 wget 等常見工具，有需要用到就要打包進去，以符合顯式聲明
  - 可採用語言依賴關係管理系統，如 Maven、Gem、CPAN、pip

### 3. Config 設定檔
Store config in the environment

  - Source code 和 Configuration 須嚴格分離，因不同環境會有不同的環境配置檔，需要拆開管理 (e.g SIT/UAT/Production)，不應環境不同而反覆修改 Source code
  - 設定檔不應包含機敏資料

### 4. Backing Services 後端服務
Treat backing services as attached resources

  - 不區分本地或第三方服務，對每個服務來說，其他服務都是該服務的 backend services
  - 保持服務間為 Decouple 狀態

### 5. Build, Release, Run
Strickly separate build and run stages

  - 嚴格區分部署三階段
    1. Build: 將 Source code 轉換成可執行的二進位檔，編譯時會採用特定的版本和依賴函式庫
    2. Release: 將 Build 的產出物跟當前部署的環境設定檔結合
    3. Run: 針對特定發布版本，在特定環境中啟動執行
  - Release 一定要有特定的唯一辨識編號，方便有問題可以 rollback，關於軟體編號法可以參考[軟體版本號 - Wiki][6]
  - 盡量採用自動觸發 build code 流程，減少採用人工操作

### 6. Processes 程序
Execute the app as one or more stateless processes

  - 應用程式應為 **無狀態 Stateless** 和 **無共享 share-nothing**，需要長期保存的資料可以放在 Backing Services (e.g. Persistent Volume / Database ...etc)
  - 不採用任何 Sticky session 實作方式，應將 Session 保存在 Redis 或 Memcached 等 Cache server 中

### 7. Port binding
Export services via port binding

  - 應用程式應可完全自我啟動以 TCP/UDP 為主的對外服務
  - 需要能綁定特定埠口，其他服務皆可以對此送出請求

### 8. Concurrency 併行
Scale out via the process model

  - Processes 是開發人員可以操作的最小單位
  - 因 Processes 所帶有的特性，故於系統急須大規模擴展的時候將會十分容易進行 **水平擴展 Scale-out**，易於分散到各系統上執行
  - 程序本身不需要有 Daemon 的存在，應該借助外部的進程管理系統來管理程序

### 9. Disposability 易處理
Maximize robustness with fast startup and graceful shutdown

  - Processes 應該是易處理的，應具備可以瞬間開啟或停止的能力，有例如快速、彈性的水平擴展應用
  - 追求最小的啟動時間
  - 接收 SIGTERM 會優雅的終止程序，意指是不接收新的請求和會等當前正在處理的 Session 被關閉後，才會退出

### 10. Dev/prod parity 開發環境及生產環境等價
Keep development, staging, and production as similar as possible

  - 為了要做到持續部署 (CI/CD)，故需縮小以下三個差異
    1. 時間差異，盡可能在幾小時或幾分鐘就部署一次
	2. 人員差異，不應該僅寫代碼，更應該密且參與部署過程及上線後的狀態
	3. 工具差異，盡可能保持開發及線上環境的一致
  - 不應在不同環境中使用不同的後端服務，不兼容所帶來的代價會十分高昂
  - Docker / Vagrant + Ansible 是你的好朋友

### 11. Logs
Treat logs as event streams

  - 程序不應該煩惱 logs 要存在哪裡，統一採用 stdout 的方式直接輸出，產生 Event Streams
  - 這類 Event Streams 若要儲存則建議利用 Fluented 等工具去擷取後，透過 Elasticsearch 分析及 Kinbana 呈現結果

### 12. Admin processes 一次性管理程序
Run admin/management tasks as one-off processes

  - 一次性管理程式應該和正常的程式使用同樣的環境
  - 一次性管理程式也是 Source code，故也須遵循 Codebase 原則，上版控

## 額外可考量因素

### 1. Microservices 微服務
  - 以業務功能為主的設計概念，為最小單位
  - 每個服務皆有自主運行的業務功能，不應受語言限制

### 2. Self Service Full Stack Infrastrcuture 全棧式自助服務
  - 提供自助服務，供開發人員自行點選所需的資源，降低需求反應時間
  - 維運人員則需透過 RBAC 管理資源分配及提供其相關的基礎建設設定

### 3. Support your choice of deployment 支援跨平台部署
  - 容器平台應要能支援部署在不同雲環境或基礎平台之上，而不限定於某一廠商
  - 雞蛋不要放在同一個籠子

### 4. Enterprise Ready Containers 採用企業級容器平台
  - 避免後續維護時產生的延伸問題，專注於創造服務
  - 一開始初始安裝都是小事，維護系統才是大事，這跟買車的道理一樣，買車錢小，養車花大

### 5. API-Based Collaboration 基於 API 的協作
  - 服務和應用程序之間的溝通應通過已發布和版本化的 API 進行溝通
  - API 撰寫規範及標準可參考 [進擊的OpenAPI - Phil Workspace][9] 此篇所述

### 6. Continuous Integration (CI) and Continous Deployment (CD) 持續整合跟持續部署
  - 系統應該透過持續整合的方式降低人工手動操作所帶來的時間浪費及增加系統透明度，進而降低風險
  - 請參閱 [[軟體工程]持續整合 (Continuous integration, CI) 簡介 - In 91][11] 

### 7. Testing Driven Development
  - 喜愛 TDD 的開發者可以考慮採用 [Integration and System Testing - Fabric8][10] 所提供的方式撰寫 Testing

## Reference
- [The Twelve-Factor Apps][1]
- [Digital Transformation is More than 12-Factor Apps][2]
- [Optimizing Twelve-Factor Apps for OpenShift by Red Hat][3]
- [軟體版本號 - Wiki][6]
- [十二因子應用程式][7]
- [微服務 - Wiki][8]
- [進擊的OpenAPI - Phil Workspace][9]
- [Integration and System Testing - Fabric8][10]
- [[軟體工程]持續整合 (Continuous integration, CI) 簡介 - In 91][11]

[1]: https://12factor.net/
[2]: https://blog.openshift.com/digital-transformation-is-more-than-12-factor-apps/
[3]: https://access.redhat.com/articles/1752483
[4]: https://medium.com/@sj82516/the-twelve-factor-app-%E9%96%B1%E8%AE%80%E7%AD%86%E8%A8%98-42f8acf3271b
[5]: http://www.the12factorapp.com/
[6]: https://zh.wikipedia.org/wiki/%E8%BB%9F%E4%BB%B6%E7%89%88%E6%9C%AC%E8%99%9F
[7]: http://www.the12factorapp.com/
[8]: https://zh.wikipedia.org/wiki/%E5%BE%AE%E6%9C%8D%E5%8B%99
[9]: https://blog.pichuang.com.tw/openapi-and-oas/
[10]: http://fabric8.io/guide/testing.html
[11]: https://dotblogs.com.tw/hatelove/archive/2011/12/25/introducing-continuous-integration.aspx