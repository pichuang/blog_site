---
layout: post
title: Red Hat OpenShift 4 DNS & NTP & PTP 網路套餐
author: Phil Huang
toc: true
tags:
  - openshift4
  - openshift
categories:
  - openshift
date: 2020-07-06 00:31:45
udpated: 2020-07-06 00:31:45
---

最近雖然受到 COVID-19 影響導致有些國外專案進度有點落後，但就 OpenShift 4 功能本身不知道為啥成長幅度變的蠻快的...有時候想跟也很難跟上，只能透過影片來了解，然後 Red Hat 本家不知道哪根筋不對了，跑去 Twitch (!!??) 開了一個頻道 [RedHatOpenShift Twitch][8]，三不五時就開直播上演下圖的狀況，相當有事

![](/images/3-bear.png)

<!--more-->

## 走馬看花第 8 天

### CoreDNS 做 DNS Forwarder

> CoreDNS: DNS and Service Discovery

其實 OpenShift 內建有 DNS 服務，主要目的是要來解析 Service IP <-> DNS，所採用的技術是使用 `CoreDNS`。關於 [CoreDNS][2] ，主要是提供雲原生的 DNS 服務和服務發現的解決方案，有興趣想參考效能比較的話，可以看 [Scaling CoreDNS in Kubernetes Clusters][3]，目前已於 2019/01 時從 CNCF 專案海中[畢業][1]。

如果你仔細觀察 CoreDNS 在 OpenShift 上的運作狀況可以發現他是從 `openshift-dns-operator` 部署，並於每一個節點採用 `DaemonSet` 的形式運行 CoreDNS

```bash
# Operator Information
$ oc get deployment/dns-operator -n openshift-dns-operator
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
dns-operator   1/1     1            1           113m

$ oc get clusteroperators/dns
NAME   VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
dns    4.3.27    True        False         False      108m

# Project Information
$ oc get all -n openshift-dns -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE                                              NOMINATED NODE   READINESS GATES
pod/dns-default-9vq2j   2/2     Running   0          84m   10.129.0.10   ip-10-0-146-166.ap-southeast-1.compute.internal   <none>           <none>
pod/dns-default-g2bps   2/2     Running   0          84m   10.130.0.9    ip-10-0-167-32.ap-southeast-1.compute.internal    <none>           <none>
pod/dns-default-mg7cx   2/2     Running   0          84m   10.128.0.13   ip-10-0-129-186.ap-southeast-1.compute.internal   <none>           <none>
pod/dns-default-sxzcn   2/2     Running   0          75m   10.128.2.3    ip-10-0-132-54.ap-southeast-1.compute.internal    <none>           <none>
pod/dns-default-wj97g   2/2     Running   0          75m   10.131.0.3    ip-10-0-154-36.ap-southeast-1.compute.internal    <none>           <none>

NAME                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
service/dns-default   ClusterIP   172.30.0.10   <none>        53/UDP,53/TCP,9153/TCP   83m

NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/dns-default   5         5         5       5            5           kubernetes.io/os=linux   83m
```

倘若你想要透過 VPN 去連線至其他雲裡面的服務，那這樣會有需要特別多加幾條 DNS Forwarder 的能力在集群裡面，遵照 [DNS Operator in OpenShift Container Platform][4]

```bash
# Default Setting
$ oc get configmap/dns-default -n openshift-dns -o yaml
apiVersion: v1
data:
  Corefile: |
    .:5353 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            upstream
            fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf {
            policy sequential
        }
        cache 30
        reload
    }
kind: ConfigMap
metadata:
  labels:
    dns.operator.openshift.io/owning-dns: default
  name: dns-default
  namespace: openshift-dns

# 新增 DNS Forwarder
$ cat << EOF >> apply-dns-forwarder.yaml
apiVersion: operator.openshift.io/v1
kind: DNS
metadata:
  name: default
spec:
  servers:
  - name: pichuang-server-farm
    zones:
      - pichuang.internal
    forwardPlugin:
      upstreams:
        - 192.168.77.100
        - 192.168.77.99:5533
  - name: redhat-server-farm
    zones:
      - rhtw.private
      - rhglobal.public
    forwardPlugin:
      upstreams:
        - 10.99.1.100
        - 172.88.2.99:5533
EOF

# 執行變更
$ oc apply -f apply-dns-forwarder.yaml

# 檢查 CoreDNS 設定
$ oc get configmap/dns-default -n openshift-dns -o yaml
apiVersion: v1
data:
  Corefile: |
    # pichuang-server-farm
    pichuang.internal:5353 {
        forward . 192.168.77.100 192.168.77.99:5533
    }
    # redhat-server-farm
    rhtw.private:5353 rhglobal.public:5353 {
        forward . 10.99.1.100 172.88.2.99:5533
    }
    .:5353 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            upstream
            fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf {
            policy sequential
        }
        cache 30
        reload
    }
kind: ConfigMap
metadata:
  labels:
    dns.operator.openshift.io/owning-dns: default
  name: dns-default
  namespace: openshift-dns
```


### 時間設定 Chrony

打從 RHEL 7 之後，Red Hat 官方因為安全及效能上的考量，建議從 `ntpd` 轉換成使用 `chronyd`，實際上也是蠻推薦多用 chronyd 當作主要的 NTP Server/Client，就連 OpenShift 預載的 CoreOS 內部也是以 Chronyd 為校時基準，參考 [NTP configuration - OpenShift Tips][5]，以下是校時的操作

1. 先檢查預設的狀態，隨便挑一台用 `oc debug node/<node name>` 進去觀察時間

```
$ oc debug node/ip-10-0-129-186.ap-southeast-1.compute.internal
Starting pod/ip-10-0-129-186ap-southeast-1computeinternal-debug ...
To use host binaries, run `chroot /host`
Pod IP: 10.0.129.186
If you don't see a command prompt, try pressing enter.

sh-4.2# chroot /host
sh-4.4# cat /etc/chrony.conf
sh-4.4#  chronyc tracking
Reference ID    : DABA0324 (time1.maxonline.com.sg)
Stratum         : 2
Ref time (UTC)  : Tue Jul 07 08:12:11 2020
System time     : 0.000018082 seconds fast of NTP time
Last offset     : +0.000020540 seconds
RMS offset      : 0.000040942 seconds
Frequency       : 1.571 ppm slow
Residual freq   : +0.007 ppm
Skew            : 0.394 ppm
Root delay      : 0.002178520 seconds
Root dispersion : 0.000406400 seconds
Update interval : 64.2 seconds
Leap status     : Normal
```

2. 準備 `chrony.conf`，然後進行 Base64 編碼
```bash
$ cat << EOF >> chrony.conf
pool 0.tw.pool.ntp.org iburst
pool 3.tw.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
EOF

$ cat chrony.conf | python3 -m base64 | xargs echo -n
cG9vbCAwLnR3LnBvb2wubnRwLm9yZyBpYnVyc3QKcG9vbCAzLnR3LnBvb2wubnRwLm9yZyBpYnVyc3QKZHJpZnRmaWxlIC92YXIvbGliL2Nocm9ueS9kcmlmdAptYWtlc3RlcCAxLjAgMwpydGNzeW5jCmtleWZpbGUgL2V0Yy9jaHJvbnkua2V5cwpsZWFwc2VjdHogcmlnaHQvVVRDCmxvZ2RpciAvdmFyL2xvZy9jaHJvbnkK
```

3. 準備 2 個聲明檔，如果你在 `machine config pool` 還有多新增 infra 的話，也要特別寫一個，原則上就是一個角色要有一個檔案就是了
```bash
$ cat << EOF > 98-worker-chrony.yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: 98-worker-chrony-configuration
spec:
  config:
    ignition:
      version: 2.2.0
    storage:
      files:
      - contents:
          source: data:text/plain;charset=utf-8;base64,cG9vbCAwLnR3LnBvb2wubnRwLm9yZyBpYnVyc3QKcG9vbCAzLnR3LnBvb2wubnRwLm9yZyBpYnVyc3QKZHJpZnRmaWxlIC92YXIvbGliL2Nocm9ueS9kcmlmdAptYWtlc3RlcCAxLjAgMwpydGNzeW5jCmtleWZpbGUgL2V0Yy9jaHJvbnkua2V5cwpsZWFwc2VjdHogcmlnaHQvVVRDCmxvZ2RpciAvdmFyL2xvZy9jaHJvbnkK
        verification: {}
        filesystem: root
        mode: 420
        path: /etc/chrony.conf
EOF

$ cat << EOF > 98-master-chrony.yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: 98-master-chrony-configuration
spec:
  config:
    ignition:
      version: 2.2.0
    storage:
      files:
      - contents:
          source: data:text/plain;charset=utf-8;base64,cG9vbCAwLnR3LnBvb2wubnRwLm9yZyBpYnVyc3QKcG9vbCAzLnR3LnBvb2wubnRwLm9yZyBpYnVyc3QKZHJpZnRmaWxlIC92YXIvbGliL2Nocm9ueS9kcmlmdAptYWtlc3RlcCAxLjAgMwpydGNzeW5jCmtleWZpbGUgL2V0Yy9jaHJvbnkua2V5cwpsZWFwc2VjdHogcmlnaHQvVVRDCmxvZ2RpciAvdmFyL2xvZy9jaHJvbnkK
        verification: {}
        filesystem: root
        mode: 420
        path: /etc/chrony.conf
EOF

$ oc apply -f 98-master-chrony.yaml
machineconfig.machineconfiguration.openshift.io/98-masters-chrony-configuration created

$ oc apply -f 98-worker-chrony.yaml
machineconfig.machineconfiguration.openshift.io/98-workers-chrony-configuration created
```

### 精確時間協定 PTP

想不到吧! OpenShift 4 有內建支援 IEEE 1588 (Precision Time Protocol, PTP) 協定，來支援硬體級別的精確時間同步，當然要使用這個技術需要仰賴硬體設備上的支援，包含像是網卡以及交換機，OpenShift 4 是將 [Linux PTP][6] 透過 PTP Operator 安裝於所有節點之上，透過內建自動能力查詢機制來協助將具備 PTP 能力之網卡設定其相關參數。其實主要這個功能是為了 O-RAN W6 裡面有定義條目 `5.5.1 Cloud Platform Time Synchronization Architecture` 而做的，裡面描述如下

> The Time Sync deployment architecture which is described below relies on usage of Precision Time Protocol (PTP) IEEE 1588-2008 (a.k.a. IEEE 1588 Version 2) to synchronize clocks throughout the Edge Cloud site.

其實安裝相當簡單，主要是透過 Operator Framework 從頭打到底，如果有興趣想看過程的可以參考該 [GitHub - pichuang/redhat-ptp-demo][7]

```bash
$ oc get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/linuxptp-daemon-2z2sx           1/1     Running   0          19m
pod/linuxptp-daemon-nllxn           1/1     Running   0          19m
pod/linuxptp-daemon-phkwp           1/1     Running   0          19m
pod/linuxptp-daemon-v7w4s           1/1     Running   0          19m
pod/linuxptp-daemon-xh5fp           1/1     Running   0          19m
pod/ptp-operator-575797775d-7bcdf   1/1     Running   0          19m

NAME                          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/ptp-monitor-service   ClusterIP   None         <none>        9091/TCP   19m

NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/linuxptp-daemon   5         5         5       5            5           kubernetes.io/os=linux   19m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ptp-operator   1/1     1            1           19m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/ptp-operator-575797775d   1         1         1       19m
```

哦對了，我身上沒有支援 PTP 的網卡，所以看不出來效果，帶有錢的大大找我去研究相關使用方式 XD


## Appendix
### Environment Information
- OpenShift 4.3.27
- Kubernetes Version: v1.16.2+d6d3cff

[1]: https://www.cncf.io/announcement/2019/01/24/coredns-graduation/
[2]: https://coredns.io/
[3]: https://github.com/coredns/deployment/blob/master/kubernetes/Scaling_CoreDNS.md
[4]: https://docs.openshift.com/container-platform/4.4/networking/dns-operator.html
[5]: https://openshift.tips/machine-config/
[6]: http://linuxptp.sourceforge.net/
[7]: https://github.com/pichuang/redhat-ptp-demo
[8]: https://www.twitch.tv/redhatopenshift