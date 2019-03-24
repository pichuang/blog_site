layout: post
title: 從 Red Hat OpenShift 外部訪問 Services 或 Pods
author: Phil Huang
tags:
  - redhat
  - openshift
  - router
  - services
  - pods
categories: []
date: 2019-03-21 14:16:00
---
Red Hat OpenShift v3.11 架構裡網路主要分為三大塊:
1. Hosts subnet
2. Service subnet
3. Pod subnet

<!--more-->

Host subnet 毫無懸念指的就是主機所使用的 IP，也就是對外可以接觸到主機的網路; 而 Service 和 Pods subnet 在 OpenShift 裡面是一個虛擬概念，正常狀況下外部網路無法透過 Pod IP/Port 和 Service IP/Port 接觸到服務

那如果我想要讓 Pod 或 Service IP 可以被外面接觸到呢? 以下分為兩個層級來討論:
1. Pod mapping to host
2. Service mapping to host

## Pod port Mapping to Host port

### 使用 `hostNetwork=true`

若沒有特別指定的話，預設 `hostPort` 等於 `containerPort`，意思是說 Pod 裡面的容器開了什麼 Port 就會直接對應到 host 上面的 Port，而 Pod IP 則會理所當然地等於 Host IP

外面訪問服務的 IP/Port，請求會直接轉到和 Pod 相同的 IP/Port ，途中不經過 iptables 及 service 處理

優點: 直接可以從外面直接接觸到 Pod 層級服務，網路轉發效率會比 nodePort 好  
缺點: 不能在同一台主機 (Host) 開啟第二個相同的 Pod，因為會有 Port 衝突的問題，也不能跨主機建立服務

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: influxdb
spec:
  hostNetwork: true <----
  containers:
    - name: nginx
      image: nginx
```

### 使用 `hostPort`

概念跟前者很像，差異會在可以指定特定的 `hostPort` 對應到 `containerPort`，但並不需要全部的 Pod Port 都跟 Host 做對應，可以有限度的開放出去

優點: 直接可以從外面直接接觸到 Pod 層級服務
缺點: 不能在同一台主機 (Host) 開啟第二個相同的 Pod，因為會有 Port 衝突的問題，也不能跨主機建立服務

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
      - containerPort: 80
        hostPort: 8080 <----
```



## Service port Mapping to Host port

### 使用 `nodePort`

廣泛應用的服務揭露方式，在預設情況下，每啟動一個 Service 都會搭配一個 `ClusterIP`，而這個 `ClusterIP` 僅能在同一個集群內部被訪問，外面不能接觸到，倘若想要讓他被接觸的話，就要指定 `nodePort` 對外揭露

hostPort 跟 nodePort 的差異在，前者是針對單一主機裡的單一容器進行設定，後者則是以一個集群來看待，服務可以跨主機

優點: 可跨主機，直接可以從外面直接接觸到 Service 層級服務，且能透過所產生的 ClusterIP 對後端的 Pod 做負載均衡
缺點: 所有集群的 Port 都會同時被開起來，讓外部做接觸

```yaml
---
kind: Service
apiVersion: v1
metadata:
  name: influxdb
spec:
  type: NodePort
  ports:
    - port: 8086
      nodePort: 30000 <---
  selector:
    name: influxdb
```

建議可以搭一個外部 Reverse Proxy 來做使用，譬如像是 F5 / A10 / HAProxy

### 使用 Ingress Controller: OpenShift Router

綜合 Pod 跟 Service 的優點的改良作法，也是普遍建立對外服務時的最佳實踐。你不需要自己對所有的 Service 去新增 `nodePort`，而可以直接在 OpenShift 上建立一個 Reverse Proxy 的服務，直接進行對外服務揭露及對內服務負載平衡，依據不同的 Ingress Controller 實作，通常功能都會包含 HTTP 路徑揭露、Session Sticky、SSL Transparent/Re-encrypt/Terminated 等等負載平衡的功能

OpenShift Router 是採用 `hostnetwork` 的方式建立，所以每一台主機至多只能安裝一個，同時 80 / 443 /1936 這三個 Port 會被預設佔走

想了解於 OpenShift 對於 Ingress Controller 的實作 `Router` 可以參考 [Router Overview - OpenShift v3.11 Docs][5]

### 使用 Service Type: `LoadBalancer`
這個功能僅限於公有雲，如果是 on-premise 的就只能自己實作或者是不要使用

## Summary
在 OpenShift 的環境之下

1. 對於 http/https 的服務，對外透過 Ingress Controller: OpenShift Router 揭露 FQDN 讓外面使用
2. 對於非 http/https 的服務，如 mysql，有下列兩種做法:
  1. 若不需要提供對外服務，僅使用 `ClusterIP` 在內部進行使用
  2. 若需要提供對外服務，有下列兩種做法:
    1. Single Instance，不需要在多個 node 上面運行，使用 `hostnetwork` 或 `hostPort`
    2. Multiple Instance，需要在多個 node 上面運行，做到 scale-out 的效果，則使用 `nodePort` 為佳


## References
- [kubernetes学习记录（3）——集群外部访问Pod或Service][1]
- [Kubernetes 網路是如何運作? Part 1- Erhwen Kuo][2]
- [Kubernetes 網路是如何運作? Part 2- Erhwen Kuo][8]
- [从外部访问Kubernetes中的Pod - JimmySong][3]
- [Accessing Kubernetes Pods from Outside of the Cluster][4]
- [Router Overview - OpenShift v3.11 Docs][5]
- [深度理解：Openshift端口方式全解析 - 大衛分享][6]
- [OpenShift 网络分析-(容器网络选型和方案建议) - 大衛分享][7]

[1]: https://blog.csdn.net/huqigang/article/details/76428017
[2]: https://www.slideshare.net/erhwenkuo/cncf-k8snetworkpart1/erhwenkuo/cncf-k8snetworkpart1
[3]: https://jimmysong.io/kubernetes-handbook/guide/accessing-kubernetes-pods-from-outside-of-the-cluster.html
[4]: http://alesnosek.com/blog/2017/02/14/accessing-kubernetes-pods-from-outside-of-the-cluster/
[5]: https://docs.openshift.com/container-platform/3.11/install_config/router/index.html
[6]: https://cloud.tencent.com/developer/article/1101219
[7]: https://cloud.tencent.com/developer/article/1375943
[8]: https://www.slideshare.net/erhwenkuo/cncf-k8snetwork02-137938815
