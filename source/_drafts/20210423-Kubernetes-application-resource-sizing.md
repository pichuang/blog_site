layout: post
title: Kuberetnes 專案資源計算
author: Phil Huang
date: 2021-04-23 12:55:32
tags:
  - kubernetes
  - openshift
  - openshift4
categories:
  - kubernetes
  - openshift
toc: true
---

首先先講一件大事，一年一度[開源人年會又來啦][1]！COSCUP 2021 歡迎各公司行號贊助議程兼找找好手，而且ㄧ如往常 Cloud Native Taiwan User Group (CNTUG) 也有一軌議程軌`開源運河上的雲原生號`，請一年一講的講者快來[投稿][2]啊啊啊啊啊！

其次，上半年忙到被鬼抓走了，沒力氣寫文章，所以預期今年出文速度會超慢...

最近遇到好幾個案子都有 App Team 遇到差不多的問題，到底怎麼估專案資源，在公有雲的使用者沒這種壓力是因為 Cloud Provider 都幫你準備好了，等著你去租用（帳單記得要付 >.^），跟地端的情境不一樣，在資源不夠之前，就要需要走採購流程採買伺服器，通常...幾個月就過了

<!--more-->

## Kubernetes 資源基本單位

關於 Kubernetes 的計算資源主要有 3 種單位：
- CPU：1 Core = 1000m
- Memory：4GB = 4GB
- Storage：120GB = 120GB

絕大部分狀況下，會需要探討的是 CPU / Memory 的估算，而 Storage 儲存類型的選擇、IOPS 和儲存大小差異，可參閱小弟早期文章 [Red Hat OpenShift v3.11 儲存類型建議 - Phil Huang][3]

至於單位計算的詳細解釋，因為蠻多文章寫的都不錯，為節省篇幅，建議可以參考 [[Kubernetes] 分配 & 管理 container 所使用到的計算資源 - 小信豬的原始部落][7]

## Kubernetes 資源類型差異

這邊要特別解釋一下 CPU 和 Memory 的資源類型差異，CPU 是`資源可壓縮`，Memory 則是`資源不可壓縮`，意思是在資源不足的時候，會有一些特殊表現出來

- CPU 主要是以`分配時間`為主，不具備狀態儲存，申請資源很快，回收也很快，當 CPU 資源不夠的時候，各程式會依據權重分配給所有應用程式使用，但不影響程式運作本體
- Memory 主要是以分配`記憶體空間`為主，具備資料及狀態儲存，申請資源很慢，回收慢，當 Memory 資源不夠的時候，會直接跟你說無法繼續申請資源，然後開始根據 QoS 進行處理

|  資源類型 | 資源 | 資源不夠的狀態 |
|:--------------:|:--------:|:------:|
|  可壓縮資源 (Compressible)  | CPU / Network IO | 慢~慢~等~，直到 CPU Scheduled 和 Allocated，容器會持續保持在同一個節點上運行 |
| 不可壓縮資源 (Incompressible) | Memory / Disk | 容器會根據 Restart Policy 進行終止 (Terminated) 和重啟 (Restart) |

## Kubernetes QoS 分類

當資源真心不足的時候，則 Kubernetes 則有支援 QoS 服務管理能力，且支援作業系統等級的 OOM 控制，最小單位為 Pod

Kubernetes 會開始依據 QoS 三種等級（由最高到最低優先權排序）：Guranted > Bursatable > BestEffort 進行權重排序，通常會發生 OOM (Out-of-Memory) 也都是因為撞到資源不足，然後優先權重又低，就被幹掉了，常見如 AI/ML 容器等會吃資源的容器服務

| 優先權 | QoS 類型 | 描述 |
|:--------------:|:--------:|:------:|
| 1 (highest) | Guaranted | request 等於 limits (兩個值皆有設定，且不等於 0) |
| 2 | Burstable | request 不等於 limites (兩個值皆有設定，且不等於 0)|
| 3 (lowest) | BestEffort | 沒有設定任何資源請求和限制 |




## References


[1]: https://www.facebook.com/groups/cloudnative.tw/permalink/1120702608433073/
[2]: https://blog.coscup.org/2021/04/coscup-2021-cfp.html
[3]: https://blog.pichuang.com.tw/20190325-redhat-openshift-v3.11-storage-recommendation/
[4]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes
[5]: https://sysdig.com/blog/kubernetes-limits-requests/
[6]: https://sysdig.com/blog/troubleshoot-kubernetes-oom/
[7]: https://godleon.github.io/blog/Kubernetes/k8s-Scheduling-Manage-Compute-Resource-for-Container/
[8]: https://access.redhat.com/documentation/zh-cn/openshift_container_platform/4.6/html/nodes/nodes-cluster-overcommit