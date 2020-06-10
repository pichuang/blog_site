---
layout: post
title: Red Hat OpenShift 4 的 etcd 之幾個你應該要知道的事之二
author: Phil Huang
toc: true
tags:
  - openshift4
  - openshift
  - etcd
categories:
  - openshift
date: 2020-05-29 10:15:40
udpated: 2020-05-29 10:15:40
---

承襲上文 [Red Hat OpenShift 4 的 etcd 之幾個你應該要知道的事之一][7]，繼續針對硬碟的部分進行調整，畢竟 etcd 對[硬碟][8] IOPS 相當要求，建議用 SSD 比較好

沒意外的，開場分享個 [Deploying OpenShift 4.4 to VMware vSphere 7][9]，整體上比較重要的是利用 VMware vSphere 7 裡面的 DRS (Dynamic Resource Scheduler) 及 NSX-T 的部分，之後有機會再說

{% youtube PdyNQXpYknI %}

<!--more-->

## etcd 節點硬碟調教

> ionice: set or get process I/O scheduling class and priority

這邊使用 [ionice (1) - Linux Man Pages][1] 來完成這件事，剛好 CoreOS 內建有支援這個指令 XD，差點就要包一個 Container 而且要蠻高的權限 (需要 root) 去定期執行了

### Before
```bash
[core@control-plane-2 ~]$ pgrep etcd
740369
740441

[core@control-plane-2 ~]$ ps uax |grep 740369
root      740369  7.6  8.4 9672968 1382392 ?     Ssl  17:05   4:25 etcd --initial-advertise-peer-urls=https://10.0.97.3:2380 --cert-file=/etc/kubernetes/static-pod-certs/secrets/etcd-all-serving/etcd-serving-control-plane-2.crt --key-file=/etc/kubernetes/static-pod-certs/secrets/etcd-all-serving/etcd-serving-control-plane-2.key --trusted-ca-file=/etc/kubernetes/static-pod-certs/configmaps/etcd-serving-ca/ca-bundle.crt --client-cert-auth=true --peer-cert-file=/etc/kubernetes/static-pod-certs/secrets/etcd-all-peer/etcd-peer-control-plane-2.crt --peer-key-file=/etc/kubernetes/static-pod-certs/secrets/etcd-all-peer/etcd-peer-control-plane-2.key --peer-trusted-ca-file=/etc/kubernetes/static-pod-certs/configmaps/etcd-peer-client-ca/ca-bundle.crt --peer-client-cert-auth=true --advertise-client-urls=https://10.0.97.3:2379 --listen-client-urls=https://0.0.0.0:2379 --listen-peer-urls=https://0.0.0.0:2380 --listen-metrics-urls=https://0.0.0.0:9978
core      901685  0.0  0.0  12760  1076 pts/0    S+   18:03   0:00 grep --color=auto 740369

[core@control-plane-2 ~]$ ps uax |grep 740441
root      740441  0.9  0.1 124764 25288 ?        Ssl  17:05   0:32 etcd grpc-proxy start --endpoints https://10.0.97.3:9978 --metrics-addr https://0.0.0.0:9979 --listen-addr 127.0.0.1:9977 --key /etc/kubernetes/static-pod-certs/secrets/etcd-all-peer/etcd-peer-control-plane-2.key --key-file /etc/kubernetes/static-pod-certs/secrets/etcd-all-serving-metrics/etcd-serving-metrics-control-plane-2.key --cert /etc/kubernetes/static-pod-certs/secrets/etcd-all-peer/etcd-peer-control-plane-2.crt --cert-file /etc/kubernetes/static-pod-certs/secrets/etcd-all-serving-metrics/etcd-serving-metrics-control-plane-2.crt --cacert /etc/kubernetes/static-pod-certs/configmaps/etcd-peer-client-ca/ca-bundle.crt --trusted-ca-file /etc/kubernetes/static-pod-certs/configmaps/etcd-metrics-proxy-serving-ca/ca-bundle.crt
core      902165  0.0  0.0  12760  1060 pts/0    S+   18:03   0:00 grep --color=auto 740441

# Get priority and class
[core@control-plane-2 ~]$ sudo ionice -p 740369 740441
none: prio 4
none: prio 0
```

### Tuning

我們可以直接登入 CoreOS 後下 `ionice` 對特定 PID 和硬碟處理權限拉高，可以搭配 ansible 來針對 master node 執行

```bash
# Set priority and class
[core@control-plane-2 ~]$ sudo ionice -c2 -n0 -p `pgrep etcd`
```
  - `-c2`: best-effort
  - `-n0`: the highest priority level

### After
```bash
[core@control-plane-2 ~]$ pgrep etcd
740369
740441

[core@control-plane-2 ~]$ sudo ionice -p 740369 740441
best-effort: prio 0
best-effort: prio 0
```

## 後話

恩...反正裝 etcd 的那個節點之硬碟選用高 IOPS，例如用 SSD，效率和穩定性會提升不少

## References

- [ionice (1) - Linux Man Pages][1]
- [etcd - tuning][3]
- [machineconfig][2]
- [etcd 性能测试与调优][4]
- [Running background tasks on nodes automatically with daemonsets][5]
- [Tuning etcd for Large Installations - Rancher][6]
- [Red Hat OpenShift 4 的 etcd 之幾個你應該要知道的事之一][7]
- [etcd - hardware][8]
- [Deploying OpenShift 4.4 to VMware vSphere 7][9]

[3]: https://github.com/etcd-io/etcd/blob/master/Documentation/tuning.md
[1]: https://www.systutorials.com/docs/linux/man/1-ionice/
[2]: https://docs.openshift.com/container-platform/4.4/architecture/architecture-rhcos.html
[4]: https://zhuanlan.zhihu.com/p/111245626
[5]: https://docs.openshift.com/container-platform/4.4/nodes/jobs/nodes-pods-daemonsets.html
[6]: https://rancher.com/docs/rancher/v2.x/en/installation/options/etcd/
[7]: https://blog.pichuang.com.tw/20200528-openshift-etcd-operator-1/
[8]: https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/hardware.md#disks
[9]: https://www.openshift.com/blog/deploying-openshift-4.4-to-vmware-vsphere-7