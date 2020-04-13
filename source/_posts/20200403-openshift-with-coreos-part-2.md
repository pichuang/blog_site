layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 2
author: Phil Huang
tags:
  - openshift4
  - container
  - coreos
categories:
  - openshift
date: 2020-04-03 00:16:00
toc: true
---

上個月寫了一篇愛的走馬看花 [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 1][1]，繼續逛下去

第二天比較偏向是 Red Hat OpenShift Infrastrcuture 除錯層面的閒逛，有些 Kubernetes 應該是不能直接套用的，譬如像是 `oc adm`、`oc debug` 這類的指令，但是我記得新版的 Kubernetes 好像有 `kubectl debug` 的指令可以用了，但不知道效果是不是一樣的

另外開頭我沒梗圖了，故附上一個我覺得蠻不錯的影片 [Deploying a Windows Server 2019 virtual machine using OpenShift Virtualization][8]

{% youtube Kx110kqoHo0 %}

<!--more-->

## 走馬看花之旅: 第二天
### 透過 OpenShift 顯示各節點上的特定服務 Log

Red Hat OpenShift 4 之後的服務，放在 Red Hat CoreOS (後面簡稱 RHCOS) 裡面運行的服務，絕大部分都是都是以容器 (Container) 的方式運行，但還是有 18 個服務是透過 `systemd` 運行的，下面列舉服務且標注對維運上比較重要的服務

```
$ oc debug node/compute-0
Starting pod/compute-0-debug ...
To use host binaries, run `chroot /host`

Pod IP: 10.0.97.4
If you don't see a command prompt, try pressing enter.
sh-4.2# chroot /host
sh-4.4# systemctl list-units --type=service --state=running

UNIT                     LOAD   ACTIVE SUB     DESCRIPTION
auditd.service           loaded active running Security Auditing Service
chronyd.service          loaded active running NTP client/server
crio.service             loaded active running Open Container Initiative Daemon
dbus.service             loaded active running D-Bus System Message Bus
getty@tty1.service       loaded active running Getty on tty1
irqbalance.service       loaded active running irqbalance daemon
kubelet.service          loaded active running Kubernetes Kubelet
NetworkManager.service   loaded active running Network Manager
polkit.service           loaded active running Authorization Manager
rpc-statd.service        loaded active running NFS status monitor for NFSv2/3 locking.
rpcbind.service          loaded active running RPC Bind
sshd.service             loaded active running OpenSSH server daemon
sssd.service             loaded active running System Security Services Daemon
systemd-journald.service loaded active running Journal Service
systemd-logind.service   loaded active running Login Service
systemd-udevd.service    loaded active running udev Kernel Device Manager
vgauthd.service          loaded active running VGAuth Service for open-vm-tools
vmtoolsd.service         loaded active running Service for virtual machines hosted on VMware

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

18 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```

這邊值得一提的是，因為 RHCOS 內建還是有 `systemd-journald`，所以依然可以使用 `oc debug` 連進去節點後，下 `journalctl` 進行觀察 Log 的行為，但...只是要看個 Log 這些步驟太麻煩了，所以 OpenShift 有整了一個指令叫做 `oc adm node-logs`

所以看 Log 可以這樣用

```bash
# Example:
# oc adm node-logs -u <service> <node-ip>
# oc adm node-logs <node-ip> <log-name> --path=kube-apiserver/<log-name>

# Show sshd logs from specific node
$ oc adm node-logs -u sshd compute-0

# Show crio logs from all master nodes
$ oc adm node-logs -u crio --role master

# Show ALL logs from specific node
$ oc adm node-logs compute-0
```

### 應用程式之 Pod 故障排除

有時候運行一個應用程式，但啟動失敗 (Failing)，而且容器也被終止掉 (Terminating)，依據 [`Pod Restart Policy`][3] 會嘗試繼續重新啟動。如果這種狀況一直持續發生的話，則部署則為失敗的。OpenShift 會將 Pod 的狀態 (Status) 會標註為 `CrashLoopBackOff` 來表示

而為了應對這種需要深入進去了解為什麼會產生 `CrashLoopBackOff` 的原因，所以 OpenShift 於 3.x 版的時候就支援了一個超好用的指令 `oc debug`，透過該指令，複製 (Clone) 出一個 Pod 出來後，則會自動覆蓋一個 Shell 進去當作進入點 (Entrypoint)

```bash
$ oc get pods

NAME               READY     STATUS      RESTARTS   AGE
welcome-1-deploy   0/1       Completed   0          1m
welcome-1-hh7h8    1/1       Running     0          0s

$ oc get dc
NAME        REVISION    DESIRED     CURRNET     TRIGGERED BY
welcome     1           1           1           config,image(welcome:latest)

# Debug a pod
$ oc debug pod/welcome-1-hh7h8

# See the pod that would be created to debug
$ oc debug pod/welcome-1-hh7h8 -o yaml

# Debug a currently running deployment by creating a new pod
$ oc debug dc/welcome
$ oc debug dc/welcome --as-root
$ oc debug dc/welcome --as-user=10000
```

要留意的事情是，在這個 Debug 狀態的 Pod 操作會有以下的限制情況發生

- 無法透過服務名稱從任何 Pod 連接到裡面
- 無法 `oc expose route` 出去
- Readiness / Liveness probe 無法正常使用

### 內部服務 DNS 解析 - DNS Operator

節錄此 [OpenShift 4.3 - DNS Operator in OpenShift Container Platform][4] 內容，預設 OpenShift 4 的內部 DNS 是採用 `CoreDNS`，安裝是透過 Operator Framework 進行維運使用

```bash
# Every new OpenShift Container Platform installation has a "dns.operator" named "default".
$ oc describe dns.operator/default

Name: default
...
API Version:  operator.openshift.io/v1
Kind:         DNS
...
Spec:
Status:
  Cluster Domain:  cluster.local
  Cluster IP:      172.30.0.10
...
```

CoreDNS 主要處理 OpenShift 內部服務的名稱解析，包含以下 4 類:

1. cluster.local
2. `<project>`.cluster.local
3. `<service>`.`<project>`.cluster.local
4. `<pod>`.`<project>`.cluster.local

### 了解當下網路資訊 - Cluster Network Operator

通常來說，建立 OpenShift 需要三段網段，而所有 IP 網段都不能重複，分別是下列:

1. Node Subnet：給實體主機用的 IP
2. Service Subnet：給 Kubernetes Service 使用的 IP，對照設定檔名字為 `serviceNetwork`
3. Pod Subnet：Pod 實際上會拿到的 IP，對照設定檔名字為 `clusterNetwork`

如果想要確認當前的網路設定資訊，可以參考下面作法

```bash
$ oc get Network.config.openshift.io cluster -oyaml

apiVersion: config.openshift.io/v1
kind: Network
...
spec:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  externalIP:
    policy: {}
  networkType: OpenshiftSDN
  serviceNetwork:
  - 172.30.0.0/16
...
```

按照上述輸出，可以了解到以下幾件事
1. clusterNetwork 也就是 Pod 實際上能使用的 IP 為 10.128.0.0/14，共 [`262142` 個 Pod IP][5]
2. networkType 也就是 CNI Plugin 的名稱，這邊是使用原生的 `OpenShift SDN`
3. serviceNetwork 也就是 Kubernetes Service 可以使用的 IP 為 172.30.0.0/16，共可以開啟 [`65534` 個 Service IP][6]
4. hostPrefix 比較難理解，主要是代表`每一個節點 (Node) 所能拿到的 Pod IP 為多少`，例如這邊是 `/23`，所以每個節點可以運行 [`510` 個 Pod IP][9] 的 Pod，若改成 `/22`，則每個節點可運行約 `1000` 左右的 Pod。有些人會問夠不夠用，其實就預設 [OpenShift 4.3 - Recommended host practices][7] 上，一個節點的上限是跑 `kubeletConfig.maxPods: 250`，最高 250，所以撞不到天花板

## 結語

我知道寫的蠻亂的，就當筆記邊研究邊紀錄

## References
- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 1][1]
- [How do I debug an application that fails to start up?][2]
- [Configuring how pods behave after restart][3]
- [DNS Operator in OpenShift Container Platform][4]
- [OpenShift 4.3 - Recommended host practices][7]

[1]: https://blog.pichuang.com.tw/20200317-openshift-with-coreos-part-1/
[2]: https://cookbook.openshift.org/logging-monitoring-and-debugging/how-do-i-debug-an-application-that-fails-to-start-up.html
[3]: https://docs.openshift.com/container-platform/4.3/nodes/pods/nodes-pods-configuring.html#nodes-pods-configuring-restart_nodes-pods-configuring
[4]: https://docs.openshift.com/container-platform/4.3/networking/dns-operator.html
[5]: http://jodies.de/ipcalc?host=10.128.0.0&mask1=14&mask2=
[6]: http://jodies.de/ipcalc?host=172.30.0.0&mask1=16&mask2=
[7]: https://docs.openshift.com/container-platform/4.3/scalability_and_performance/recommended-host-practices.html#recommended-node-host-practices_
[8]: https://www.youtube.com/watch?v=Kx110kqoHo0&list=PLaR6Rq6Z4IqeGIzsnWX1ifTLwJaYBQSUs&index=39&t=0s
[9]: http://jodies.de/ipcalc?host=10.128.0.0&mask1=23&mask2=