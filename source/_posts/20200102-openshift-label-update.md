layout: post
title: Red Hat OpenShift 4 特定節點 Label 更新
author: Phil Huang
tags:
  - openshift4
  - openshift
categories:
  - openshift
date: 2020-01-02 10:45:00
---
以前的[演講 - 那些年的 OpenShift 3.11 容器平台技術選型_20190122
][2]有分享過，Red Hat OpenShift 對於節點的角色有分 3 種：

1. Master
2. Infra
3. Worker

![](/images/label-overview.png)

而 OpenShift 4 的安裝方式是預設不會有 Infa 節點的存在，僅會先有 Master 及 Worker，而需要特別設定 Label - 從 Worker 轉成 Infra 節點。本文將花點篇幅講一下轉法

<!--more-->


## Q: 如何辨識特定節點為何種角色?
看標籤 (Label)，對於 Red Hat OpenShift 來講，三個角色分別對應的標籤名為：

1. Master `node-role.kubernetes.io/master=""`
2. Infra `node-role.kubernetes.io/infra=""`
3. Worker `node-role.kubernetes.io/worker=""`

只要看著標籤，就知道該節點是什麼角色。除此之外，同一個節點可以擁有多個角色

## Q: 如何列舉所有節點名稱?
```bash
$ oc get nodes

NAME                                         STATUS   ROLES    AGE   VERSION
ip-10-0-135-207.us-east-2.compute.internal   Ready    master   23h   v1.14.6+9fb2d5cf9
ip-10-0-142-198.us-east-2.compute.internal   Ready    worker   22h   v1.14.6+9fb2d5cf9
ip-10-0-147-86.us-east-2.compute.internal    Ready    worker   22h   v1.14.6+9fb2d5cf9
ip-10-0-156-112.us-east-2.compute.internal   Ready    master   23h   v1.14.6+9fb2d5cf9
ip-10-0-172-190.us-east-2.compute.internal   Ready    master   23h   v1.14.6+9fb2d5cf9
ip-10-0-173-156.us-east-2.compute.internal   Ready    worker   22h   v1.14.6+9fb2d5cf9
```

## Q: 如何列舉所有節點的 Label 資訊?
```bash
$ oc get nodes --show-labels

NAME                                         STATUS   ROLES    AGE   VERSION             LABELS
ip-10-0-135-207.us-east-2.compute.internal   Ready    master   37h   v1.14.6+9fb2d5cf9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=m4.xlarge,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=us-east-2,failure-domain.beta.kubernetes.io/zone=us-east-2a,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-10-0-135-207,kubernetes.io/os=linux,node-role.kubernetes.io/master=,node.openshift.io/os_id=rhcos
ip-10-0-142-198.us-east-2.compute.internal   Ready    worker   37h   v1.14.6+9fb2d5cf9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=m5.2xlarge,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=us-east-2,failure-domain.beta.kubernetes.io/zone=us-east-2a,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-10-0-142-198,kubernetes.io/os=linux,node-role.kubernetes.io/worker=,node.openshift.io/os_id=rhcos
ip-10-0-147-86.us-east-2.compute.internal    Ready    worker   37h   v1.14.6+9fb2d5cf9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=m5.2xlarge,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=us-east-2,failure-domain.beta.kubernetes.io/zone=us-east-2b,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-10-0-147-86,kubernetes.io/os=linux,node-role.kubernetes.io/worker=,node.openshift.io/os_id=rhcos
ip-10-0-156-112.us-east-2.compute.internal   Ready    master   37h   v1.14.6+9fb2d5cf9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=m4.xlarge,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=us-east-2,failure-domain.beta.kubernetes.io/zone=us-east-2b,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-10-0-156-112,kubernetes.io/os=linux,node-role.kubernetes.io/master=,node.openshift.io/os_id=rhcos
ip-10-0-172-190.us-east-2.compute.internal   Ready    master   37h   v1.14.6+9fb2d5cf9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=m4.xlarge,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=us-east-2,failure-domain.beta.kubernetes.io/zone=us-east-2c,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-10-0-172-190,kubernetes.io/os=linux,node-role.kubernetes.io/master=,node.openshift.io/os_id=rhcos
ip-10-0-173-156.us-east-2.compute.internal   Ready    worker   37h   v1.14.6+9fb2d5cf9   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=m5.2xlarge,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=us-east-2,failure-domain.beta.kubernetes.io/zone=us-east-2c,kubernetes.io/arch=amd64,kubernetes.io/hostname=ip-10-0-173-156,kubernetes.io/os=linux,node-role.kubernetes.io/worker=,node.openshift.io/os_id=rhcos
```


## Q: 如何列舉特定節點的 Label 資訊?

以 `ip-10-0-142-198.us-east-2.compute.internal` 為例：

```bash
# Example: oc label --list node <node>
$ oc label --list node ip-10-0-142-198.us-east-2.compute.internal

beta.kubernetes.io/arch=amd64
failure-domain.beta.kubernetes.io/region=us-east-2
kubernetes.io/os=linux
node-role.kubernetes.io/worker=
beta.kubernetes.io/os=linux
failure-domain.beta.kubernetes.io/zone=us-east-2a
node.openshift.io/os_id=rhcos
kubernetes.io/arch=amd64
beta.kubernetes.io/instance-type=m5.2xlarge
kubernetes.io/hostname=ip-10-0-142-198
```

## Q: 如何列出所有角色為 worker 的節點?
```bash
$ oc get nodes -l node-role.kubernetes.io/worker

NAME                                         STATUS   ROLES    AGE   VERSION
ip-10-0-142-198.us-east-2.compute.internal   Ready    worker   37h   v1.14.6+9fb2d5cf9
ip-10-0-147-86.us-east-2.compute.internal    Ready    worker   37h   v1.14.6+9fb2d5cf9
ip-10-0-173-156.us-east-2.compute.internal   Ready    worker   37h   v1.14.6+9fb2d5cf9
```

## Q: 如何針對特定節點新增 Label 

以 `ip-10-0-142-198.us-east-2.compute.internal` 為例，新增 `infra` 標籤

```bash
# Example: oc label node <node> node-role.kubernetes.io/infra=""
$ oc label node ip-10-0-142-198.us-east-2.compute.internal node-role.kubernetes.io/infra=""

node/ip-10-0-142-198.us-east-2.compute.internal labeled
```

做完之後可以用 `oc get nodes` 或者是開 Web console 看一下狀態改變

![](/images/label-change-1.png)

![](/images/label-change-2.png)


## Q: 如何針對特定節點移除 Label?

以 `ip-10-0-142-198.us-east-2.compute.internal` 為例，移除 `worker` 標籤

```bash
# oc label node <node-name> node-role.kubernetes.io/worker-
$ oc label node ip-10-0-142-198.us-east-2.compute.internal node-role.kubernetes.io/worker-

node/ip-10-0-142-198.us-east-2.compute.internal labeled
```

做完之後可以用 `oc get nodes` 或者是開 Web console 看一下狀態改變

![](/images/label-remove-2.png)

![](/images/label-remove-1.png)

## Q: 如何把 OpenShift Router 搬到 Infra 節點上去?

要對 Operator 進行修改操作，不要直接修改 router 裡面的內容，會一直被 Operator 洗掉設定

```bash
oc patch ingresscontroller/default --type=merge -n openshift-ingress-operator -p '{"spec": {"nodePlacement":{"nodeSelector":{"matchLabels":{"node-role.kubernetes.io/infra": ""}}}}}'
```

如果是要搬其他的服務，也是以此類推，或者是參考 [Moving resources to infrastructure MachineSets][4]


## Q: 如果有除了 3 大角色以外的節點分類該怎麼辦?

假設你今天有 3 個節點，要準備跑特定的服務，譬如像是 Ceph，歸類角色為 `ceph-node`，依據 `同一個節點可以擁有多個角色` 的特性，可以執行如下指令：

```bash
# Example: oc label node <node> node-role.kubernetes.io/ceph-node=""
$ oc label node ip-10-0-150-37.us-east-2.compute.internal node-role.kubernetes.io/ceph-node=""

node/ip-10-0-150-37.us-east-2.compute.internal labeled
```

![](/images/label-ceph-node.png)


## References
- [KB 4287111 - Creating an Infra Node in OpenShift v4][3]
- [那些年的 OpenShift 3.11 容器平台技術選型_20190122][2]
- [OpenShift 4 - Creating infrastructure MachineSets][1]
- [Moving resources to infrastructure MachineSets][4]

[1]: https://docs.openshift.com/container-platform/4.2/machine_management/creating-infrastructure-machinesets.html
[2]: https://speakerdeck.com/pichuang/na-xie-nian-de-openshift-3-dot-11-rong-qi-ping-tai-ji-shu-xuan-xing-20190122?slide=11
[3]: https://access.redhat.com/solutions/4287111
[4]: https://docs.openshift.com/container-platform/4.2/machine_management/creating-infrastructure-machinesets.html#moving-resources-to-infrastructure-machinesets