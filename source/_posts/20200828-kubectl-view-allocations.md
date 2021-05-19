---
layout: post
title: 如何條列出單一 Namespace 內的 Kubernetes 計算資源?
author: Phil Huang
toc: true
tags:
  - kubernetes
  - openshift
  - openshift4
categories:
  - kubernetes
  - openshift
date: 2020-08-28 00:31:45
udpated: 2020-08-28 00:31:45
---

最近這一個月出了一些事，導致太久沒寫廢文。最近有個需求需要`逆向`研究出 Red Hat OpenShift 4 原生安裝後，它預設安裝一堆的 Namespace (在 OpenShift 稱作 Project，但在這邊可以視為一樣)，各自的 CPU / Memory / Storage 資源消耗多少，身為一位計算資源精算師，所以就有了下面這篇的紀錄

本月無心力，無梗圖呈現

<!--more-->

## 為什麼要這樣做?

這件事跟 Kubernetes 計算資源估算有關係，基於[前一篇][3]所提到的 `Kubernetes 資源估算方式`，若把實機或虛機的完整可用資源當作一個玻璃杯來看，若有服務需要使用 Kubernetes 計算資源的話，則視為往玻璃杯倒水進去，當水位滿的時候，就代表該實機或虛機不能在擺放其他的服務了，當水位比較低的時候，則代表可以較有餘裕放置其他服務在裡面

![](/images/glass.png)

## Kubectl-view-allocations

這邊要推薦 [davidB/kubectl-view-allocations][1] 這個專案，相當好用，之前還沒找到這個專案之前，我都是用 `jsonpath` 慢慢爬，但我覺得不僅指令相當長 (如下指令)，而且輸出結果[不明確][2]，加上很難閱讀，所以才又另外找了一下這個專案來用用

```bash
oc get pods -o=jsonpath="{range .items[]}{range .spec.containers[]} {.name} CPU_Requests:{.resources.requests.cpu} CPU_Limites:{.resources.limits.cpu} Memory_Requests:{.resources.requests.memory} Memory_Limites:{.resources.limits.memory} {'\n'}{end}{'\n'}{end}" | column -t
```

### 超簡單安裝 Kubectl-view-allocations
```bash
curl https://raw.githubusercontent.com/davidB/kubectl-view-allocations/master/scripts/getLatest.sh | bash
sudo mv kubectl-view-allocations /usr/local/bin
kubectl-view-allocations -V
```

## Use Cases
### 條列 Namespace 內資源

譬如說我想要查 `openshift-etcd` 裡面的資源預設消耗如何，可以用下列指令辦到

```bash
kubectl-view-allocations -n openshift-etcd
```

![](/images/show-openshift-etcd.png)

### 概觀了解各節點資源

類同於 `oc adm top node`

```bash
kubectl-view-allocations -r cpu -g node
kubectl-view-allocations -r memory -g node
```

![](/images/show-node-cpu-mem.png)

### 詳細了解各節點資源

當你想要了解你的 master node 上面到底都放了什麼碗糕的東西，這個還蠻好用的...

```bash
kubectl-view-allocations -r cpu
kubectl-view-allocations -r memory
```

![](/images/show-node-memory-details.png)


## Appendix

### Environment Information
- Red Hat OpenShift 4.4.0
- Kubernetes v1.18.3


[1]: https://github.com/davidB/kubectl-view-allocations
[2]: https://gist.github.com/pichuang/59c42de25991c516909c4736caa7893d
[3]: https://blog.pichuang.com.tw/20200713-bm-and-vm-container-deployment-consideration/#Kubernetes-%E8%B3%87%E6%BA%90%E4%BC%B0%E7%AE%97%E6%96%B9%E5%BC%8F
[4]: https://www.openshift.com/kubecon