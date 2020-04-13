---
layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 3
author: Phil Huang
tags:
  - openshift4
  - container
  - coreos
categories:
  - openshift
date: 2020-04-12 21:59:29
udpated: 2020-04-12 21:59:29
toc: true
---

承襲愛的走馬看花系列的亂寫亂紀錄優良傳統，繼續記錄操作，有興趣想要看前面的文章可以參考這兩邊

- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 1 - Phil Huang][1]
- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 2 - Phil Huang][2]

第三天要來操作的是
1. 實裝 Kubernetes NFS-Client Provisioner
2. 升級 OpenShift 4.y.z
3. 正確匯出 YAML 方式

另外我依然沒梗圖，所以開場 Po 個影片 `Installing OpenShift to Red Hat Virtualization using full-stack automation`，現在 `libvirt` 體系平台也可以使用 `node scaling` 的能力了，影片三分鐘，OpenShift 4.4 之後就會開始支援了

{% youtube uFypQRWEKqo %}

<!--more-->

## 走馬看花之旅: 第三天
### 實裝 Kubernetes NFS-Client Provisioner

我們將利用已經準備好的 NFS Server 給 OpenShift 作為持久儲存 (Persistent Storage) 的後端，並且提供`動態持久儲存 (Dynamic PV)` 的能力，如果不是很清楚 Dynamic PV 的概念，可以參考以前寫的投影片 [20190218_OpenShift Storage 架構思考 - Phil Huang - p10][3]

這邊我已經準備好由 `Synology DS916+` 提供的 NFS Server，你也可以用 RHEL 安裝好 NFS 的服務來作替代，資訊如下：

1. NFS Server FQDN: `farm.pichuang.local`
2. Shared Directory: `/volume1/ocp_nfs`

這邊因為只是簡單測試把 `NFS-Client Provisioner` 相關的驅動程式放在 Project `default`，當然比較好的設計是另外開一個 Project 作為放置驅動的集散地，只是要留意 RBAC 要另外做修改

#### 準備階段

```bash bastion.ocp4.internal
$ git clone https://github.com/kubernetes-incubator/external-storage
$ cd ./external-storage/nfs-client
$ NAMESPACE=`default` # By default, nfs-client provisioner will install in project default
$ sed -i'' "s/namespace:.*/namespace: $NAMESPACE/g" ./deploy/rbac.yaml
$ oc create -f deploy/rbac.yaml
$ oc adm policy add-scc-to-user hostmount-anyuid system:serviceaccount:$NAMESPACE:nfs-client-provisioner
```

#### 部署 Deployment

```yaml deploy/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          # In diconnected environment, please pull the image before installing
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            # Replace NFS Server and PATH
            - name: NFS_SERVER
              value: farm.pichuang.local
            - name: NFS_PATH
              value: /volume1/ocp_nfs
      volumes:
        - name: nfs-client-root
          nfs:
            # Replace NFS Server and PATH
            server: farm.pichuang.local
            path: /volume1/ocp_nfs
```

這邊提供 `git diff` 的差異，比較能了解需要修改的地方
![](/images/nfs-provisioner-diff.png)

```bash bastion.ocp4.internal
$ oc create -f deploy/deployment.yaml
deployment.apps/nfs-client-provisioner created

$ oc get pod
NAME                                      READY   STATUS    RESTARTS   AGE
nfs-client-provisioner-7df5d4f459-4lrnx   1/1     Running   0          73m
```

#### 部署 StorageClass

```yaml deploy/class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage # or choose another name, the name will be applied by Dynamic PV
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
```

```bash bastion.ocp4.internal
$ oc create -f deploy/class.yaml
storageclass.storage.k8s.io/managed-nfs-storage created
```

#### 驗證 Dynamic PV 能力

```bash
#
# Pre Check
#
$ oc get pv,pvc
No resources found in default namespace.

#
# Create PVC
#
$ cat deploy/test-claim.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-claim
  annotations:
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi

$ oc create -f deploy/test-claim.yaml
persistentvolumeclaim/test-claim created

#
# Create Pod
#
# Goal:
# 1. Tocuh a new file - SUCCESS in /mnt
# 2. Mount pv - nfs-pvc
#
$ cat deploy/test-pod.yaml
kind: Pod
apiVersion: v1
metadata:
  name: test-pod
spec:
  containers:
  - name: test-pod
    image: gcr.io/google_containers/busybox:1.24
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: test-claim

$ oc create -f deploy/test-pod.yaml
pod/test-pod created

#
# Post Check
#
$ oc get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS          REASON   AGE
persistentvolume/pvc-719adae1-8fa4-4058-ae48-d9da1c5e7873   1Mi        RWX            Delete           Bound    default/test-claim   managed-nfs-storage            3m6s

NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
persistentvolumeclaim/test-claim   Bound    pvc-719adae1-8fa4-4058-ae48-d9da1c5e7873   1Mi        RWX            managed-nfs-storage   3m6s
```

如果這時候切到 NFS 的目錄裡面的話，就可以看到該檔案了

![](/images/nfs-view.png)

### 升級 OpenShift 4.y.z

#### Cluster Version Operator, CVO
現在 OpenShift 4 的升級已經是改由以 Operator Framework 為基礎實作 `Cluster Version Operator` 出來，主要能力就是提供OpenShift 線上更新機制 (Over The Air, OTA)，

![](/images/ota-clusterversion.png)

前陣子 Red Hat 釋出了一個解釋該 OTA 機制的影片，可以參考 [Red Hat OpenShift: Cluster Upgrades and Application Operator Updates][5]

#### Channel 的選擇

另基於 [Updating a cluster within a minor version from the web console - OpenShift 4.3][6] 所述，預設的 `upgrade channels and release` 會有 3 種管道類型(依穩定度排序):

1. stable-4.y.z: 發行版本 (GA)，最穩定的版本，提供具有 Red Hat OpenShift SRE Team 的長期技術支援下的更新
2. fast-4.y.z: 預發行版本 (pre GA)，發布一段時間後，會發布至 `stable-4.y.z`
3. candidate-4.y.z: 候選版本 (RC)，

Red Hat 技術團隊正式支援的是 `stable-4.y.z` 及 `fast-4.y.z`，不包含 `candidate 4.y.z`，一般沒什麼特別需要，當然是以 `stable-4.y.z` 為優先選項

下面提供一下具體升級拆解步驟，相當...簡單，一堆更新動作都被 `Cluster Version Operator` 做掉了，除了以下的 CLI 操作以外，你也可以使用 `Web Console` 進行更新，詳細可以參考 [OpenShift 4 Over the air updates][7]

```bash
# Simple View
$ oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.3.5     True        False         25d     Cluster version is 4.3.5

# Rich View
$ oc get clusterversion -o json|jq ".items[0].spec"
{
  "channel": "stable-4.3",
  "clusterID": "9d18a202-2a8a-4258-86fb-4319e20c8080",
  "desiredUpdate": {
    "force": false,
    "image": "quay.io/openshift-release-dev/ocp-release@sha256:f0fada3c8216dc17affdd3375ff845b838ef9f3d67787d3d42a88dcd0f328eea",
    "version": "4.3.9"
  },
  "upstream": "https://api.openshift.com/api/upgrades_info/v1/graph"
}

# Check current version and available upgrade inforamtion
$ oc adm upgrade
Cluster version is 4.3.5

Updates:

VERSION IMAGE
4.3.8   quay.io/openshift-release-dev/ocp-release@sha256:a414f6308db72f88e9d2e95018f0cc4db71c6b12b2ec0f44587488f0a16efc42
4.3.9   quay.io/openshift-release-dev/ocp-release@sha256:f0fada3c8216dc17affdd3375ff845b838ef9f3d67787d3d42a88dcd0f328eea

# Upgrade to specific version
$ oc adm upgrade --to=4.3.9
Updating to 4.3.9

# Waiting... The installation time should be over 1 hr
$ watch -n 30 oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.3.5     True        True          4s      Working towards 4.3.9: downloading update

# If you see any information mention
# Unable to apply 4.y.z: the cluster operator <Operator Name> has not yet successfully rolled out
# Solution: Just Wait or use oc describe clusterversion

$ oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.3.9     True        False         49s     Cluster version is 4.3.9
```

### 正確匯出 YAML 方式

上游 Kubernetes 將於 1.18 版本之後，要廢止 `--export` 的參數，建議直接使用 `-o, --output=''` 輸出對應的格式

> -o, --output='': Output format. One of: json|yaml|wide|name|custom-columns=...|custom-columns-file=...|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...

```bash
# Format
# oc get <object kind>/<object name> -o yaml

$ oc get pvc/image-registry-pvc -o yaml
$ oc get pvc/image-registry-pvc -o yaml > image-registry-pvc.yaml
```

## 結語

倘若你需要在 OpenShift 裡面開起 NFS Server 的服務，而不是單單使用 NFS Client 的話，可以參考 [kubernetes-incubator/external-storage/nfs][9] 裡面的操作進行設定，只是我覺得架構上這樣使用好像沒什麼特別的優勢，僅供參考而已。

另剛介紹的 `NFS-Client Provisioner` 其實不在 Red Hat OpenShift 正式支援的功能之一，但技術可用，倘若使用相較穩定的第三方 NFS Client Provisioner，則可以參考 NetApp Trident CSI ，但就是一定要搭著 NetApp Appliance 使用就是了

## References
- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 1 - Phil Huang][1]
- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 2 - Phil Huang][2]
- [20190218_OpenShift Storage 架構思考 - Phil Huang][3]
- [[Kubernetes] 設定 StorageClass (以 NFS 為例) - 小信豬的原始部落][4]
- [Red Hat OpenShift: Cluster Upgrades and Application Operator Updates][5]
- [Updating a cluster within a minor version from the web console - OpenShift 4.3][6]
- [OpenShift 4 Over the air updates][7]
- [KB3660871: `oc export` is deprecated, use `oc get --export`][8]
- [kubernetes-incubator/external-storage/nfs][9]
- [kubernetes-incubator/external-storage/nfs-client][10]

[1]: https://blog.pichuang.com.tw/20200317-openshift-with-coreos-part-1/
[2]: https://blog.pichuang.com.tw/20200403-openshift-with-coreos-part-2/
[3]: https://speakerdeck.com/pichuang/20190218-openshift-storage-jia-gou-si-kao?slide=10
[4]: https://godleon.github.io/blog/Kubernetes/k8s-Config-StorageClass-with-NFS/
[5]: https://www.youtube.com/watch?v=xYd1OyZQBNY
[6]: https://access.redhat.com/documentation/zh-cn/openshift_container_platform/4.3/html-single/updating_clusters/index
[7]: https://www.youtube.com/watch?v=01_F_nyqlLk
[8]: https://access.redhat.com/solutions/3660871
[9]: https://github.com/kubernetes-incubator/external-storage/tree/master/nfs
[10]: https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client