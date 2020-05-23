---
layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 7
author: Phil Huang
toc: true
tags:
  - null
categories:
  - null
date: 2020-05-07 01:48:52
udpated: 2020-05-07 01:48:52
---

## 走馬看花第 6 天
### OpenShift 新增私有容器倉庫
#### 紅帽容器目錄 (Red Hat Container Catalog)

![](/images/rhcc.png)

Red Hat 有提供一個訂閱服務叫做 `Red Hat 容器目錄服務 (Red Hat Container Catalog, RHCC)`，這服務是完全對比 Docker Hub 的功能，最大差異就是裡面提供的 Container Images 都是由 Red Hat 官方提供維護和支援

Docker Hub 下載映像檔是沒什麼問題

- [iThome - Docker Hub上映像檔被發現存在挖礦綁架蠕蟲][1]
- [iThome - Docker Hub遭入侵，外洩19萬名用戶憑證][2]
- [iThome - 駭客掃描網路Docker植入挖礦程式，還修改設定、留下後門][3]
- [iThome - Docker移除17個暗藏挖礦程式的惡意容器][4]

以下是 2 條更新規則:
1. 每 6 週更新一次
2. 有重大 CVE 問題會更新

#### 未經認證

![](/images/rhcc-nginx-rh.png)

你在拉映像檔的時候其實會遇到一個問題，就是這類容器倉庫 (Container Registry) 其實是私有 (Private) 的，你需要有帳號密碼才能拉相關的 Images 下來，舉個下面的例子來說，我想要用 RHCC 提供的一個映像檔叫做 [`rhel8/nginx-116`][5]，正常狀況下會遇到下列顯示

```bash
# New project for testing
$ oc new-project test-images

# Import rhel8/nginx-116 in non-authenticated
$ oc import-image rhel8/nginx-116 --from=registry.access.redhat.com/rhel8/nginx-116 --confirm
error: tag latest failed: Internal error occurred: registry.access.redhat.com/rhel8/nginx-116:latest: unsupported: This repo requires terms acceptance and is only available on registry.redhat.io
imagestream.image.openshift.io/nginx-116 imported with errors

Name:                   nginx-116
Namespace:              test-images
Created:                Less than a second ago
Labels:                 <none>
Annotations:            openshift.io/image.dockerRepositoryCheck=2020-05-03T09:11:06Z
Image Repository:       image-registry.openshift-image-registry.svc:5000/test-images/nginx-116
Image Lookup:           local=false
Unique Images:          0
Tags:                   1

latest
  tagged from registry.access.redhat.com/rhel8/nginx-116

  ! error: Import failed (InternalError): Internal error occurred: registry.access.redhat.com/rhel8/nginx-116:latest: unsupported: This repo requires terms acceptance and is only available on registry.redhat.io
      Less than a second ago
```

#### 針對 registry.redhat.io

1. 針對 `registry.redhat.io` 新增一個 Pull Secret

```yaml registry-redhat-io-pull-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-redhat-io-pull-secret
data:
  .dockerconfigjson: <i dont know>
type: kubernetes.io/dockerconfigjson
```

```bash
# Submit secret to the project openshift in the cluster
$ oc create -f registry-redhat-io-pull-secret.yaml -n test-images

# Check
$ oc get secret registry-redhat-io-pull-secret -n test-images
$ oc describe sa default
Name:                default
Namespace:           test-images
Labels:              <none>
Annotations:         <none>
Image pull secrets:  default-dockercfg-d86fm
Mountable secrets:   default-token-pn8jj
                     default-dockercfg-d86fm
Tokens:              default-token-pn8jj
                     default-token-q6q5c
Events:              <none>

# To use a secret for pulling images for Pods, you must add the secret to your service account. The name of the service account in this example should match the name of the service account the Pod uses. default is the default service account
# https://docs.openshift.com/container-platform/4.3/openshift_images/managing_images/using-image-pull-secrets.html#images-allow-pods-to-reference-images-from-secure-registries_using-image-pull-secrets
# oc secrets link <serviceaccount> <secret> --for=<pull or mount>
$ oc secrets link default registry-redhat-io-pull-secret --for=pull

$ oc describe sa default
Name:                default
Namespace:           test-images
Labels:              <none>
Annotations:         <none>
Image pull secrets:  default-dockercfg-d86fm
                     registry-redhat-io-pull-secret
Mountable secrets:   default-token-pn8jj
                     default-dockercfg-d86fm
Tokens:              default-token-pn8jj
                     default-token-q6q5c
Events:              <none>

```


若你是 Red Hat 的訂閱用戶的話，千萬不要傻傻地把自己的帳號密碼放到 secret 裡面，Red Hat 有提供 Registry Service Account 的申請能力可以讓你放類似機器人帳號在 secret 裡，相當好用，詳細可以 [Red Hat Container Registry Authentication][7] 看看這篇

## References
- [iThome - Docker Hub上映像檔被發現存在挖礦綁架蠕蟲][1]
- [iThome - Docker Hub遭入侵，外洩19萬名用戶憑證][2]
- [iThome - 駭客掃描網路Docker植入挖礦程式，還修改設定、留下後門][3]
- [iThome - Docker移除17個暗藏挖礦程式的惡意容器][4]
- [rhcc - rhel8/nginx-116][5]
- [KB4177741 - Pull images from Authenticated Registry in OpenShift 3][6]
- [Red Hat Container Registry Authentication][7]
- [Allowing Pods to reference images from other secured registries - OpenShift 4.3][8]


[1]: https://www.ithome.com.tw/news/133655
[2]: https://www.ithome.com.tw/news/130275
[3]: https://www.ithome.com.tw/news/134470
[4]: https://www.ithome.com.tw/news/123887
[5]: https://catalog.redhat.com/software/containers/detail/5d400ae7bed8bd3809910782?container-tabs=overview&gti-tabs=unauthenticated
[6]: https://access.redhat.com/solutions/4177741
[7]: https://access.redhat.com/RegistryAuthentication
[8]: https://docs.openshift.com/container-platform/4.1/openshift_images/managing_images/using-image-pull-secrets.html#images-allow-pods-to-reference-images-from-secure-registries_using-image-pull-secrets