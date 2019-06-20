layout: post
title: 設計出 Production-Ready Container 的 5 條戒律
author: Phil Huang
date: 2019-06-20 17:37:14
tags:
---
![](/images/rh-ubi-1.png)

Red Hat 官方前陣子推出 Red Hat Enterprise Linux 8 (RHEL8)，裡面包含一個很大的變化就是針對 Linux 容器技術有非常大的改進和變化。應對 Docker 商業策略上的變化，由 Google、Red Hat、SUSE、IBM、Intel 攜手一起開發了一個`專門對 Kubernetes 進行優化`的容器運行環境 (Container Runtime) 的專案 [CRI-O][3] (Container Runtime Interface Orchestrator)。故 RHEL 8 內建的容器技術將以 CRI-O 為核心，連帶對應的容器工具 Podman、Buildah、Skopeo 一起往未來發展。而今天要分享的是如何設計出生產環境等級的容器映像檔 (Container Images)，基於 Red Hat Summit 2019 分享出的簡報 [Building Production-Ready Containers - Scott McCarty & Ben Breard][4] 撰寫，一位是專職的 Linux Container Product Manager，另一位是 CoreOS Product Manager，意思是戰鬥值爆表 XD。

<!--more-->


## 你所認為的 Container Images?

> 金玉其外，或許...敗絮其內?

很多所被建構出來的 Container Images，實際上使用可以用，但內部卻是亂糟糟的，例如：

- 混亂不明的依賴性
- 不知道為什麼會在那邊的 Process (ex: sshd)
- 發生 CVE 的時候不知道要怎麼修怎麼更新

以上三個狀況是平時維運或使用容器技術時最常見的狀況：升級障礙、設計錯誤、資安修補



## 5 條戒律 (Commandents)

以下的 5 條戒律的討論基礎是基於所有內容、工作都視為可以從原始碼 (Source Code) 構建，具體內容為以下：

1. 標準化 (Standardize)：盡可能確保每一個人都使用相同的基礎映像檔 (Base Images)
2. 最小化 (Minimize)：映像檔僅裝載實際工作負載所需的內容
3. 委派 (Delegate)：保持映像檔各層次 (Layers) 由各具備專門技術的人負責
4. 過程和自動化 (Process and Automation)：透過 Helm Charts、Ansible Playbooks 或 Operators 等部署方式，盡可能做到自動化佈建
5. 迭代 (Iterate)：盡可能在迭代過程修正任何發現的原始碼錯誤

### 第一戒律：Standardize
![](/images/Commandent-1-standardize.png)

### 第二戒律：Minimize
![](/images/Commandent-2-minimize.png)

### 第三戒律：Delegate
![](/images/Commandent-3-delegate.png)

### 第四戒律：Process and Automation
![](/images/Commandent-4-focus-on-automation.png)

### 第五戒律：Iterate
![](/images/Commandent-5-iterate.png)




## References
- [CRI-O official website][3]
- [Building Production-Ready Containers - Scott McCarty & Ben Breard][4]
- [Linux Container Intermals - how they really work][1]


[1]: https://developers.redhat.com/blog/2019/06/04/container-related-information-you-might-have-missed-at-red-hat-summit/
[3]: https://cri-o.io/
[4]: http://crunchtools.com/files/2019/05/Summit-2019_-Building-Production-Ready-Containers.pdf