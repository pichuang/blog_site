layout: post
title: >-
  導讀 OpenShift Commons Telco SIG: Juniper Contrail: Kubernetes Multi-Interface
  Pods For Telco Use Cases
author: Phil Huang
tags:
  - openshift
  - redhat
  - juniper
  - contrail
categories:
  - openshift
date: 2019-04-05 02:14:00
toc: true
---

本導讀文基於 2019/1/25 [OpenShift Commons Telco SIG Mtg Jan 25 2019 with Guest Speakers Juniper Networks][1] 公開內容作為基礎來分享心得

{% youtube afeCtTBuppw %}

<!--more-->

## 什麼是 Juniper Contrail?

Juniper Networks 出品的 SDN Controller 產品，可同時控制 Underlay 及 Overlay 網路，也可跨平台 (OpenStack/OpenShift) 整合網路，詳細介紹請洽官網 [Contrail Enterprise Multicloud][6]

![](/images/juniper-0.png)

## Service Chain 服務鏈是?

在一般網路架構中，一條資料流通常會需要通過多個網路設備 (FW/IDS/IPS/...) 最後才能到達另一端，這就是 Service Chain (a.k.a SFC, Service Function Chain) 最常見的應用場景

在傳統網路中，Service Chain 有幾個天生限制:
1. 極受限於實體網路拓墣
2. Service Chain 很難做到依不同需求，而短時間改變
3. Scale-Out 工作負載不易

上述三個問題，基本上能透過 NFV 網路功能虛擬化及 SDN 軟體定義網路兩者技術相加起來獲得到更多的靈活彈性及擴充性

- NFV: 因從實體網路設備轉換成 VNF (Virtual Network Function) 且能放在 Common Server Hardware 上，獲得到彈性且軟硬分離的優勢
- SDN: 而軟體定義網路則可以將不同的 VNF 或 PNF (Physical Network Function) 隨需求將流量將接在一起，形成一個動態可控的 SFC，常見案例請參考下個章節

![](/images/juniper-1.png)

## Service Chain 常見案例

Juniper 分享 4 種常見案例，分別是:
1. Multiple Services in a Service Chain (include VNF, PNF)
2. Multiple Service Chains between 2 networks
3. Multiple Service Instances (Scale-out, a.k.a active-active HA)
4. Service Instances Active-backup HA

目前我個人遇到討論最多的應該還是以 `1. Multiple Services in a Service Chain (include VNF, PNF)` 為最大宗，因為這是最直覺也是第一關需要技術先克服的

![](/images/juniper-2.png)

## CNF 是現在進行式了

前不久才在 [Cloud Native Network Functions (CNF) Testbed 心得感想 - Phil Workspace][2] 講說 VNF 都還沒玩熱，CNF (Cloud Native Network Funcion) 就要出來了，沒想到 Juniper 早在那先前就先推出 [cSRX Container Firewall][3]，把自家的 SRX Firewall 方案放進去到容器裡面使用，還使用概念跟 [Multus CNI][4] 類似的 Multi-Interface Pods 技術，讓原先每一個 Pod 僅支援一個網路介面的狀況可以獲得到改善，讓容器可以獲得到更多彈性的應用

![](/images/juniper-3.png)
![](/images/juniper-4.png)


## Summary

真心訝異有廠商針對 Container 這麼快就有對應的 CNF 於容器平台上可供使用者使用，而且的確是很面向 Telco 的需求，如果大家有興趣想看詳細的 `Live Demo - Juniper Contrail 和 Red Hat OpenShift` 可以參考一下該影片後大半段的操作介紹

## References
- [OpenShift Commons Telco SIG Mtg Jan 25 2019 with Guest Speakers Juniper Networks][1]
- [Cloud Native Network Functions (CNF) Testbed 心得感想 - Phil Workspace][2]
- [cSRX Documentation - Juniper Networks][3]
- [intel/multus-cni - GitHub][4]
- [SDN中的服务链（SFC）闲聊][5]
- [Contrail Enterprise Multicloud][6]

[1]: https://youtu.be/afeCtTBuppw
[2]: https://blog.pichuang.com.tw/20190319-cnf-testbed/
[3]: https://www.juniper.net/documentation/product/en_US/csrx
[4]: https://github.com/intel/multus-cni
[5]: https://zhuanlan.zhihu.com/p/24423694
[6]: https://www.juniper.net/us/en/products-services/sdn/contrail/contrail-enterprise-multicloud/