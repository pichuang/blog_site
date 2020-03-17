layout: post
title: 在 OpenShift 如何於使用不同的 UserID 運行容器?
author: Phil Huang
tags: []
categories: []
date: 2020-03-17 00:16:00
---

誠如大家所知，因爲種種技術及非技術因素上 - [Why Red Hat is investing in CRI-O and Podman][1]，OpenShift 4 底層有兩個蠻大的更新

1. 容器運行時環境 (Container Runtime) 從 docker 換成了 [CRI-O][2]
2. 作業系統從通用型 Red Hat Enterprise Linux 換成了專為容器特化的容器作業系統 Red Hat CoreOS

本文將記錄一下於 Red Hat CoreOS (RHCOS) 上的探索紀錄

<!--more-->

## 前提
![](/images/ocp4.png)

因為小弟已經裝好了 Red Hat OpenShift 4 的環境，所以我已經有 3 master 及 2 compute 節點可以登進去使用，倘若你想要自己研究的話，可以下載 [Fedora CoreOS][3] 試試看

## 探索之旅

### 1. 登入 CoreOS 節點

有兩個方式:
1. 直接透過 SSH 登入
2. 透過 `oc debug` 進入


先講第一種方式，這邊唯一的要留意的就是，預設登入的帳號是 `core`，而不是 root，而且僅能拿當初透過 ignition file 匯入至 CoreOS 的 SSH Private Key 登入

```bash
$ ssh core@compute-0.ocp4.internal -i ~/.ssh/id_rsa_ocp4_vcenter

Red Hat Enterprise Linux CoreOS 43.81.202003052336.0
  Part of OpenShift 4.3, RHCOS is a Kubernetes native operating system
  managed by the Machine Config Operator (`clusteroperator/machine-config`).

WARNING: Direct SSH access to machines is not recommended; instead,
make configuration changes via `machineconfig` objects:
  https://docs.openshift.com/container-platform/4.3/architecture/architecture-rhcos.html

---
Last login: Tue Mar 17 03:01:29 2020 from 10.0.97.100
[core@compute-0 ~]$ cat /etc/os-release
NAME="Red Hat Enterprise Linux CoreOS"
VERSION="43.81.202003052336.0"
VERSION_ID="4.3"
OPENSHIFT_VERSION="4.3"
RHEL_VERSION=8.0
PRETTY_NAME="Red Hat Enterprise Linux CoreOS 43.81.202003052336.0 (Ootpa)"
ID="rhcos"
ID_LIKE="rhel fedora"
ANSI_COLOR="0;31"
HOME_URL="https://www.redhat.com/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="OpenShift Container Platform"
REDHAT_BUGZILLA_PRODUCT_VERSION="4.3"
REDHAT_SUPPORT_PRODUCT="OpenShift Container Platform"
REDHAT_SUPPORT_PRODUCT_VERSION="4.3"
OSTREE_VERSION='43.81.202003052336.0'
```

第二種方式是透過 `oc debug` 運行一個 debug 用的 pod 後，再透過 `chroot` 切換到 / 目錄

```bash
$ oc debug node/compute-0
Starting pod/compute-0-debug ...
To use host binaries, run `chroot /host`

Pod IP: 10.0.97.4
If you don't see a command prompt, try pressing enter.
sh-4.2# chroot /host
sh-4.4# cat /etc/os-release
NAME="Red Hat Enterprise Linux CoreOS"
VERSION="43.81.202003052336.0"
VERSION_ID="4.3"
OPENSHIFT_VERSION="4.3"
RHEL_VERSION=8.0
PRETTY_NAME="Red Hat Enterprise Linux CoreOS 43.81.202003052336.0 (Ootpa)"
ID="rhcos"
ID_LIKE="rhel fedora"
ANSI_COLOR="0;31"
HOME_URL="https://www.redhat.com/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="OpenShift Container Platform"
REDHAT_BUGZILLA_PRODUCT_VERSION="4.3"
REDHAT_SUPPORT_PRODUCT="OpenShift Container Platform"
REDHAT_SUPPORT_PRODUCT_VERSION="4.3"
OSTREE_VERSION='43.81.202003052336.0'
```


### 2. 切換成 root

預設在 RHCOS 裡面僅會有兩個帳號，分別是 `core` 及 `root`，而預設已經把 sudoer 寫好了，所以可以直接 sudo 切過去

```bash
[core@compute-0 ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
core:x:1000:1000:CoreOS Admin:/var/home/core:/bin/bash
[core@compute-0 ~]$ sudo su -
Last login: Tue Mar 17 02:41:02 UTC 2020 on pts/0
[root@compute-0 ~]#
```

### 3. CRI-O 探索

誠如開頭所說的，OpenShift 4 預設的 Container Runtime 是採用 `CRI-O`，所以若要在 CoreOS 上找尋類似於 `docker` 或 `podman` 的指令，則為 `crictl`

常用的指令組合不外乎為 crictl [images | ps | pods | stats]，詳細的操作指令輸出可以參考 [Debugging Kubernetes nodes with crictl][4]

```bash
[root@control-plane-1 ~]# crictl images
IMAGE                                                                                 TAG                 IMAGE ID            SIZE
<none>                                                                                <none>              493f2db8b5178       728MB
<none>                                                                                <none>              fa4b1c816921a       251MB
<none>                                                                                <none>              c3a63b432c903       420MB
```

### 4. 運行一個容器於 CoreOS 上

假設我想要跑 `tcpdump` 在節點上，但原生 RHCOS 是沒有包含這個套件的，這時候就得要執行一個自帶 `tcpdump` 的容器運行在節點上，而先前有針對這議題寫過一篇文章 [Troubleshooting from Container to Any - Phil Huang][5]，也是基於同樣的概念







```bash
# 找尋 K-V
oc get node --show-labels=true

# 開新專案
oc new-project debugging

# 指定 compute-0
oc run ocp-debug-container --image=quay.io/pichuang/debug-container \
   --restart=Never --attach -i --tty --rm \
   --overrides='{ "kind": "Pod", "apiVersion": "v1", "spec": { "hostNetwork":true, "nodeSelector":{"kubernetes.io/hostname":"compute-0"}}}'

# 移除 ocp-debug-container
oc delete pod ocp-debug-container
```

## Appendix

### 如何使用任意 User ID 啟動 Container Image？

使用 `oc create serviceaccount` 新增一個可以跑 anyuid 的帳號

```bash
# 確認當前 service account
oc get serviceaccount

# 新增 service account - runasanyuid
oc create serviceaccount runasanyuid

# 確認新增後 service account
oc get serviceaccount
```

預設狀況下，容器會受限於 SCC `restricted` 下運行，Run As User 的 Must Run As Range 的 UID 範圍會受限於專案 (Porject) 設定

倘若需要允許應用程式以任何使用者 ID (包含 root) 執行，則需要新增 SCC `anyuid` 給指定的 service account

```bash
# 以 administrator 權限，確認當前 SCC 清單
oc get scc --as system:admin

# 確認 SCC - restricted
oc describe scc restricted

# 新增 SCC - anyuid 能力給 service account - runasanyuid 於當前專案 project - debbuging
oc adm policy add-scc-to-user anyuid -z runasanyuid --as system:admin
```


## References
- [Why Red Hat is investing in CRI-O and Podman - Red Hat Blog][1]
- [CRI-O - LIGHTWEIGHT CONTAINER RUNTIME FOR KUBERNETES][2]
- [Installing CoreOS on Bare Metal - Fedora CoreOS][3]
- [Debugging Kubernetes nodes with crictl - Kubernetes][4]
- [Troubleshooting from Container to Any - Phil Huang][5]
- [How to TCPdump effectively in Kubernetes (part 1)][8]
- [How to TCPdump effectively in Kubernetes (part 2)][7]
- [How can I enable an image to run as a set user ID?][9]
- [配置安全環境定義限制][10]

[1]: https://www.redhat.com/en/blog/why-red-hat-investing-cri-o-and-podman
[2]: https://cri-o.io/
[3]: https://docs.fedoraproject.org/en-US/fedora-coreos/bare-metal/
[4]: https://kubernetes.io/docs/tasks/debug-application-cluster/crictl/
[5]: https://blog.pichuang.com.tw/20190715-troubleshooting-from-container-to-any/
[7]: https://medium.com/@xxradar/how-to-tcpdump-effectively-in-kubernetes-part-2-7e4127b42dc7
[8]: https://medium.com/@xxradar/how-to-tcpdump-effectively-in-kubernetes-part-1-a1546b683d2f
[9]: https://cookbook.openshift.org/users-and-role-based-access-control/how-can-i-enable-an-image-to-run-as-a-set-user-id.html
[10]: https://cloud.ibm.com/docs/openshift?topic=openshift-openshift_scc&locale=zh-tw