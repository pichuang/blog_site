layout: post
title: 為什麼我佈署的 Kubernetes 服務不會動!? 個人除錯思路分享
author: Phil Huang
date: 2021-11-29 12:55:32
tags:
  - kubernetes
categories:
  - kubernetes
toc: true
---

某週，下班前臨時被 S 學姊抓去看了一個 Kubernetes 服務為何不會正常作動，故身為學弟的我出了篇技術文希望記錄一下相關除錯思路給各位參考

> 友人 A：覺得比搞 Kubernetes 還難的事情，大概就是談戀愛這件事了
友人 B：Kubernetes 起碼跟他告白 (a.k.a. 聲明式宣告, Declarative)，它會最大滿足你期望狀態 (Desired State)，但跟妹子告白可能會被抓去警察局，順便送你一張跟騷法
友人 C：這樣想想 Kubernetes 好像還比較和藹可親

<!--more-->

![](https://learnk8s.io/a/a-visual-guide-on-troubleshooting-kubernetes-deployments/troubleshooting-kubernetes.zh_cn.v2.png)

首先，很久以前 [Learnk8s A visual guide on troubleshooting Kubernetes deployments][2] 有為了廣大受到 Kubernetes 佈署(荼毒)失敗之苦的朋友出了一份 Troubleshooting Kubernetes deployments 流程圖 [簡中版][3] / [英文版][4]，我個人覺得很具備參考價值，內文主要 3 大階段是：

1. 確認 Pod 運作正常 (約略佔比 50%)
2. 確認 Service 運作正常 (約略佔比 25%)
3. 確認 Ingress 運作正常 (約略佔比 25%)

除了確認 Pod 運作正常是比較偏向 App 開發及程式佈署議題，其他兩個絕大部分都跟網路脫不了太大關係，所以我這邊基於該文對於網路除錯，包含 DNS、IP、HTTP 測試方式進行一些個人補充建議

## 除錯前的基礎知識: Pod

Pod，是實際上你的容器映像檔運行在 Kubernetes 的最小單位，裡面可以是 1 個或多個 Containers 在裡面，而同一個 Pod 裡面會共享同一個 Linux Namespace 的資源，包含 PID / NID 等。於本篇文章內比較重要的是同一個 Pod 裡面的 Linux Network Namesapce 的網路共享能力，意指每 1 個 Pod ，無論有多少個 Containers 在裡面運行，都會共享於該 Kubernetes 叢集內唯一的 IP，也就是 `Pod IP`

## 一切的起點，1 個 Debug Container

首先，你需要先有一個已經封裝好`具備除錯工具`的容器映像檔，後面簡稱 `Debug Container`

這個 Debug Container 的來源，主要是 2 個來源，其一是可以是採用社群善心人士編譯好的容器映像檔，如 `docker.io/nicolaka/netshoot`等，或其二是隨自家需求，從 `Base Image` 自行安裝、打包容器映像檔，如 `quay.io/pichuang/debug-container` 等，保持容器映像檔可控、可自行維護等好處

本文採用 `docker.io/nicolaka/netshoot` 作為主要 Debug Container 的基礎，詳細包含之工具可參考 [nicolaka/netshoot][5]

把這個 Debug Container 運行在 Kubernetes 上後，會以 Kubernetes Pod 的形式運作於上，所以也會有 1 個 Pod IP 可以拿來跟其他 Pod 做交叉比對使用，所以身為一個 Kubernetes 使用者，有一個好上手的 Debug Container 是一個很合理的事情

## 運行 Debug Container 之 2 種方式

至於運行方式是使用 kubectl 進行操作，以`指定 namespace 為角度`出發進行操作，主要有下列 2 種作法但不限於：

1. 於指定 Namespace 執行 Debug Container
```bash
#
# Running debug container within the Kubernetes Namespace on Any Nodes
#
$ kubectl run -n default debug-container --rm -i --tty --image docker.io/nicolaka/netshoot -- /bin/bash
```

2. 於指定 Namespace 和指定 Node 執行 Debug Container
```
#
# Running debug container within the Kubernetes Namespace on Specific Node (ex: tkg-worker-1)
#
# List of all Kubernetes nodes
$ kubectl get nodes
NAME                           STATUS   ROLES                  AGE   VERSION
tkgm14-control-plane-pv76m     Ready    control-plane,master   38d   v1.21.2+vmware.1
tkgm14-md-0-66f85bf86b-wfjt7   Ready    <none>                 38d   v1.21.2+vmware.1

$ kubectl run -n default debug-container --rm -i --tty --overrides='{ "apiVersion": "v1", "spec": {"kubernetes.io/hostname":"tkgm14-control-plane-pv76m"}}' --image docker.io/nicolaka/netshoot -- /bin/bash
```

正常狀況下，你應該直接使用方法 1 就可以找出很多問題了，除非你懷疑特定節點怪怪的，再使用方法 2

## 從 Pod 出發的 5 種流量走法，5 種除錯方向

整體上，Kubernetes 內到外網路流量，約略可以條列成為下列 5 種流量走法但不限：

1. Container to Container
2. Pod to Pod in the same nodes
3. Pod to Pod across different nodes
4. Pod to Service
5. Pod to Internet

詳細可參考小弟於香港開源人年會之分享 [20200612, How I do troubleshooting on container, more than docker?, HKOSCon 2020][1]

因為實際上所有實際服務都是被包裝在 Pod 裡面運行，故當遇到服務不通或異常的時候，想要快速進行網路流量除錯，可以以 `Pod IP 角度且以 Kubernetes Service 常見分類為主`之 4 種除錯方向:

1. Pod IP to Pod IP
2. Pod IP to ClusterIP
3. Pod IP to NodePort IP
4. Pod IP to LoadBalancer IP (雲端常見，或特定 LB 軟體有提供整合能力，如 AVI Networks)

另外還有 1 個 Kubernetes Service 編制外但常見的 Ingress Controller

5. Pod IP to Ingress Hosts (地端常見，如 Contour、Nginx)

### 收集除錯前之必要資訊

要開始使用 Debug Container 進行除錯之前，務必要先收集好相關資訊 `kubectl get pods,svc,nodes -owide`，主要是要有 IP 和名字之其相關資訊，如以下操作：

```bash
$ kubectl get pods,svc,nodes,ingress -owide
NAME                                    READY   STATUS    RESTARTS   AGE   IP            NODE                           NOMINATED NODE   READINESS GATES
pod/tanzu-deployment-5769f6c4df-2fl28   1/1     Running   0          60m   100.96.1.33   tkgm14-md-0-66f85bf86b-wfjt7   <none>           <none>
pod/tanzu-deployment-5769f6c4df-8jj87   1/1     Running   0          60m   100.96.1.35   tkgm14-md-0-66f85bf86b-wfjt7   <none>           <none>
pod/tanzu-deployment-5769f6c4df-nlcg8   1/1     Running   0          60m   100.96.1.34   tkgm14-md-0-66f85bf86b-wfjt7   <none>           <none>

NAME                     TYPE         CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE   SELECTOR
service/kubernetes       ClusterIP    100.64.0.1      <none>          443/TCP          38d   <none>
service/tanzu-service    NodePort     100.70.88.107   <none>          3000:30390/TCP   60m   app=tkgm-deployment
service/tanzu-lb-service LoadBalancer 100.70.88.108   172.18.30.100   3000:30340/TCP   60m   app=tkgm-deployment

NAME                                STATUS   ROLES                  AGE   VERSION            INTERNAL-IP    EXTERNAL-IP    OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
node/tkgm14-control-plane-pv76m     Ready    control-plane,master   38d   v1.21.2+vmware.1   172.18.30.38   172.18.30.38   Ubuntu 20.04.2 LTS   5.4.0-77-generic   containerd://1.4.6
node/tkgm14-md-0-66f85bf86b-wfjt7   Ready    <none>                 38d   v1.21.2+vmware.1   172.18.30.39   172.18.30.39   Ubuntu 20.04.2 LTS   5.4.0-77-generic   containerd://1.4.6
```

### 1. Pod IP to Pod IP Traffic

中等難度的測試，包含 Ping / Curl 都要能獲得預期的反應，如果有問題的話，可以參閱 [Learnk8s A visual guide on troubleshooting Kubernetes deployments][2] 進行 Pod 內部運行的問題除錯，這時候下 `kubectl logs <pod name>` 和 `kubectl describe pod <pod name>` 應該都會有些蛛絲馬跡

```bash
# Run debug container
$ kubectl run -n default debug-container --rm -i --tty --image docker.io/nicolaka/netshoot -- /bin/bash

# List of Routing Table
bash-5.1# ip r
default via 100.96.1.1 dev eth0
100.96.1.0/24 dev eth0 proto kernel scope link src 100.96.1.36

#
# Ping
# Test Connectivity
bash-5.1# ping 100.96.1.35
bash-5.1# ping 100.96.1.34
bash-5.1# ping 100.96.1.33
PING 100.96.1.33 (100.96.1.33) 56(84) bytes of data.
64 bytes from 100.96.1.33: icmp_seq=1 ttl=64 time=0.875 ms

# Curl web page
bash-5.1# curl 100.96.1.33:3000
bash-5.1# curl 100.96.1.34:3000
bash-5.1# curl 100.96.1.35:3000
Hello World!
```

### 2. Pod IP to ClusterIP Traffic

最核心的測試，首先你需要先對 Kubernetes Services 運作要有一定程度理解，詳細可以參考 [Hwchiu - Kubernetes What Is Service?][6] 一文，在這個階段主要測試目標是，確認可以透過 Kubernetes Cluster IP 獲得 Pod 的內容

```bash
# Run debug container
$ kubectl run -n default debug-container --rm -i --tty --image docker.io/nicolaka/netshoot -- /bin/bash

# List of Routing Table
bash-5.1# ip r
default via 100.96.1.1 dev eth0
100.96.1.0/24 dev eth0 proto kernel scope link src 100.96.1.37

#
# DNS Lookup
# https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
#
# nslookup <service name>.<namespace name>
# nslookup <service name>.<namespace name>.svc.cluster.local
bash-5.1# nslookup tanzu-service.default
bash-5.1# nslookup tanzu-service.default.svc.cluster.local
Server:		100.64.0.10
Address:	100.64.0.10#53

Name:	tanzu-service.default.svc.cluster.local
Address: 100.70.88.107

#
# Ping (FAILED)
# Ping doesn't work with service's cluster IPs like 100.70.88.107, as it is a virtual IP
# You should be able to ping a specific pod, but no a service.
bash-5.1# ping 100.70.88.107
PING 100.70.88.107 (100.70.88.107) 56(84) bytes of data.
-- Pending --

#
# Curl
#
# curl <service name>.<namespace name>:<svc port>
# curl <service name>.<namespace name>.svc.cluster.local:<svc port>
# curl <cluster ip>:<svc port>>

# Curl web page (OK)
bash-5.1# curl tanzu-service.default:3000
Hello World!
bash-5.1# curl tanzu-service.default.svc.cluster.local:3000
Hello World!
bash-5.1# curl 100.70.88.107:3000
Hello World!

# Curl Kubernetes API Service (OK)
bash-5.1# curl -o /dev/null -s -w "%{http_code}\n" --insecure https://kubernetes.default:443
403
bash-5.1# curl -o /dev/null -s -w "%{http_code}\n" --insecure https://kubernetes.default.svc.cluster.local:443
403
bash-5.1# curl -o /dev/null -s -w "%{http_code}\n" --insecure https://100.64.0.1:443
403
```

### 3. Pod IP to NodePort IP

執行一次 `2. Pod IP to Cluster IP` 測試以外，還需要另外確認對外服務揭露的部份，主要以 Node IP 為基礎

```bash
# Run debug container
$ kubectl run -n default debug-container --rm -i --tty --image docker.io/nicolaka/netshoot -- /bin/bash

# List of Routing Table
bash-5.1# ip r
default via 100.96.1.1 dev eth0
100.96.1.0/24 dev eth0 proto kernel scope link src 100.96.1.40

#
# Ping
# Test Connectivity
# Pod IP -> Node IP
#
bash-5.1# ping 172.18.30.39
PING 172.18.30.39 (172.18.30.39) 56(84) bytes of data.
64 bytes from 172.18.30.39: icmp_seq=1 ttl=64 time=0.463 ms

#
# Curl
# curl <node ip>:<node port>
bash-5.1# curl 172.18.30.39:30390
Hello World!
```

### 4. Pod IP to LoadBalancer IP

執行一次 `2. Pod IP to Cluster IP` 測試以外，還需要另外確認對外服務揭露的部份，主要以 Loadbalacner IP 為基礎

```bash
# Run debug container
$ kubectl run -n default debug-container --rm -i --tty --image docker.io/nicolaka/netshoot -- /bin/bash

# List of Routing Table
bash-5.1# ip r
default via 100.96.1.1 dev eth0
100.96.1.0/24 dev eth0 proto kernel scope link src 100.96.1.40

#
# Ping
# Test Connectivity
# Pod IP -> LoadBalancer IP
#

bash-5.1# ping 172.18.30.100
PING 172.18.30.100 (172.18.30.100) 56(84) bytes of data.
64 bytes from 172.18.30.100: icmp_seq=1 ttl=64 time=0.3 ms

#
# Curl
# curl <LoadBalancer ip>:<LB port>
bash-5.1# curl 172.18.30.100:30340
Hello World!
```

### 5. Pod IP to Ingress Hosts

執行一次 `2. Pod IP to Cluster IP` 測試以外，還需要另外確認對外服務揭露的部份，主要以 Ingress Hosts 為基礎

```bash
#
$ kubectl get ingress
NAME                        HOSTS                                                   ADDRESS       PORTS   AGE
name-virtual-host-ingress   tutum.training.boxboat.io,jwilder.training.boxboat.io   172.17.0.58   80      9m24s
path-ingress                training.boxboat.io                                     172.17.0.58   80      9m2s

# Run debug container
$ kubectl run -n default debug-container --rm -i --tty --image docker.io/nicolaka/netshoot -- /bin/bash

# List of Routing Table
bash-5.1# ip r
default via 100.96.1.1 dev eth0
100.96.1.0/24 dev eth0 proto kernel scope link src 100.96.1.40

#
# Ping
# Test Connectivity
#

bash-5.1# ping 172.17.0.58
PING 172.17.0.58 (172.17.0.58) 56(84) bytes of data.
64 bytes from 172.17.0.58: icmp_seq=1 ttl=63 time=0.706 ms

# Curl
# curl <HOSTS>:<PORTS>
bash-5.1# curl training.boxboat.io:80
```

## Pod to Ingress v.s. Pod to LoadBalancer

如果你有仔細看，會發現 Kubernetes LoadBalacner 跟 Kubernetes Ingress 除錯起來套路很像，理由是他們都是實際上接收到 User Traffic 最前線的服務，至於想要了解兩者網路流量差異可參考 [Kubernetes NodePort vs LoadBalancer vs Ingress? When should I use what?][8]，及 [Hwchiu - Introduction to Kubernetes Ingress (Nginx)][9]

> Kubernetes LoadBalacner 是 Kubernetes Service 其中一種類型，外面必然會有一個獨立於 Kubernetes 容器平台外部的 External LB 存在
Kubernetes Ingress 並不是 Kubernetes Service，它受到特定 Ingress Controller 管理，建立於 Kubernetes 容器平台內部

如果你沒有什麼特定 LB 可以提供 Kubernetes Service 呼叫使用的話，那基本上你大多數都是用 Kubernetes Ingress 作為主要使用，但兩者也可以並用就是了，盡可能不在同一個節點上即可，因為 hostNetwork 有蠻高機會會衝 Port，如 80/443 等

## 常見問題

### Q1: deployment.yml 裡的 Kubernetes API 改版了，該怎麼無縫地修正
```bash
$ kubectl apply -f deployment.yml
error: unable to recognize "deployment.yml": no matches for kind "Deployment" in version "extensions/v1beta1"
```

A1: 1. 找到正確的 API 名字
```bash
$ kubectl api-resources |grep deployments
deployments                       deploy       apps/v1                                              true         Deployment
machinedeployments                md           cluster.x-k8s.io/v1alpha3                            true         MachineDeployment
```
2. 修正 deployment.yaml 內的檔案，從 `apiVersion: extensions/v1beta1` 到 `apiVersion: apps/v1`，重新執行
3. 沒意外應該還會有錯誤，再根據錯誤訊息，進行欄位的增刪修

### Q2: 如果你是 Kubernetes Platform Administrator 想要對特定節點之作業系統除錯那該怎麼辦?

A2: 佈署的時候，多一個 `"hostNetwork": true` 參數即可

```bash
$ kubectl run -n default debug-container --rm -i --tty --overrides='{ "apiVersion": "v1", "spec": {"kubernetes.io/hostname":"tkgm14-md-0-66f85bf86b-wfjt7", "hostNetwork": true}}' --image nicolaka/netshoot -- /bin/bash
bash-5.1# ip a show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:9e:85:bb brd ff:ff:ff:ff:ff:ff
    inet 172.18.30.39/24 brd 172.18.30.255 scope global dynamic eth0
       valid_lft 1059832sec preferred_lft 1059832sec
    inet6 fe80::250:56ff:fe9e:85bb/64 scope link
       valid_lft forever preferred_lft forever
```



[1]: https://speakerdeck.com/pichuang/how-do-i-troubleshooting-on-container-more-than-docker
[2]: https://learnk8s.io/troubleshooting-deployments
[3]: https://learnk8s.io/a/a-visual-guide-on-troubleshooting-kubernetes-deployments/troubleshooting-kubernetes.zh_cn.v2.pdf
[4]: https://learnk8s.io/a/a-visual-guide-on-troubleshooting-kubernetes-deployments/troubleshooting-kubernetes.en_en.v2.pdf
[5]: https://github.com/nicolaka/netshoot
[6]: https://www.hwchiu.com/kubernetes-service-i.html
[7]: https://www.hyscale.io/wp-content/uploads/2020/07/HS-Troubleshooting-guide-300KB.pdf
[8]: https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0
[9]: https://www.hwchiu.com/ingress-1.html