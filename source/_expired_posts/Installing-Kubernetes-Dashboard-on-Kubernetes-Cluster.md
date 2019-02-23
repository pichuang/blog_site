---
layout: post
title: Installing Kubernetes Dashboard on Kubernetes Cluster
date: 2017-05-24 14:03:38
updated: 2017-05-24 14:03:38
category:
- Infra
- Container
tags:
- k8s
- kubernetes-dashboard
---

## Objectives
- Install [k8s dashboard][2] on Kubernetes cluster

## Prerequisites
- Master: k8s-1.on.ec
    - Ubuntu 16.04
    - 192.168.100.52
- Worker: k8s-2.on.ec
    - Ubuntu 16.04
    - 192.168.100.53
- Worker: k8s-3.on.ec
    - Ubuntu 16.04 
    - 192.168.100.54
- [Installing Kubernetes on Linux with kubeadm][1]

## Instructions
### Install the latest stable release
```bash
kubectl create -f https://git.io/kube-dashboard
```

### Proxy
```bash
kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^*$'
```
- [StackOverflow - kubectl proxy unauthorized when accessing from another machine][3]
- [Solution][4]

## Reference
- [Installing Kubernetes on Linux with kubeadm][1]


[1]: http://blog.pichuang.com.tw/Installing-Kubernetes-on-Linux-with-kubeadm/#more
[2]: https://github.com/kubernetes/dashboard
[3]: https://stackoverflow.com/questions/42095142/kubectl-proxy-unauthorized-when-accessing-from-another-machine
[4]: https://github.com/kubernetes/dashboard/issues/692#issuecomment-264619451