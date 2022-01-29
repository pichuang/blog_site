layout: post
title: 如何科學地估算 Kubernetes 所需的資源? App 角度篇
author: Phil Huang
date: 2021-11-12 12:55:32
tags:
  - kubernetes
categories:
  - kubernetes
toc: true
---

今年 2021 小弟上半年過度忙碌，加上換球隊打球，再加上有點遇到中年危機，導致發廢文量大幅下降，請大家多多擔待

> 地上爬的我：算 Kubernetes Sizing 好累，我都以為我改行成為 Excel 架構師了呢
天上飛的學長：雲原生服務到底是要 Sizing 什麼 =_= ? 不夠不就給他加下去就對了?
地上爬的我：我們是地端...Q_Q
天上飛的學長：恩...你保重

<!--more-->

## 系列文前言

早在本篇開始，其實早在先前 [Learnk8s Kubernetes Instance Calculator][1] 已經有推出過一個蠻好看的網頁資源計算機供大家參考，但這個的問題是他沒有考量到多個 Kubernetes Namespace 部署的狀況，畢竟不可能每一個 Pod 的資源都長一樣。其次，地端跟雲端不一樣，前者採購硬體需要事先準備，後者則具備`有限的無限資源`可供使用，所以只要前者一個估不準就會出事。但身為一名 Ops 人員，我又應該要如何請 App 人員提供數字給你作為參考？

所以為了廣大的鄉民，我提供預估資源資源作法，大概可以解決`八成左右`的狀況 (其他兩成狀況，請洽業務找我去幫忙算 XD)。故以為了要管理好多個 Kubernetes 叢集需求之下，需要在每一個 Kubernetes 所部署的程式資源 [Tanzu Mission Control Extension Manager][2] 的這支程式作為估算範例

主要分為兩個方向夾擊抓資源

1. Top-Down：以 App 角度出發的 Kubernetes Application 資源估算
2. Bottom-Up：以 Ops 角度出發的 Hardware (or Cloud Provider) 資源估算

## 以 App 角度出發的 Kubernetes Application 資源估算

想問問大家一個 Kubernetes 要部署一個 Application 會需要什麼東西? 是的，你需要去描述你的應用程式資源和相關資訊，並且部署在特定的 Kubernetes Namespace，如 `vmware-system-tmc`，所以身為一個 App 人員的目標就是把 `基於 namespace 為主的內部所需資源條列出來`，擷取 [Tanzu Mission Control Extension Manager L880-L957][3] 內容，如下

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    tmc.cloud.vmware.com/orphan-resource: "true"
  labels:
    app: extension-manager
    control-plane: extension-manager
    controller-tools.k8s.io: "1.0"
    tmc-extension: "true"
    tmc-extension-name: extension-manager
    tmc.cloud.vmware.com/managed: "true"
  name: extension-manager
  namespace: 'vmware-system-tmc'
spec:
  minReadySeconds: 30
  progressDeadlineSeconds: 600
  replicas: 1
  selector:
    matchLabels:
      control-plane: extension-manager
      controller-tools.k8s.io: "1.0"
      tmc-extension: "true"
  strategy:
    rollingUpdate:
      maxSurge: 100%
  template:
    metadata:
      labels:
        control-plane: extension-manager
        controller-tools.k8s.io: "1.0"
        tmc-extension: "true"
        tmc-extension-name: extension-manager
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
            - matchExpressions:
              - key: beta.kubernetes.io/os
                operator: In
                values:
                - linux
      containers:
      - command:
        - /usr/local/bin/manager
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: 'extensions.vmware-cloud.tmc.cloud.vmware.com/extensions/extension-manager/extension-manager@sha256:'
        imagePullPolicy: Always
        name: extension-manager
        ports:
        - containerPort: 9876
          name: webhook-server
          protocol: TCP
        resources:
          limits:
            cpu: 100m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 128Mi
        securityContext:
          runAsGroup: 1000
          runAsUser: 10000
      serviceAccountName: extension-manager
      tolerations:
      - operator: Exists
```

對於 Kubernetes Application 資源估算你應該要關心以下幾個點

1. kind 的類型：主要先以可涵蓋大部分部署場景為主，常見有 3 大 Kind，Deployment、StatefulSet、DaemonSet，如本範例為例 `kind: Deployment`
2. spec.replicas 的數字：主要是要確認待會下面的 CPU / Memory 是要乘上多少，常見應該都是以 `1` 為主，至於為什麼可參閱我於 [之前在 Microsoft DevDays 2021 演講的內容][4]，如果微服務能夠 Scale-Out 的話，就是 `2` 或以上的數字
3. 每一個 Pod 所需要的 CPU 和 Memory 資源：主要是確定一個 Container 所需要的 CPU 及 Memory 的上下限，要留意 `1 個 Pod 裡面會有 1 個或多個 Containers`，可詳閱[之前在 HKOSCon 2020 演講內容][6]
```
        resources:
          limits:
            cpu: 100m
            memory: 256Mi
          requests:
            cpu: 100m
            memory: 128Mi
```
4. Storage 的選型：主要是要確定是要透過 hostPath 肚子的硬碟還是使用 PV/PVC 採用基於 CSI 介面的外掛儲存服務，常用協定如 NFS、iSCSI 及 FC 等，這個主要都是以服務對於 IOPS 或其特性的考量為採用標的，但本文不深入探究

所以把上面的資訊收集起來，會得到下面這張表

![](/images/tmc-res.png)

但實際上你填完，會注意到一些事情

1. 這樣總資源是對的嗎?
2. 有些欄位沒填的怎麼辦?

回應第一題，一定不對，因為你看 `Replicas` 有些是 2，所以實際上你應該要把左邊的資源全部乘上 `Replicas` 數字，身為一個 App 團隊你應該會得到下面這張表

![](/images/tmc-res-1.png)

這時候你把全部的資訊撈出來，可以獲得到 `大約`這支整個程式再怎麼跑，頂多 `vmware-system-tmc` 這個 namesapce，就是吃掉 `4700m` CPU 跟 `6382M` Memory，至於對於 Kubernetes 計算單位不熟的可以參考 [Kubernetes Managing Resources for Containers][7]，另附上 [Google Sheet 參考文件][9]

回應第二題，呈上，如果你沒寫資源，這不是代表他運作不用資源，是代表這份 Yaml 沒有定義 limits / requests 兩個數字去限制他的資源使用，而是會用 BestEffort 的方式運行。在這個狀況下，如果真的寫不出來，真的是只能憑經驗先估一個數字給 requests 確保不會爆炸，再來用 Metrics 服務，如自建 Prometheus、或 SaaS 服務 Tanzu Observability 長期觀測調整數字為比較務實的作法。倘若你如果選用的程式是 Java JVM 或者是自帶程式語言虛擬機，會相當好估算，但本文不深入探究

那額外問題來了，如果我在資源裡面填 0 的話會怎樣?

## 關於 0 的那些事

> 先講結論，建議還是要設定 limits 跟 requests 的值上去，最少也要有 limits

當資源真心不足的時候，則 Kubernetes 則有內建支援 QoS 服務管理能力，且支援作業系統等級的 OOM 控制，最小計算單位為 Pod。`一般來說，估算計算資源 limits 比較重要，實際上運行調度則是 requests 比較重要`

Kubernetes 會開始依據 QoS 三種等級（由最高到最低優先權排序）：Guranted > Bursatable > BestEffort 進行權重排序，通常會發生 OOM (Out-of-Memory) 也都是因為撞到資源不足，然後優先權重又低，就被幹掉了，常見如 AI/ML 容器等會吃資源的容器服務，確切的詳細內容請參閱 [Kubernetes - 配置 Pod 的服务质量][8]

| 優先權 | QoS 類型 | 描述 |
|:--------------:|:--------:|:------:|
| 1 (highest) | Guaranted | Pod 裡面的每一個 Container 都要設定 requests 等於 limits (兩個值皆有設定，且不等於 0) |
| 2 | Burstable | Pod 裡面的任一個 Container 具有 requests 不等於 limits 數值，且不符合 Guaranted 條件|
| 3 (lowest) | BestEffort | 沒有設定任何資源請求和限制 |


當然你看完上面應該會霧傻傻，所以身為 Excel 精算師的我幫大家整理了以下列舉了已知組合，供大家參考

![](/images/tmc-res-4.png)

你看完上面的圖應該會知道一個事實，`0` 在 Kubernetes 來說是有意義的，所以資源估算請不要亂寫，他不代表無限大資源的意思...如果真的不知道請留空，而不是寫 0

## 以 App 角度出發的常見資源錯估誤區

你是不是覺得到這邊就可以估算結束了？`很抱歉沒有`，你還少算運行 Kubernetes 所需要的服務，如 kubelet、CNI、kube-proxy、OS 本身服務等等這些不存在於 kubernetes 的資源清單上，但實際上是需要這些服務才能夠好好的運行 Kubernetes 服務，那這是什麼意思呢?

拿房子權狀比喻，以上所做的事情都是在算室內面積（客廳、浴室、陽台），實際上你還有公設（樓梯、電梯、健身房）要算啊！所以最終估算 1 個 Kubernetes 所需資源的結果，一定會比 App 所寫的還要來得大上一點

![](https://learnk8s.io/a/5505f01f0ed678a7ac42642b4dfd7b1c.svg)

而各大廠商為了要降低公設比，紛紛都推出了 Container OS 進行資源和效能上的最佳化，畢竟誰希望拿到 1 個 4c/8g 的 worker 節點，把一堆跟 kubernetes 沒關係的服務開起來就剩不到一半資源可以用，但因為各家出 Container OS 系統都不盡相同，所以會很仰賴各自廠商工程師的精算

## 結語

看完以上的文章，相信大家一定會覺得為啥我要個 Kubernetes 為啥要搞得這麼複雜，所以下篇會講，`以 Ops 角度出發的 Hardware (or Cloud Provider) 資源估算`，就是寫不出來的狀況下身為 Ops 到底該怎麼辦？

希望今年能寫得出來...希望...

特別感謝 Gene Kuo 和 Hwchiu 幫忙校稿 XD

[1]: https://learnk8s.io/kubernetes-instance-calculator
[2]: https://gist.github.com/pichuang/eb50ee49e4cdb64549df335216cc5290
[3]: https://gist.github.com/pichuang/eb50ee49e4cdb64549df335216cc5290#file-sample-tmc-deployment-yaml-L880-L957
[4]: https://speakerdeck.com/pichuang/20210812-ru-he-cai-yong-yun-yuan-sheng-zuo-fa-velero-bei-fen-huan-yuan-ni-de-kubernetes-cong-ji?slide=8
[5]: https://gist.github.com/pichuang/eb50ee49e4cdb64549df335216cc5290#file-sample-tmc-deployment-yaml-L943-L949
[6]: https://speakerdeck.com/pichuang/how-do-i-troubleshooting-on-container-more-than-docker?slide=17
[7]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
[8]: https://kubernetes.io/zh/docs/tasks/configure-pod-container/quality-service-pod/
[9]: https://docs.google.com/spreadsheets/d/1zQswfXuZhpKjsmmFvCHiGt19Ej6s6hpr5yiB1wnPR70/edit?usp=sharing
