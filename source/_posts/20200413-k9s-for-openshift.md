---
layout: post
title: 用 k9s 操作 OpenShift 4
author: Phil Huang
toc: true
tags:
  - openshift4
  - k9s
categories:
  - openshift
date: 2020-04-13 23:53:06
udpated: 2020-04-13 23:53:06
---

OK...社群損友們跟我講了一個專案叫做 `k9s`，雖然不知道在幹嘛的，但秉持 OpenShift 完完全全就是個 Kuberentes 上游的精神直接裝裝看

![](https://k9scli.io/assets/k9s.png)

<!--more-->

## k9s

> Kubernetes CLI To Manage Your Clusters In Style!

k9s 這個專案提供了一個 Terminal UI 來進行 Kubernetes 集群的管理。此專案目標主要是要簡化 Kubernetes 導航、觀察和管理放在上面的雲原生應用程式，並且持續監視 Kubernetes 的更改並提供一些指令可以讓你更方便管理集群

## 環境資訊

- Red Hat OpenShift 4.3.9 (Kubernetes v1.16.2)
- k9s v0.19.1
- Red Hat Enterprise Linux 7.7 as bastion server

## 如何安裝?

相當簡單，去 `https://github.com/derailed/k9s/releases` 選個版本，這邊選擇是 `v0.19.1`

```bash
$ wget https://github.com/derailed/k9s/releases/download/v0.19.1/k9s_Linux_x86_64.tar.gz
$ tar zxvf k9s_Linux_x86_64.tar.gz
$ mv k9s /usr/bin/
$ k9s
```

沒了 = =

## 畫面截圖

我只能說相當潮啊...

### Node 截圖
![](/images/k9s-node.png)

![](/images/k9s-node-1.png)

### Pods 截圖
![](/images/k9s-pods.png)

### Namespace 截圖
![](/images/k9s-namespace.png)

### Pulses 截圖
![](/images/k9s-pulses.png)

### Xray 截圖
![](/images/k9s-xray.png)

## 結語

免責聲明一下，這個是 k9s 是社群專案，Red Hat 沒提供維護和技術支援，只是抓來試試而已

## References
- [k9s - Kubernetes CLI To Manage Your Clusters In Style!][1]

[1]: https://k9scli.io/