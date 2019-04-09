layout: post
title: 到底 Operator Framework 用起來會長得如何?
author: Phil Huang
tags:
  - redhat
  - openshift
  - operator
categories:
  - openshift
date: 2019-04-09 22:49:00
---
打從 Red Hat 收購 CoreOS 後，底下的子專案也一併地納入紅帽的管理，其中 Operator Framework 也在其中。而很多朋友只聞 Operator Framework 聲響，不聞 Operator Framework 身影，今天透過此篇文章來介紹一下 Operator Framework 實際上對 Red Hat OpenShift 容器平台會有怎樣的玩法？

![](/images/operator-0.png)

<!--more-->

## 為什麼我需要了解 Operator Framework?

一般來說透過 Kubernetes / OpenShift 這類容器平台管理無狀態應用服務 (Stateless Apps) 是容易的，不需要額外的狀態處理，可以很快速地做到 Scale-Out 或輕鬆地從故障 (Auto-Healing) 恢復，畢竟這是一開始設計的目標。而現實生活中，大多數都是有狀態應用服務 (Statefule Apps) 居多，包含資料庫、Cache System 等，這些系統需要各自領域的知識才能將服務做到 Scale-Out / Upgrade / Re-configuration。

各位可以試想，一般部署的時候，是不是都是透過撰寫 YAML 檔將 Deployment / RC 等等不同類型的資源描述出來，遇到更新或需要修改設定檔的時候，基本上也都要一個一個地進行修改，而這件事是沒有問題的，但問題是... `你怎麼知道針對這些特定領域的系統，你所進行修改的檔案或升級的步驟是對的?`，這也是當初 Opeator 誕生的原因
  
> 透過 Operator 將特定領域的系統知識或操作經驗，以 Source Code 的形式呈現；而這個 Source Code 的表述形式，則是用擴展 Kubernetes API 的方式: CRD (Custom Resource Definition) 來實現

至於該專案誕生的其他故事，可以參考 [Kubernetes API 与 Operator：不为人知的开发者战争（完整篇）][13]

## 所以... Operator Fraemwrok 包含什麼?

![](/images/operator-1.png)


Operator Framework 本身是一個框架，裡面包含 3 大部分:
- Operator SDK：針對開發人員，提供三種不同的 SDK：Go / Ansible / Helm
- Operator Lifecycle Management (OLM)：針對維運人員，帶有五種能力等級：Basic Install / Seamless Upgrades / Full Lifecyle / Deep Insights / Auto Pilot
- Operator Metering：針對維運人員，提供監控回報及數據，需搭配 OLM

目前大多數的實作都還是落在利用 Operator SDK 實作出 OLM 所需的資源，還不到整合 Operator Metering 的階段

## 那 Operator Lifycycle Management 的生命週期指的是?

![](/images/operator-3.png)

主要分五個階段
1. Basic Install: 最基本能力，可以自動部署相關程式和設定在容器平台上
2. Seamless Upgrade: 可支援進行升級小版本和進行修補 patch
3. Full Lifecycle: 具備 backup / failure / recovery 程式資料能力，也是現行各 Operator 實作上最多的階段
4. Deep Insights: 支援 Metrics / Alerts / Log 等分析數據指標能力
5. Auto-pilot: 可支援自動擴展和自動性能調教



## 什麼是 Operatorhub.io?


假設我們都同意 `Operator` 是個好東西，想要試試看的話，那要從哪裡獲得到這些 operator 的集中化資訊? 

首先最常聽到的就是 [operator-framework/awesome-operators - GitHub][14] 這個 Repos，收集了由社群貢獻的 operator 程式，但基本上就是一個條列清單供使用者直接去點選使用。但倘若想要做初期評估的話，想快速了解特定的 Operator 具備什麼樣子的能力、CRD 清單和描述，就會要花費一定的時間慢慢地研究個專案的 README。

而 Red Hat 在前陣子放出了一個網站 [`operatorhub.io`][15]，目的就是將這些 Operator 資訊集中化顯示在 WEB 上，可以方便各大 ISVs 、社群專案能夠在上面發布各自專案的資訊，詳細如下圖

![](/images/operator-2.png)

現在 OperatorHub 也廣徵各式好手將自己的 Operator 放在上面做分享 [OperatorHub.io - Contibute][16]，這邊同時也分享教學影片 [How to Add Operator to Operator Hub 2019][17]

## 那在 Red Hat OpenShift 上會如何使用 Operator?

推薦將兩個 YouTube 的內容看過 [OpenShift 4 Red Hat Operators - YouTube][1] 和 [OpenShift 4 ISV Operators - YouTube][2]

{% youtube HzkE7CZU7Bg %}


## 想要深入技術細節?

如果想要了解 Operator SDK for Helm / Ansible 的朋友歡迎點選下列兩個影片
- [OpenShift Commons Briefing Helm Operators Deep Dive - OLM for Helm People Rob Szumski - YouTube][4]
- [OpenShift Commons Briefing Operators For Ansible People - Michael Hrivnak @RedHat - YouTube][5]

若想要嘗試用 go 寫人生第一個 Operator 的話，可以參考強者我朋友 Samina 的文章 - [第一次玩 operator-sdk 就上手 - Life & Technological Journey of Samina][11]

既然 Operator 可以做到有狀態程式的管理，那有沒有人做 Operator of Operator? 不是開玩笑的，還真的有 - [OpenShift Commons Briefing All Things Operators State of Operators with Daniel Messer - YouTube][3]


## References
- [OpenShift 4 Red Hat Operators - YouTube][1]
- [OpenShift 4 ISV Operators - YouTube][2]
- [OpenShift Commons Briefing All Things Operators State of Operators with Daniel Messer - YouTube][3]
- [OpenShift Commons Briefing Helm Operators Deep Dive - OLM for Helm People Rob Szumski - YouTube][4]
- [OpenShift Commons Briefing Operators For Ansible People - Michael Hrivnak @RedHat - YouTube][5]
- [Installing OpenShift 4 on AWS with operatorhub.io integration - YouTube][6]
- [OperatorHub.io][7]
- [After CoreOS_20190214 - Phil Huang][8]
- [Prometheus Operator 介紹與安裝 - KaiRen's Blog][10]
- [第一次玩 operator-sdk 就上手 - Life & Technological Journey of Samina][11]
- [CoreOS釋出Operators開發框架，助自動化管理Kubernetes應用程式 - iThome][12]
- [Kubernetes API 与 Operator：不为人知的开发者战争（完整篇）][13]
- [operator-framework/awesome-operators - GitHub][14]
- [OperatorHub.io][15]
- [OperatorHub.io - Contibute][16]
- [OpenShift Commons Operator Framework SIG Mtg Rob Szumski How to Add Operator to Operator Hub 2019][17]

[1]: https://www.youtube.com/watch?v=HzkE7CZU7Bg
[2]: https://www.youtube.com/watch?v=KNbHNXXHzFY
[3]: https://www.youtube.com/watch?v=GgEKEYH9MMM
[4]: https://www.youtube.com/watch?v=1on_wRY2dzQ
[5]: https://www.youtube.com/watch?v=iIvwhNiYKYE
[6]: https://www.youtube.com/watch?v=kQJxGtsqphk
[7]: https://operatorhub.io/
[8]: https://speakerdeck.com/pichuang/after-coreos-20190214?slide=22
[9]: https://github.com/operator-framework/awesome-operators
[10]: https://k2r2bai.com/2018/06/23/devops/prometheus/prometheus-operator/
[11]: https://bestsamina.github.io/posts/2019-02-04-first-operator-sdk-helm/
[12]: https://www.ithome.com.tw/news/122842
[13]: http://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2019/01/08/kubernetes-api-and-operator-history.html
[14]: https://github.com/operator-framework/awesome-operators
[15]: https://operatorhub.io
[16]: https://operatorhub.io/contribute
[17]: https://www.youtube.com/watch?v=-6dLnKOtgVY