layout: post
title: Troubleshooting from Container to Any
author: Phil Huang
tags:
  - openshift
  - container
categories:
  - openshfit
date: 2019-07-15 00:12:00
---
建立好一座 Kubernetes / OpenShift 集群後，大部分的疑難排解文章都是教你如何從 Kubernetes / OpenShift 的角度，透過 Kubernetes 的 `kubectl` 或 OpenShift 的 `oc` 指令來進行一系列的偵查，而這個查詢的結果都是基於 `kube-apiserver` 為核心的結果，中間已經抽象過很多層，距離底層資訊已有段距離。

若今天想要特意建立一個 Container 出來，用來進行 OS 或 Kubernetes 層級的 Troubleshooting 該如何實作呢?

<!--more-->

## 這樣做的動機是?

普遍在離線環境下，很常在客戶端遇到以下幾種事情

1. 客戶家沒有套件，不能臨時下載套件
2. 不能亂安裝套件，合規性會有問題
3. 想要的版本跟能提供的版本不一樣

當遇到系統問題的時候，但手頭受限於上述問題影響，導致做起事情綁手綁腳，故想要有一個完整可信任的除錯工具包 (容器) 來協助快速定位問題。同一時間這個工具也要能一體適用於 Kubernetes 平台上進行 Pod-Level 的除錯，尤其是針對 Networking 的部分。


## 當我有一個 OS...

在這個狀況下，會有兩個使用情境：

1. 容器位於 OS 之上 (Container on OS)
2. 容器位於 OS 之內 (Container within OS)

![](/images/container-on-os.png)

第一個情境比較好理解，畢竟大多數的人用 Container 都是為了這個特性。技術上其實就是直接下載 Container Images 後執行該容器內部的工具，任何的操作都不會跟外部主機 (Host OS)有任何影響。但也因為這個特性，導致如果 Container 內部有一些好用的工具，其實是不能讓外部主機做應用，白話來說就是風馬牛完全不相干。參考指令如下：

```
docker run -it --rm --name debug-container quay.io/pichuang/debug-container
```

![](/images/container-within-os.png)

第二個情境，是利用 docker `volume` 的掛載特性，將外部主機的特定幾個路徑，如 `/dev`、`/proc` 等掛載到容器裡面，而容器因為需要讀取到系統資訊，所以皆需要 `--privileged` 權限。這類在容器的操作會影響到外部主機，所以要特別留意，如果有需要特別保護的話針對特定 volume 使用 `ro` 進行保護，避免人為誤操作。參考指令如下：

```
docker run -it --rm --name debug --privileged \
       --ipc=host --net=host --pid=host -e HOST=/host \
       -e NAME=debug-container -e IMAGE=pichuang/debug-container \
       -v /run:/run -v /var/log:/var/log \
       -v /etc/localtime:/etc/localtime -v /:/host \
       quay.io/pichuang/debug-container
```

另外如果想要自己實作屬於自己的 debug container 的話，要留意一下各家 OS 的路徑規則 (man hier) 不盡相同，建議是看外部主機是什麼 OS 就選哪一種 Base Image，譬如說我個人用的主機是 RHEL7，所以 debug container 的基礎映像檔就選 rhel7 或者是 centos7 為主的映像檔，避免路徑不一樣導致神秘的現象發生。

## 當我有一個容器平台...

![](/images/k8s-networking.png)

基本 Kubernetes 的介紹就跳過了，針對網路的部分，Kubernetes 設計致力於解決以下五種網路溝通問題：

1. Container to Container
2. Pod to Pod in same node
3. Pod to Pod in different nodes
4. Pod to Service
5. Kubernetes to Internet

如果稍微認識到 Kubernetes 的看官，會了解到 Kubernetes 的最小計算單位是 `Pod`，而一個 Pod 可以`擁有一個或多個以上的 Container`，只是大多數狀況下都是一個 Pod 裡面塞一個 Container，所以很容易誤會這兩個是一樣的等級。

針對 Container to Container，基本同一個 Pod 內的 Containers 是不會跨主機的，使用相同的 network namespace，所以視為標準 L2 Bridge 情境即可。而 Pod to Pod 及 Pod to Service 的情況，Kubernetes 都會以 Pod 為基礎，透過 CNI Plugin 指派唯一的 IP 供使用，相互之間是可以溝通的。

有時候遇到一些網路問題，譬如像是不同 Node 之間網路不通或者是本身主機有異常狀況發生，其實透過 Kuebernetes 的角度是有點難以辨別到底發生什麼事情，所以類同於上面章節提到的作法，可以將特定的 Container 用 Pod 的形式運行在 namespace (or project) 裡面來協助除錯。OpenShift 參考指令如下：

```
oc project <PROJECT NAME>
oc run ocp-debug-container --image=quay.io/pichuang/debug-container \
   --restart=Never --attach -i --tty --rm
```

倘若想要將 Pod 跑在指定的節點上，則要利用 `nodeSelector` 的概念指定在特定主機上面

```
oc project <PROJECT NAME>
oc run ocp-debug-container --image=quay.io/pichuang/debug-container \
   --restart=Never --attach -i --tty --rm \
   --overrides='{ "apiVersion": "v1", "spec": { "nodeSelector":{"<KEY>":"<VALUE>"}}}'
```

記得自己要把 KEY 和 VALUE 換成自己環境的設定，關於 Label 的資訊可以從 `oc get node --show-labels=true` 這邊找

![](/images/run-on-openshift.png)

## 所以我說那個原始碼和環境呢?

以下是關於本文使用到的原始碼和 Image 位置: 
1. Container Image: [pichuang/debug-container - Quay](https://quay.io/repository/pichuang/debug-container?tab=info)
2. Dockerfile: [pichuang/debug-container - GitHub](https://github.com/pichuang/debug-container)

而測試環境為以下

1. OS: RHEL7
2. Container Runtime: Docker 1.13+
3. Container Platform: Red Hat OpenShift Container Platform v3.11

理論上來說，觀念上是適用於其他以 Docker 及 Kubernetes 為主的容器平台，所以可以自行在家試試看

## 同場加映 Part 1: sysdig

```
docker run -it --rm --name=sysdig --privileged=true \
          --volume=/var/run/docker.sock:/host/var/run/docker.sock \
          --volume=/dev:/host/dev \
          --volume=/proc:/host/proc:ro \
          --volume=/boot:/host/boot:ro \
          --volume=/lib/modules:/host/lib/modules:ro \
          --volume=/usr:/host/usr:ro \
          sysdig/sysdig
```

## 同場加映 Part 2: netshoot
- [nicolaka/netshoot - GitHub](https://github.com/nicolaka/netshoot)
```
# Container's Network Namesapce
docker run -it --net container:<container_name> nicolaka/netshoot

# Host's Network namesapce
docker run -it --net host nicolaka/netshoot

# Networks's Network namespace
# Use `nsenter`

# K8s's network debugging
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
```