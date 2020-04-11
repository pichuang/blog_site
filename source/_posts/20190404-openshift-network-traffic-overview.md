layout: post
title: Red Hat OpenShift v3.11 東西南北向網路探討
author: Phil Huang
tags:
  - redhat
  - openshift3
  - openshift
  - networking
categories:
  - openshift
date: 2019-04-04 00:45:00
---
## 資料中心東西南北向網路流

一般討論的資料中心資料流量，主要分為兩大流向
- 南北向 (North-South Traffic):  Client 與 Server 之間的網路流量
- 東西向 (East-West Traffic): Server 與 Server 之間的網路流量

![](/images/network-traffic.png)

那如果套用在 Red Hat OpenShift v3.11 容器平台架構下，那實際網路流量是如何對應?

<!--more-->

## Red Hat OpenShift 東西南北向網路流

首先先來張常見的架構圖

![](/images/openshift-network-traffic-1.png)

以 Red Hat OpenShift 為核心視角，可以分為三段網路
1. 北向網路流 Ingress Traffic
2. 東西向網路流
3. 南向網路流 Egress Traffic


### 北向: Ingress Traffic

從 [What is Ingress? - Kubernetes][2] 定義上來說，可以了解到 Ingress 主要的目的就是讓外部使用者可以透過 `Ingress Controller` 接觸到 Kubernestes 裡面的 Service，而在 Red Hat OpenShift 裡面的術語則是使用 `Router` 來做闡述

故在 [Routes - OpenShift Docs v3.11][4] 裡面所描述的內容，可以用下表進行術語上的對應

Kubernetes | OpenShift
---|---
Ingress resource (rules) | Route (rules)
Ingress controller (Nginx/Ambassador/...) | Router (HAProxy)

而 Ingress Controller 其實是一個統稱，在 OpenShift Router 具體實踐上主要有[支援兩種類型][3]
1. OpenShift Router (預設)
2. F5 BIG-IP Controller (要搭 F5)

有些人會問說同一平台內，能不能存在多個 Ingress Controller? 答案是可以，但要留意各 Ingress Controller 的實作方式，譬如說 OpenShift Router 的方式是採用 `hostnetwork` 的走法，所以每一台 host 僅能安裝一個，詳細可參考 [從 Red Hat OpenShift 外部訪問 Services 或 Pods - Phil Workspace][6]

### 東西向: East-West Traffic

在 Red Hat OpenShift 平台底下，無論是要跨 Pod / Service / Host 進行橫向的溝通，都是需要有 CNI (Cotainer Network Interface) 的實踐才能將不同主機之間的連線建立起來。CNI 本身是一套標準，實作上就會有百花齊放的專案可供使用，詳細可參考 [常見 CNI (Container Network Interface) Plugin 介紹 - hwchiu][7]

而預設上，Red Hat OpenShift 提供三個 CNI Plugin 可供使用
1. ovs-subnet: 單純 L2，類同 flat network
2. ovs-multitenant: 可針對不同的 namespace/project 進行隔離
3. ovs-networkpolicy: 控制細粒度最細，可以使用 `NetworkPolicy` object，類同於寫 iptables rules 一樣

當然如果遠遠覺得這三個功能不符合需求的話，還可以介接第三方廠商的 CNI Plugin，譬如 Juniper Contrail、VMWare NSX-T 等專業網路廠商的方案進行虛實整合 (Overlay + Underlay Integration)

### 南向: Egress Traffic

Egress 顧名思義可以了解到是從 OpenShift 的 Pod 向外去存取外部服務 (例: Database)，在沒有任何限制之下，在 OpenShift 底下的任何 Pod 當然是可以隨意地去存取外部服務，以下圖表接節錄 [Network Security for Apps on OpenShift - Red Hat Summit][8]

![](/images/egress-network.png)

試想若你的外部服務有需要為了資安保護進行存取限制，而你的 Compute Node 卻有複數台以上的話，是不是要特別把這些 Compute Node 的 IP 收集起來後，在外部服務中設定白名單開啟? 倘若環境又是經常性變動的話，是不是每次變動都要進行一次修改呢? 所以這時候就會有針對 Egress Traffic 的三種做法

1. OpenShift Egress Firewall
2. OpenShift Egress Routers
3. Egress static IP

![](/images/egress-firewall.png)
![](/images/egress-router.png)
![](/images/egress-static.png)

其中 OpenShift Egress Routers 的技術細節跟上面所討論的 Ingress Controller: OpenShift Router 是很類似的，而模式上也有三種做法可以選擇，分別是 Redirect、HTTP Proxy、DNS Proxy

分類 | Redirect mode | HTTP Proxy mode | DNS Proxy mode
---|---|---|---
Support Protocols | TCP/UDP | HTTP/HTTPS | TCP
Image | ose-pod | ose-egress-http-proxy | ose-egress-dns-proxy
Technology | Netfilter rules | Squid | HAProxy


## Summary

無論是 Kubernetes / OpenShift 一般在討論容器平台網路的不外乎都是在討論 Pod / Service / NodePort 等等的應用，但實際上使用的時候會蠻推薦用資料流 (Data Flow) 的觀點下去做每一個點的討論，這樣在技術選型上才不會思考太久，當然整體網路考慮不會只有單單這一篇文章就可以全部講完的，其實有非常多細節可以做探討，尤其是如何跟現有網路做對接、防火牆環境隔離等等都得一併考慮進去。

希望這篇能幫助大家用高視角的方式來理解整個 Red Hat OpenShift v3.11 網路流量走向

## Appendix: 針對 OpenShift v3 版本之 F5 設定

![](/images/ocp3_f5.png)


## References
- [Cisco Data Center Spine-and-Leaf Architecture: Design Overview White Paper][1]
- [What is Ingress? - Kubernetes][2]
- [Available router plug-ins - OpenShift Docs v3.11][3]
- [Routes - OpenShift Docs v3.11][4]
- [OpenShift SDN - OpenShift Docs v3.11][5]
- [從 Red Hat OpenShift 外部訪問 Services 或 Pods][6]
- [常見 CNI (Container Network Interface) Plugin 介紹 - hwchiu][7]
- [Network Security for Apps on OpenShift - Red Hat Summit][8]
- [Controlling Egress Traffic - OpenShift Docs v3.11][9]

[1]: https://www.cisco.com/c/en/us/products/collateral/switches/nexus-7000-series-switches/white-paper-c11-737022.html
[2]: https://kubernetes.io/docs/concepts/services-networking/ingress/#what-is-ingress
[3]: https://docs.openshift.com/container-platform/3.11/architecture/networking/assembly_available_router_plugins.html#architecture-additional-concepts-router-plugins
[4]: https://docs.openshift.com/container-platform/3.11/architecture/networking/routes.html
[5]: https://docs.openshift.com/container-platform/3.11/architecture/networking/sdn.html
[6]: https://blog.pichuang.com.tw/20190321-reach-out-services-and-pods-from-outside-into-openshift/
[7]: https://www.hwchiu.com/cni-compare.html
[8]: https://www.redhat.com/files/summit/session-assets/2018/Network-security-for-apps-on-OpenShift.pdf
[9]: https://docs.openshift.com/container-platform/3.11/admin_guide/managing_networking.html#admin-guide-controlling-egress-traffic