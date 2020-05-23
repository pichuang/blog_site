---
layout: post
title: Red Hat OpenShift Pod 與 Container 的愛恨情仇
author: Phil Huang
toc: true
tags:
  - openshift4
  - openshift
categories:
  - openshift
date: 2020-05-21 10:15:40
udpated: 2020-05-21 10:15:40
---

這幾天有位客戶問了一個很不錯的問題

> 如果在 1 個 Pod 裡面，有 2 個 Containers 的狀況下，OpenShift 該如何針對特定的 Container 進行操作?

按照 Kuberentes 的設計，Kubernetes 最小單位是 Pod，而不是 Container，然而 1 個 Pod，可以有 1 個或多個的 Container 被包含在裡面，圖示可以參考 [20190817 Container Bare Metal for Networking][1]，但實際上遇到多個 Container 被包含在 1 個 Pods 的時候，在 OpenShift 裡面是怎麼操作的呢? 秉持著本人遵循客戶驅動服務 (Customer Driven Service)，特別寫一篇記錄

因為最近太累，懶得想梗圖，所以附上一個近期我覺得相當不錯的 Demo，在 OpenShift 裡面開 Windows Server 2019 起來做使用

{% youtube Kx110kqoHo0 %}

<!--more-->

## 走馬看花之旅: 第七天

### 顯示一個 Pods 裡面所包含的 Container

預設狀況下，其實內建的 `api-resources` 並沒有包含一個類型是 `container`，所以如果要知道一個 Pod 裡面有包含多少個 Container 能用下列 2 個方式找

1. 透過 oc 指令 filter 出來
```bash
# 先獲得到 Pod 清單，這邊選擇 productpage-v1-597b74b4c-x2pgk
$ oc get pods

NAME                             READY   STATUS    RESTARTS   AGE
details-v1-6657b8bdf-mm4r4       2/2     Running   0          34m
productpage-v1-597b74b4c-x2pgk   2/2     Running   0          34m
ratings-v1-66cddbfb8f-wvpbz      2/2     Running   0          34m
reviews-v1-6788566f98-slm9q      2/2     Running   0          34m
reviews-v2-7c4bffdcc4-6wmg2      2/2     Running   0          34m
reviews-v3-69b6d8786-rbhw5       2/2     Running   0          34m

# oc get pods <PDO NAME> -o jsonpath='{.spec.containers[*].name}'
$ oc get pods productpage-v1-597b74b4c-x2pgk -o jsonpath='{.spec.containers[*].name}'

productpage istio-proxy
```

2. 透過 OpenShift Web Console

這個做法是直接透過相當好用的 Web Console 直接觀察，直觀方便

![](/images/show-containes.png)

### 登入其中一個 Container 裡面

一樣也是 2 種方式

1. 透過 oc rsh 指令

Red Hat OpenShift 有一個內建的指令可以做到 - `oc rsh`

> oc rsh = Open a remote shell session to a container

一般狀況下，`oc rsh <PDO NAME>` 都是直接連到`第一個` Container，如果要選擇裡面的第二個，更甚至是第三個的容器，則需要特別多一個 `--container` 指定，更多內容可以直接參考 `oc rsh -h`

```bash
# Open a shell session on the container named 'index' inside a pod
# oc rsh -c <CONTAINER NAME> pod/<PDO NAME>
$ oc rsh -c istio-proxy pod/productpage-v1-597b74b4c-x2pgk

sh-4.4$ ls
bin   dev  home  lib64       media  opt   root  sbin  sys  usr
boot  etc  lib   lost+found  mnt    proc  run   srv   tmp  var

$ oc rsh -c productpage pod/productpage-v1-597b74b4c-x2pgk

$ ls
__pycache__       productpage.py    static      stdout.log  test_productpage.py
microservice.log  requirements.txt  stderr.log  templates
```

好我知道各位一定看不懂，下面有截圖

![](/images/rsh-container.png)


2. 透過 OpenShift Web Console

相當直觀就是直接對 Web Console 操作，如下圖示表現

![](/images/terminal-container.gif)

### 針對其中一個 Container 除錯

只有一個作法，透過 Red Hat OpenShift 內建的指令 - `oc debug`

> oc debug = Launch a command shell to debug a running application

一般狀況下，如同 `oc rsh`，使用 `oc debug` 時，都是直接連到`第一個` Container，如果要選擇裡面的第二個，更甚至是第三個的容器，則需要特別多一個 `--container` 指定，更多內容可以直接參考 `oc debug -h`

```bash
$ oc debug -c istio-proxy pod/productpage-v1-597b74b4c-x2pgk

Starting pod/productpage-v1-597b74b4c-x2pgk-debug ...
Pod IP: 10.128.2.27
If you dont see a command prompt, try pressing enter.

sh-4.4$ ls
bin   dev  home  lib64       media  opt   root  sbin  sys  usr
boot  etc  lib   lost+found  mnt    proc  run   srv   tmp  var
sh-4.4$

$ oc debug -c productpage pod/productpage-v1-597b74b4c-x2pgk
Starting pod/productpage-v1-597b74b4c-x2pgk-debug ...
Pod IP: 10.128.2.28
If you dont see a command prompt, try pressing enter.

$ ls
__pycache__       productpage.py    static     test_productpage.py
microservice.log  requirements.txt  templates
$
```

一樣我知道各位還是看不懂，截圖如下

![](/images/debug-container.png)

## 後話

如果有仔細看內容的話，其實我的操作環境就是建好一個 bookinfo 之後，Service Mesh 自動注入 (inject) Sidecard 進去到每一個 Pod，所以才會有 1 個 Pod 會有 2 個 Container 的現象發生，除此之外，個人經驗還有如果需要整合網路廠商所提供的 CNI Plugin，也有機會會遇到這種等級的操作

結論上來說，雖然 Kubernetes 最小單位是 Pod，一般操作的層級都是在這個層級，但若想在 OpenShift 上針對特定的 Container 進行指令操作、除錯，都還是可以很輕鬆地辦到，也相當的直覺

最後，請大家多多訂閱 Red Hat 服務、分享開源技術及開啟技術友善好環境，我們宅宅們就靠各位客戶養了 XD

## References

- [20190817 Container Bare Metal for Networking][1]

[1]: https://speakerdeck.com/pichuang/20190817-container-bare-metal-for-networking?slide=6