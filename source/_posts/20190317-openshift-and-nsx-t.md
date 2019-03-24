layout: post
title: Red Hat OpenShift and VMWare NSX-T 架構點評
date: 2019-03-17 20:40:52
udpated: 2019-03-17 20:40:52
tags:
- openshift
- nsx-t
categories:
  - openshift
---

最近 VMWare NSX-T 出了一篇跟 Red Hat OpenShift 3.11 的[整合文章][1]及[影片][2]，下圖的整體技術架構上是十分靠譜的，但有些點是可以另外加入考慮的，但本文不贅述，

<!--more-->

![](https://blogs.vmware.com/networkvirtualization/files/2019/02/Screen-Shot-2019-02-11-at-16.47.08-1024x553.png)

因為小弟工作關係和經驗，我會從 OpenShift 的觀點看待這個組合

## Red Hat OpenShift

Red Hat OpenShift 是一個企業級容器化容器平台 (Container Platform)，3.11 已經是 OpenShift v3 的長期支援版本 (Long-Term Support, LTS)，所以現行架構上也已經是非常清晰，包含三大節點角色：master / infra / compute，有興趣的可以參考一下小弟前陣子演講的投影片 - [那些年的 OpenShift 3.11 容器平台技術選型_20190122][3]

OpenShift 除了有內建三個不同的 CNI Plugins (ovs-subnet / ovs-multitenant / ovs-networkpolicy) 可供使用以外，其實還有跟各大知名網路廠商有做 CNI 整合驗證，VMWare NSX-T 就是其中一個可選的方案。

## VMWare NSX-T

討論 OpenShift 基礎平台資源 (Infrastructure)時，不外乎就還是 Compute / Network / Storage 這三個領域分別討論。因這個架構底層是採用 VMWare 為基底的技術，所以底層虛擬化、VM 毫無懸念都是由 VMWare vSphere 提供計算資源給上層 OS 使用。(有一個例外狀況，就是不考慮使用 VM 而用 BM 的方式部署，這邊不討論)

網路資源的部分， VMWare NSX-T 基本上是針對網路虛擬化 Network Virtualization 的技術，因需要充分納管所有 OpenShift 所需要的三大 IP 網段 (host IP / Service IPs / Pod IPs)，所以這當中會需要對 OpenShift 本體的網路做抽換 CNI Plugin 及 vSwitch 的操作。

## References
- [NSX-T Integration with Openshift][1]
- [NSX-T Openshift 3.11 Installation/Integration demo][2]
- [那些年的 OpenShift 3.11 容器平台技術選型_20190122][3]

[1]: https://blogs.vmware.com/networkvirtualization/2019/02/nsx-t-integration-with-openshift.html/
[2]: https://www.youtube.com/watch?v=uEQ5UAgh770
[3]: https://speakerdeck.com/pichuang/na-xie-nian-de-openshift-3-dot-11-rong-qi-ping-tai-ji-shu-xuan-xing-20190122