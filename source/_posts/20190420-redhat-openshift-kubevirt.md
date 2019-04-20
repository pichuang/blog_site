layout: post
title: Red Hat OpenShift 把 VM 當 Container 管!?
author: Phil Huang
tags:
  - openshift
  - redhat
  - cnv
  - kubevirt
categories:
  - openshift
date: 2019-04-20 02:24:00
---
![](https://avatars2.githubusercontent.com/u/18700703?s=280&v=4)

Kubevirt 是由 Red Hat 於 2017 年初發起的一個容器虛擬化技術 Container-native Virtulaization (CNV) 專案，主要功能是以 Kubernetes 管理容器的角度來管理 VM

> Kubervirt: Add virtual machines as you know them to your OpenShift projects

<!--more-->

## Why Kubevirt?

如同[官網][2]所說，這個專案的主要目的是認為說雖然不是所有的原生在 VM  上的服務都適合進行容器化 (Containerized)，但卻又想要享有 Container Platform 原生所帶來的彈性調度能力，所以才會誕生出這麼一個專案，讓開發團隊在同一個 Container Platform 上依據不同的需求放置 VM 或 Container


## Architecture

![](/images/kubevirt.png)

從底層開始，依然是需要依靠 Bare Metal / OS / Kubernetes 建立基本 Container Platform 的能力，但原先上層所乘載的 Pod 裡面的型態應該是 Container，但現在有了 Kubevirt 這樣的專案後，可以在一個 Pod 裡面起 VM 來讓使用者做使用

技術上來說這個 VM Instance 的實作是採用 `KVM/QEMU` 的方式進行使用，而他的資源配置則是採用跟 Container 的方式依樣用 YAML 描繪資源需求，有趣的事情是 CNV 掛載硬碟的方式也是要透過 OpenShift PV/PVC 機制來進行使用

而目前 Kubevirt 的能力可以做到以下事情
- Create a predefined VM
- Schedule a VM on a Kubernetes cluster
- Launch a VM
- Stop a VM
- Delete a VM


## 技術分析

![](/images/kubevirt-1.png)

從 [kubevirt/kubevirt - GitHub][8] 得知，Kubevirt 有使用 CRD (CustomResourceDefinition) 實作專門控制 KVM 的 `VMI CRD`，而這個 CRD 包含:

- Machine type
- CPU type
- Amount of RAM and vCPUs
- Number and type of NICs
- ...

而除此之外還有實作三個不同的程式來接受和執行來至 VMI CRD 的要求 - `virt-controller` 、 `virt-handler` 和 `virt-launcher`，而整個處理流程可以參考下圖

![](https://github.com/kubevirt/kubevirt/raw/master/docs/architecture.png)


## 跟 Red Hat 的關係?

Kubevirt 是所謂的 Upstream Project，也就是大家可以任意使用的 Community 版本，可以隨意下載安裝在自己的環境自己玩  
而 [Container-native Virtualization (CNV)][3] 則是 Red Hat Downstream product，面向的是終端客戶使用，有任何問題會由 Red Hat 工程師處理 (基本上就是那些在 GitHub 上飄來飄去的大大們)

 
## Demo of CNV

可以觀看以下的 Youtube 影片，這個 Demo 是釋放於 Red Hat Summit 2018 上

{% youtube _jzZqi4qQDM %}

若意猶未盡的話，可以參考更詳細的簡報 [Introducing Container-native Virtualization - Red Hat Summit 2018][7]

## References
- [Kubevirt.io][2]
- [Getting Started with KubeVirt Containers and Virtual Machines Together - Red Hat OpenShift Blog][1]
- [Container-native Virtualization Release Notes - Red Hat OpenShift v3.11 Officail Docs][3]
- [KubeVirt - Kubernetes, Virtualization and Your Future Data Center][4]
- [紅帽正在研發原生容器虛擬化技術 -  iThome][5]
- [A First Look at KubeVirt  - Red Hat OpenShift Blog][6]
- [Introducing Container-native Virtualization - Red Hat Summit 2018][7]
- [kubevirt/kubevirt - GitHub][8]

[1]: https://blog.openshift.com/getting-started-with-kubevirt/
[2]: https://kubevirt.io/
[3]: https://docs.openshift.com/container-platform/3.11/cnv_release_notes/cnv_release_notes.html#cnv_introduction_to_cnv-cnv-release-notes
[4]: https://blog.openshift.com/wp-content/uploads/201708-KubeVirt.pdf
[5]: https://www.ithome.com.tw/news/125386
[6]: https://blog.openshift.com/a-first-look-at-kubevirt/
[7]: https://www.slideshare.net/sgordon2/introducing-containernative-virtualization
[8]: https://github.com/kubevirt/kubevirt