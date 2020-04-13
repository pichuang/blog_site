---
layout: post
title: Red Hat OpenShift v4 東西南北向網路流
author: Phil Huang
toc: true
tags:
  - openshift4
  - openshift
categories:
  - openshift
date: 2020-04-13 01:18:32
udpated: 2020-04-13 01:18:32
---

去年 2019/04 的時候有釋放出 [Red Hat OpenShift v3.11 東西南北向網路探討 - Phil Huang][1] 一文，過了一年後，Red Hat OpenShift 4 出了，那兩個不同大版本的網路設計有什麼樣子差異呢?

{% youtube DMb4L92Z7z8 %}

<!--more-->

## Red Hat OpenShift 4 東西南北向網路流

首先先來張針對 v4 的架構圖

![](/images/ocp4-network-traffic.png)

以 Red Hat OpenShift 為核心視角，可以分為三段網路流

1. 北向網路流 Ingress Traffic
2. 東西向網路流
3. 南向網路流 Egress Traffic

另外，以上這些流量，實際上都是透過每一台節點上的 `Node IP` 進行溝通的，也就是你只需要提供這些節點`一個 IP` 即可使用，意指`不需要指派多網卡於單一節點上`，那具體是哪個 IP 呢? 主要是看 `INTERNAL-IP` 那個欄位的 IP

```bash
$ oc get nodes -o wide
NAME              STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                                                       KERNEL-VERSION                CONTAINER-RUNTIME
compute-0         Ready    worker   26d   v1.16.2   10.0.97.4     10.0.97.4     Red Hat Enterprise Linux CoreOS 43.81.202003230848.0 (Ootpa)   4.18.0-147.5.1.el8_1.x86_64   cri-o://1.16.3-28.dev.rhaos4.3.git9aad8e4.el8
compute-1         Ready    worker   26d   v1.16.2   10.0.97.5     10.0.97.5     Red Hat Enterprise Linux CoreOS 43.81.202003230848.0 (Ootpa)   4.18.0-147.5.1.el8_1.x86_64   cri-o://1.16.3-28.dev.rhaos4.3.git9aad8e4.el8
control-plane-0   Ready    master   26d   v1.16.2   10.0.97.1     10.0.97.1     Red Hat Enterprise Linux CoreOS 43.81.202003230848.0 (Ootpa)   4.18.0-147.5.1.el8_1.x86_64   cri-o://1.16.3-28.dev.rhaos4.3.git9aad8e4.el8
control-plane-1   Ready    master   26d   v1.16.2   10.0.97.2     10.0.97.2     Red Hat Enterprise Linux CoreOS 43.81.202003230848.0 (Ootpa)   4.18.0-147.5.1.el8_1.x86_64   cri-o://1.16.3-28.dev.rhaos4.3.git9aad8e4.el8
control-plane-2   Ready    master   26d   v1.16.2   10.0.97.3     10.0.97.3     Red Hat Enterprise Linux CoreOS 43.81.202003230848.0 (Ootpa)   4.18.0-147.5.1.el8_1.x86_64   cri-o://1.16.3-28.dev.rhaos4.3.git9aad8e4.el8
```

### 北向: Ingress Traffic

Ingress 主要的目的就是讓外部使用者可以透過 Ingress Controller 以接觸到 Kubernestes 裡面的 Service，而在 Red Hat OpenShift 裡面的術語則是使用 Router 來做闡述

隨著各式各樣的服務都在 OpenShift 4 上都被實作成 Operator Framework，當然連 Ingress Controller 也不例外，詳細可以參考[Configuring ingress cluster traffic overview - OpenShift 4.3][9]

從外部連到到 OpenShift 上 Service 的方式主要有 4 種方式

1. 使用 OpenShift Router
2. 使用 Service Type: LoadBalancer
3. 使用 externalIngressIP
4. 使用 nodePort

詳細可以參考 [從 Red Hat OpenShift 外部訪問 Services 或 Pods][10]

### 東西向: East-West Traffic

在 Red Hat OpenShift 平台底下，無論是要跨 Pod / Service / Host 進行橫向的溝通，都是需要有 CNI (Cotainer Network Interface) 的實踐才能將不同主機之間的連線建立起來。CNI 本身是一套標準，實作上就會有百花齊放的專案可供使用，詳細可參考 [20190817 Container Bare Metal for Networking][4]

![](/images/ocp-multus.png)

預設安裝下，`Multus CNI` 會被安裝在 OpenShift 4 裡面，當然也可以不安裝把它關掉，請參考 [KB4850181: How to install OpenShift 4 without Multus in place][5]

而原生必須安裝的 OpenShift SDN CNI 主要有 2 個可以使用:

1. OpenShiftSDN (預設)
2. OVNKubernetes (4.5 GA)

其他第三方供應商的 CNI Plugin 就不細列了

而如果有需要實踐 [Multiple Networks][6] 則可以選擇 5 種 CNI Plugin 之一掛載

1. bridge
2. host-device
3. macvlan
4. ipvlan
5. [SR-IOV][7]

## 與 OpenShift 3.11 的網路差異

1. 於 4 代，Web Console 由 `Infra Node` 上的 `OpenShift Router` 提供，而不是透過 `Master Node` 提供
2. 於 4 代，OpenShift Router 的功能，放置在 `openshift-ingress-operator`，而不是放置在 `default`
3. 於 4 代，OpenShift CNI 提供 `OpenShiftSDN` 及 `OVNKubernetes` 和預設提供 `multus` 提供 Pod 多網卡能力

## 實務上在 OpenShift 4 的網路設定

實際上對應到 Red Hat OpenShift 4 的設定檔 [install-config.yaml][3]

```yaml install-config.yaml
apiVersion: v1
baseDomain: example.com
...omit...
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
...omit...
```

實際上，需要設定 3 大段網路，

1. Node Subnet: 主要給`北向`或`南向`溝通用
2. Service Subnet: 主要給`東西向`網路流用
3. Pod Subnet: Kubernetes Pod 實際上拿到的 IP，溝通主要都是使用 `Service Subnet` 進行溝通

其他詳細參數解釋請看 [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 2 - Cluster Network Operator][2]

## References
- [Red Hat OpenShift v3.11 東西南北向網路探討 - Phil Huang][1]
- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 2 - Cluster Network Operator][2]
- [Sample install-config.yaml file for bare metal - OpenShift 4.3][3]
- [20190817 Container Bare Metal for Networking][4]
- [KB4850181: How to install OpenShift 4 without Multus in place][5]
- [Understanding multiple networks - OpenShift 4.3][6]
- [About Single Root I/O Virtualization (SR-IOV) hardware networks][7]
- [Configuring ingress cluster traffic overview - OpenShift 4.3][9]
- [從 Red Hat OpenShift 外部訪問 Services 或 Pods][10]


[1]: https://blog.pichuang.com.tw/20190404-openshift-network-traffic-overview/
[2]: https://blog.pichuang.com.tw/20200403-openshift-with-coreos-part-2/#%E4%BA%86%E8%A7%A3%E7%95%B6%E4%B8%8B%E7%B6%B2%E8%B7%AF%E8%B3%87%E8%A8%8A-Cluster-Network-Operator
[3]: https://docs.openshift.com/container-platform/4.3/installing/installing_bare_metal/installing-bare-metal.html#installation-bare-metal-config-yaml_installing-bare-metal
[4]: https://speakerdeck.com/pichuang/20190817-container-bare-metal-for-networking
[5]: https://access.redhat.com/solutions/4850181
[6]: https://docs.openshift.com/container-platform/4.3/networking/multiple_networks/understanding-multiple-networks.html
[7]: https://docs.openshift.com/container-platform/4.3/networking/hardware_networks/about-sriov.html
[8]: https://docs.openshift.com/container-platform/4.3/networking/configuring_ingress_cluster_traffic/overview-traffic.html
[9]: https://docs.openshift.com/container-platform/4.3/networking/configuring_ingress_cluster_traffic/overview-traffic.html
[10]: https://blog.pichuang.com.tw/20190321-reach-out-services-and-pods-from-outside-into-openshift/