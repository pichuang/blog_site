layout: post
title: 給行銷跟業務的 Kubernetes 101 中翻中介紹
author: Phil Huang
date: 2021-01-11 12:55:32
tags:
  - kubernetes
  - openshift
  - openshift4
categories:
  - kubernetes
  - openshift
toc: true
---

擔任售前以來，常常遇到很多剛接觸 Kubernetes 的業務、行銷或者是非資訊領域的人，其實一直對 Kubernetes 到底在各個文案和標書可以扮演什麼角色一直不是很清楚。今天我嘗試用中翻中的角度，也就是用白話文來跟各位解釋 Kubernetes

有鑒於太多人對於 OpenShift 4 有些小功能很好奇，我有機會會慢慢錄一些無聲小影片給各位參考

{% youtube yGhcPmTtoPU %}

<!--more-->

## 正常的 Kubernetes 介紹

我不太會講，貼別人的比較快

- [Hwchiu Learning Note - Hwchiu][7]
- [淺談 Kubernetes 高可靠架構 - Kyle][6]

## 中翻中的 Kubernetes 組件介紹

![](/images/building.jpeg)

> 我們就是在蓋一個中大型住戶社區，然後塞滿各式各樣的行李箱的平台

請先想像建立 Kubernetes 完整的叢集服務，其實就像是要建立一個中大型住戶社區，那所對應的幾個名詞解釋就會是下面的描述

- Kubernetes：建立一個住戶社區的標準藍圖
- 虛擬化平台、三大公有雲平台、實體機：地皮（怕有人看不懂，主要指 AWS / GCP / Azure / VMware vSphere / Red Hat OpenStack / Nutanix / Proxmox / 純實體機資源）
- 虛擬機、實體機：住戶社區內一棟一棟的大樓坪數空間（怕有人看不懂，主要是指要 vCPU/vRAM還是用 pCPU/pRAM 計算）
- OS 作業系統：每棟大樓骨幹和地基
- Master Node：住戶社區管委會主要居住大樓，為保持高可用最少要 3 棟
- Etcd Cluster：管委們，為保持高可用，故建議最少三位管委，且互相投票選出一位委員長
- Worker Node：住戶們主要居住大樓
- OCI (Open Container Initiative)：訂立大樓鋼筋水泥及行李箱標準的組織
- CRI (Container Runtime Interface)：大樓鋼筋水泥廠商
- CNI (Container Network Interface)：大樓水電力系統廠商
- CSI (Container Storage Interface)：大樓空間規劃廠商
- Pod：住戶，而一棟大樓裡面可以有很多的住戶
- Pod IP：每個住戶的門牌號碼
- Ingress Controller：住戶社區的社區大門，可以指定讓社區成員都固定走同一個或多個門入口管控
- Egress Controller：住戶社區的社區後門，可以指定讓社區成員都固定走同一個或多個門出口管控
- Internal DNS / Service Discovery：大樓住戶黃頁簿 (怕有人看不懂，這個技術上常見是 CoreDNS)
- External DNS：指向各大樓的路標 (怕有人看不懂，這個技術上常見是 AD / bind9 / dnsmasq)
- Service：住戶社區裡的社團（如太極拳、種花愛好會、麻將會）
- Service Mesh：社團的聯絡名冊
- Containers：被打開的且正在被使用的行李箱，在一個住戶裡面，可以放一個或者是多個以上
- Container Images：還沒被打開的行李箱
- Container Registry：行李箱集中存放中心
- Bastion：維護整座社區的工程車
- Promethus：住戶社區整體監控中心（Meterics）
- Grafana：監控中心裡的超大型 LED 儀表板
- Elaticsearch：住戶社區整體情資中心（Logging）
- Kinbana：情資中心裡的超大型 LED 訊息版

## 常見問題
### Q1: 每個人宣稱它是 Kubernetes 認證，那到底有什麼不一樣呢？

拿到符合 Kubernetes 認證的標記，代表具備有建設社區大樓的能力，但並不代表每個人建設出來的社區大樓都是一模一樣的。隨著選商 (Vendor)、用料 (Source Code)、工法 (Complier) 等不同會有價格上的差異，或者是隨著維護 (Maintain)、升級 (Update/Upgrade)、安全管理 (Security) 等不同會有管理上的難易程度

> 你應該不會說每一個建商所蓋出的大樓品質都會是一樣的吧？有些只有蓋好大樓，內裝、外觀、拼裝、工法、維護都亂七八糟，這也是個能住的大樓嗎？

技術上來說，[Certified Kubernetes][2] 裡面羅列了基於 Kubernetes 個別版號通過的平台版本，具備參考價值

### Q2: 有些資安廠商說他們也可以支援 Kubernetes Security，但具體是保護什麼?

就像社區大樓保全解決方案，有些弄監視器、有些是監控設備、有些是人員管控，要看各自專案或廠商想要解決的問題，據我個人理解有些廠商是專注在保護作業系統、有些是會對 Container 進行映像檔靜態、或者是 Runtime 狀態下的動態掃描等等各種類型

技術上來說，Kubernetes 官方有提出 [Overview of Cloud Native Security][1] 觀念，針對 Cloud、Cluster、Container、Code 四個層面進行保護，當然還不單止這樣，對外及對內的網路管理也相當重要，所以實際上比較多人會討論的是 Ingress Controller 比較多，也就是住戶社區的社區大門人流控制跟安全性

### Q3: 我們常說 Kubernetes 的最小單位是 Pod 這個可以怎麼理解呢?

你家管委會應該不會管住戶家裡的行李箱塞什麼吧

### Q4: 那麼 Docker 在裡面的角色是什麼?

眾多 CRI (Container Runtime Interface) 選擇之一，也就是大樓鋼筋水泥廠商有很多間，只是有一間叫做 Docker 的特別有名，但 Docker 當初成立的時候，是早於 Kubernetes 的，所以導致為了要讓 Kubernetes 能支援 Docker，在功能上加了一些料進去並且維持了一段時間，故這個狀況在 Kubernetes 部落格 2020/12/02 發布的文章 [Don't Panic: Kubernetes and Docker][3] 被改變了，文末有提及說建議可以換成比較通用的 CRI-O 及 Containerd 其中之一為主

若想要更深入了解此改動影響的話，相當建議收聽 Cloud Native Taiwan User Group 出品的 [一言難盡的關係：Kubernetes 和 Docker 之線上討論分享會][4] 內有多個角色的分享，包含開發者、自建維運者、供應商的角度

### Q5: 這意味著 CRI / CNI / CSI 是不是可以被替換成其他的廠商?

Yes，所以你會看到各式各樣的廠商都開始支援 Kubernetes 是因為這個原因，因為
一般狀況下 Kubernetes 並沒有特別限定 `大樓鋼筋水泥廠商`、`大樓水電力系統廠商`、`大樓空間規劃廠商` 選型，只要有符合各自定義的標準即可，但要留意 Kubernetes 的版號會連帶影響這三個標準介面 CRI / CNI / CSI *能支援*的發行版本號，千千萬萬務必要留意相容性問題，強烈建議規劃時洽專業蓋住戶社區的廠商諮詢

正常來說，最少 CRI 跟 CNI 這兩個的選擇，最好要聽蓋大樓的廠商建議，因為...沒測過和壓過到時候維護的時候會很可怕，而 CSI 選擇彈性就相當多樣了

> 你應該不會希望隨便找一個沒驗過的水電廠商就來蓋大樓吧？漏水和漏電會很可怕的喔...

### Q6: 整個住戶社區最重要的角色是什麼?

那三棟委員會大樓，和裡面的三位委員，三位掛掉一位還可以維持正常運作，掛掉兩位維持唯讀運作

### Q7: Kubernetes、VM、Container 的差異性該如何理解?

你可以建立好一個大樓 (VM)，隨意放置一個或多個行李箱 (Containers)，你需要手動管理這些行李箱，如果資源不足或者是這個大樓倒了，是沒有辦法自動飄移這些行李箱所裝載的內容

但如果你多具備有了 Kubernetes 能力，你可以建立多個大樓 (VM)，透過 Kubernetes 所規定的放置計畫 (e.g. Deployment / DaemonSet / ...)，可以統一調度這些 Pod，當某個大樓資源不夠的時候，可以根據 Kubernetes 的規則進行搬遷或擴充大樓等動作

### Q8: 為什麼常常有人會把 Container 跟 Pod 混在一起講?

因為絕大部分的狀況下，都是一個用戶 (Pod) 放一個行李箱 (Container)，所以導致有這種誤會。實際上，一個用戶 (Pod) 是可以放置一個或多個行李箱在裡面的

### Q9: 常聽到 Node Scaling 是什麼意思?

當住戶太多的時候，Kubernetes 可以自動或手動興建大樓，然後把多出來的住戶塞進去

{%youtube MGm47ncUp6o %}

### Q10: OS 作業系統在 Kubernetes 的領域中重不重要?

當有地震還是什麼災害來襲的時候，你會覺得地基跟骨幹很穩很重要...

技術理由可以參考 [Containers are Linux][8] 那一個章節內容

### Q11: 宣告式或聲明式管理是什麼意思?

只要你有需要把行李箱放置到這個社區內，都需要提出部署計畫 (Deployment) 給委員會審核，只要通過委員會審核，他們就會照你`事先聲明的計畫`，盡`最大可能性`放置行李箱

所以，如果你有什麼變更計畫，都需要遞交一份新的部署計畫書出來，讓委員會重新審核和接受

### Q12: 規劃上，是不是都要每棟大樓的坪數 (CPU/RAM) 都要一樣? 可以混用嗎?

可以混用，只是要注意每棟大樓本身還是有基本供給的水電費的部分需要維運，也就是作業系統啟動的基礎成本

### Q13: 承上，那我不就建一個超大坪數的大樓就好了？還需要分大樓嗎

實際上，依據不同廠商的設計，一棟大樓裡面最多可以塞的住戶 (Pod) 數量從 25 ~ 500 戶不等，而且這還牽扯到高可用性 (HA) 及水電資源 (Networking) 估算問題，因為也不是可以無限狂拉的好嗎...是有一個臨界值可以用計算機算出來的

若你具備網路概論知識的話，知道 IP Subnet (或者知道如何用 IP 計算機)，可以參考 [了解當下網路資訊 - Cluster Network Operator][9] 部分，深入了解實際計算方式



## 結語

> Red Hat 就是家專做 IT 版本的住戶社區建設公司及室內空間規劃公司，已經蓋了第四代的 Kubernetes，也就是 OpenShift 4

![](/images/redhat-building.jpeg)

希望這篇能幫助到廣大有看 Kubernetes 但沒有懂這是什麼東東的業務和行銷朋友們

[1]: https://kubernetes.io/docs/concepts/security/overview/
[2]: https://github.com/cncf/k8s-conformance
[3]: https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/
[4]: https://www.youtube.com/watch?v=nc3mBN3LzvM&t=2s
[5]: https://speakerdeck.com/pichuang/how-do-i-troubleshooting-on-container-more-than-docker-openshift-special-edition
[6]: https://k2r2bai.com/2019/09/19/ironman2020/day04/
[7]: https://www.hwchiu.com/
[8]: https://blog.pichuang.com.tw/20200713-bm-and-vm-container-deployment-consideration/
[9]: https://blog.pichuang.com.tw/20200403-openshift-with-coreos-part-2/#%E4%BA%86%E8%A7%A3%E7%95%B6%E4%B8%8B%E7%B6%B2%E8%B7%AF%E8%B3%87%E8%A8%8A-Cluster-Network-Operator