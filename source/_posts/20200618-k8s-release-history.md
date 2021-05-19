---
layout: post
title: Kubernetes / OpenShift 發布歷史時間表
author: Phil Huang
toc: true
tags:
  - kubernetes
  - openshift
categories:
  - kubernetes
date: 2020-06-18 00:31:45
udpated: 2020-10-30 00:31:45
---

打從 Kubernetes 1.15 之後，OpenShift 的版本迭代速度就是落差一版而已，在那之前基本上都是兩版的差距，這一切都是為了 [Kubernetes 的版本生命週期規定][1] 的教條而改變的...

![](/images/ocp-and-k8s.png)

<!--more-->

## OpenShift / Kubernetes 釋出歷史紀錄

| Kubernetes Version | K8S  Release Date | OCP Release Date | OpenShift Version |
|--------------------|-------------------|------------------|-------------------|
| 1.21               |                   |                  | 4.8               |
| 1.20               | 2020/12/8         |                  | 4.7               |
| 1.19               | 2020/8/26         | 2020/10/28       | 4.6 (EUS)         |
| 1.18               | 2020/3/24         | 2020/7/9         | 4.5               |
| 1.17               | 2020/1/7          | 2020/4/29        | 4.4               |
| 1.16               | 2019/9/16         | 2020/1/15        | 4.3               |
| 1.15               | 2019/6/17         | -                | -                 |
| 1.14               | 2019/3/25         | 2019/10/16       | 4.2               |
| 1.13               | 2018/12/3         | 2019/6/4         | 4.1               |
| 1.12               | 2018/9/27         | 2019/5/7         | 4.0               |
| 1.11               | 2018/7/3          | 2018/10/10       | 3.11              |

## 關於 OpenShift 發布版本的週期

Red Hat OpenShift Container Platform 主要發布版本的資訊會放置於 [OpenShift Release CI](https://openshift-release.svc.ci.openshift.org) 上面

以近期 OpenShift 發布了 4.6 這個可以使用到 2022 年 3 月的版本，僅需要特別追蹤 `4-stable` 這個部分就好，如果想要比較版本之間的 Changelog 差異的話，則可以使用網站上方的 `Compare` 進行比較

![](/images/ocp-release.png)


## 結語

若你有看完 Kubernetes 的版本生命週期，會獲得到一個結論就是

> 你至少每 9 個月一定要升級 Kubernetes ...

有人跟你說 Kubernetes 建完用一輩子，只能祈禱不會遇到跨版本的高度危險漏洞，譬如像 [CVE-2018-1002105][3]


## References
- [Kubernetes 升級規劃][2]
- [Kubernetes爆重大漏洞！不法人士可取得管理員權限，竊取機敏資料、癱瘓企業應用][3]
- [OpenShift vs Kubernetes: What are the Differences?][4]

[1]: https://kubernetes.io/docs/setup/release/version-skew-policy/
[2]: https://medium.com/infuseai/kubernetes-%E5%8D%87%E7%B4%9A%E8%A6%8F%E5%8A%83-1e0843c1ebcf
[3]: https://www.ithome.com.tw/news/127431
[4]: https://www.whizlabs.com/blog/openshift-vs-kubernetes/