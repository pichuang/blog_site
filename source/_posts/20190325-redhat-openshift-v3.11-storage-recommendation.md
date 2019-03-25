layout: post
title: Red Hat OpenShift v3.11 儲存類型建議
author: Phil Huang
tags:
  - redhat
  - openshift
  - storage
categories:
  - openshift
date: 2019-03-25 00:02:00
---



在小弟先前的演講中 [20190218_OpenShift Storage 架構思考][2]，有提到針對每個容器在容器平台 (Kubernetes / OpenShift / ...) 上，所儲存的資料會分為三大種架構類型：

1. Data in the Container
2. Data in a Host Volume
3. Data in a Shared Storage

但我那天沒講的事情是，這些 Backend Storage 的類型應該要選擇什麼是比較恰當的？所以下面整理了一下

<!--more-->

## Backend Storage Recommendation

如果連同 Cloud 和 On-premise 儲存解決方案總計下來，有太多廠牌的儲存名字需要條列，這邊就只取明確的四種 Storage Type (Block / File / Object / hostPath) 做整理

順序由高到低為: `Recommended > Configurable > Not Configurable`

Storage Type | Single Registry | Scaled Registry | Metrics | Logging | Apps |
---|---|---|---|---|---|---|---|---|
Block|Configurable|Not Configurable|Recommended|Recommended|Recommended
File|Configurable|Configurable|Configurable|Configurable|Recommended
Object|Recommended|Recommended|Not Configurable|Not Configurable|Limited Recommended
hostPath|Configurable|Configurable|Recommended|Recommended|Configurable

Apps / Object 的欄位是 `Limited Recommended`，是因為建議直接在 Apps 層級上直接透過 API 或協定直接跟 Object Storage 溝通，而不是透過 PV / PVC 的作法來做掛載，因為現在 PV / PVC 僅有支援 Ceph RBD (Block)，而沒有 Ceph RGW (Object)

那如果你真的很想...很想...要用 Ceph RGW (Object) 來當 PV 使用的話，請等前陣子 Red Hat 收購的一個專案 [Rook.io][3] 所推出的 Operator 作法，該專案現在也是 CNCF 的成員之一，詳細的內容等 TP (Tech Preview) 階段再來介紹

![](/images/rook-1.png)

## 常見 Q&A
### Q1: 針對開發人員寫 Applications 時，到底 NFS 能不能成為 Backend Storage 的選擇之一?

A1:  
有兩個狀況
1. 如走自建 NFS 服務的話，不建議，但可以設定 (OS: 我相信99%的人都是走這個)
2. 若走特定企業儲存廠商的話，建議，可以設定

Workload Loading 如果不算高的話，其實還是可以用 NFS，但若是屬於 High Disk I/O 類型的，譬如說 Metrics / Logging / Database 這類的，就不太建議了，應該要考量其他的作法

### Q2: 那我用 Object Stroage 的話，要怎麼把 Applications 所產生的資料放進去?

A2: 建議在 Applicaions 層級上用 Object Storage 所提供的 S3/S3A API 做正常資料的存取

### Q3: 是不是用了容器平台後，所有儲存問題都可以獲得到完美的解法?

A3: 先把這篇 [你到底知不知道什麼是 Kubernetes? - hwchiu][4] 看完後，應該就能回答這題了 XD


## References
- [Optimizing persistent storage - OpenShift v3.11 docs][1] 
- [20190218_OpenShift Storage 架構思考 - Phil Huang][2]
- [rook.io - File, Block, and Object Storage Services for your Cloud-Native Environments][3]
- [你到底知不知道什麼是 Kubernetes? - hwchiu][4]

[1]: https://docs.openshift.com/container-platform/3.11/scaling_performance/optimizing_storage.html#back-end-recommendations
[2]: https://speakerdeck.com/pichuang/20190218-openshift-storage-jia-gou-si-kao
[3]: https://rook.io/
[4]: https://www.hwchiu.com/kubernetes-concept.html
