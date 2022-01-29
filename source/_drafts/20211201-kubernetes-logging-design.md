layout: post
title: Kubernetes Log 吐到哪裡去了?
author: Phil Huang
date: 2021-12-01 12:55:32
tags:
  - kubernetes
categories:
  - kubernetes
toc: true
---

<!--more-->


## 先備知識

如同[20211122_ Kubernetes 多叢集及單叢集架構選擇探討 - 何謂 1 個 Kubernetes 節點 (Node)][5]所介紹，最核心作業系統要麻就是 Linux 不麻就是 Windows，但前者的使用應該佔比是絕大宗的，所以本文以 Linux 作為基礎為主

## Kubernetes Logging 到底怎麼收集?

![](/images/kubernetes-logging.png)

基於官方 Kubernetes.io Kubernetes Logging [英文版][1] / [簡中版][2] 一文作為基礎，小弟換一個方向`以角色 (Roles)`的角度轉譯一下內容

以角色 -- Dev 和 Ops 來分類，主要兩個角色會關心的角度為以下
1. Application Logs
2. System Logs





## 常見問題

Q1: 如何用 kubectl 登入到任一節點上進行操作?
A1: 安裝 [kubectl-node-shell][4]，然後請謹慎使用

- 安裝過程
```bash
$ curl -LO https://github.com/kvaps/kubectl-node-shell/raw/master/kubectl-node_shell
$ chmod +x ./kubectl-node_shell
$ sudo mv ./kubectl-node_shell /usr/local/bin/kubectl-node_shell
```

- 登入過程
```bash
$ kubectl get nodes
NAME                                            STATUS   ROLES                  AGE    VERSION
tkg-workload-cluster-01-control-plane-hmp44     Ready    control-plane,master   200d   v1.20.5+vmware.1
tkg-workload-cluster-01-md-0-754c588789-cscwt   Ready    <none>                 200d   v1.20.5+vmware.1
tkg-workload-cluster-01-md-0-754c588789-rzpqb   Ready    <none>                 200d   v1.20.5+vmware.1

# Usage:
# Get standard bash shell
# kubectl node-shell <node>
#
# Execute custom command
# kubectl node-shell <node> -- echo 123
#
# Use stdin
# cat /etc/passwd | kubectl node-shell <node> -- sh -c 'cat > /tmp/passwd'
#
# Run oneliner script
# kubectl node-shell <node> -- sh -c 'cat /tmp/passwd; rm -f /tmp/passwd'

$ kubectl node-shell tkg-workload-cluster-01-md-0-754c588789-rzpqb
spawning "nsenter-mp4ztc" on "tkg-workload-cluster-01-md-0-754c588789-rzpqb"
If you don't see a command prompt, try pressing enter.
root@tkg-workload-cluster-01-md-0-754c588789-rzpqb:/# cat /etc/os-release |grep -i PRETTY_NAME
PRETTY_NAME="Ubuntu 20.04.2 LTS"
```


## References
- [Kubernetes.io - Logging Architecture English][1]
- [Kubernetes.io - Logging Architecture 簡中][2]
- [Tanzu Kubernetes Grid multi-cloud (TKGm) from the tkg Command Line Interface][3]
- [kubectl-node-shell][4]

[1]: https://kubernetes.io/docs/concepts/cluster-administration/logging/
[2]: https://kubernetes.io/zh/docs/concepts/cluster-administration/logging/
[3]: https://cormachogan.com/2020/07/06/tanzu-kubernetes-grid-from-the-tkg-command-line-interface/
[4]: https://github.com/kvaps/kubectl-node-shell
[5]: https://speakerdeck.com/pichuang/20211122-kubernetes-duo-cong-ji-ji-dan-cong-ji-jia-gou-xuan-ze-tan-tao?slide=9